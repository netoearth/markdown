~哪个男孩不想拥有一个速度非常快的博客~ 之前我也写过少许 [关于 Web 性能优化的文章](https://blog.skk.moe/tags/%E5%89%8D%E7%AB%AF%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/)，但是却从未介绍过自己的博客是如何优化的。这次我来水篇文章，罗列一下我的博客的几个零散的优化点。

## 服务器与 CDN

很多人都在我的博客下评论，问我的博客用了哪里的服务器这么快。实际上，我不仅没有用香港、新加坡等亚太服务器，而是用的被誉为「减速器」的 Cloudflare，因为不少好事者和脚本小子喜欢拿我的博客开涮。有了 Cloudflare，我可以预先编写好 FIrewall Rules，几乎可以无视所有的 Layer 7 DDoS 攻击：

![cf-attack-waf](https://img10.360buyimg.com/ddimg/jfs/t1/89687/20/20002/55145/62579dadEf78dea36/a4bd04c1eff4add6.png)

> 可以看到，6 月 22 日我的博客遭受了一次非常小规模的 Layer 7 DDoS 攻击，但是由于我预先编写的完善的 FIrewall Rules，所有攻击请求全部被 Block、博客的正常运行毫无影响，甚至不需要人工介入干预。

但是，Cloudflare 中国大陆方向的网络质量是我们无法控制的，因此如果要缩短网站响应耗时、减少 TTFB（首字节时间），就只能在 Cloudflare 回源上做优化。在这一点上我做到了极致，直接 [将博客部署在 Cloudflare Workers Site 上](https://blog.skk.moe/post/deploy-blog-to-cf-workers-site/)，意味着我的博客就托管在 Cloudflare 位于全球的 200 余数据中心上，没有任何回源用时。这样 TTFB 唯一的影响因素就是访客到 Cloudflare 的网络质量。

你可以通过浏览器的 DevTools 或者用 curl，从响应头中查看 Cloudflare Workers Site 的内部用时：

```
curl -s -o /dev/null -D - https://blog.skk.moe | grep 'sukka'
```

其中：

-   `sukka-cache`：缓存命中状态
-   `sukka-timer`：各项计时，包括
    -   cache：从内部缓存获取资源的用时
    -   kv：从 Cloudflare KV Storage 中获取资源的用时
    -   resp：Cloudflare Workers Site 生成响应的完整用时

## 静态资源优化

### 静态资源压缩

类似于 HTML 压缩、CSS 压缩、图片压缩之类的方法已经成为老生常谈了，我也不必赘述，简单介绍一下我用的工具：

-   HTML 压缩我使用的是 [HTMLMinifier](https://www.npmjs.com/package/html-minifier)，我自己写了目录遍历和多核多线程压缩，比单线程执行快两倍。为 Node.js 多线程编写多线程应用可参考我的「[Node.js 多线程 —— worker\_threads 初体验](https://blog.skk.moe/post/say-hello-to-nodejs-worker-thread/)」
-   CSS 压缩我使用的是 [clean-css](https://github.com/jakubpawlowicz/clean-css) 而不是 cssnano，因为后者依赖 PostCSS 工具链、而且压缩效果不如前者
-   图片压缩我使用的是 TinyPNG，相比 ImageOptim 等工具 TinyPNG 的压缩率更高、有损压缩时图片细节丢失最少。

### 减少静态资源体积

CSS 经过 clean-css 压缩后文件大小会减少 20%，HTML 经过 HTMLMinifier 压缩后一般会缩小 40%，更别文本文件经过 Gzip 压缩后一般会缩小到原来的三分之一，但是压缩更像是一种亡羊补牢的方式。与其说优化，不说从开发伊始就应该将性能纳入考虑之中。

我的博客使用的是 [Bulma CSS 框架](https://bulma.io/)。完整打包的 `bulma.css` 文件大小为 237 KiB、Gzip 压缩后为 25.3 KiB；即使对其进行 CSS 压缩、`bulma.min.css` 文件大小依然有 200 KiB、Gzip 后 23.9 KiB。如果我的博客直接引入完整的 `bulma.min.css`，结果可想而知。

因此我选择引入 Bulma 原始的 Sass，在主题开发中按需加载组件，对于臃肿的组件则精简掉不需要的样式、必要时直接重写组件。Hexo 有 [Sass 插件](https://github.com/knksmith57/hexo-renderer-sass) 和 [CSS 压缩插件](https://github.com/hexojs/hexo-clean-css)，因此我可以直接将 CSS 编译和压缩集成在 Hexo 的 workflow 之中。我博客的 `style.css` 大小只有 25.2 KiB、Gzip 后只有 6.4 KiB，是原始的 `bulma.css` 的十分之一。

同样的，在编写 HTML 模板的时候我也经过精心的组织标签和结构。相比其它主题，我的主题生成的 HTML 文件体积更小、但是能够展示的信息量却更大。

更小的 HTML 和 CSS 不仅减少了传输流量，由于浏览器不再需要解析无用的 DOM 和 CSS Rules，网站整体的渲染性能也得到了提升。

## 静态资源加载优化

当压缩成为常态以后，以什么顺序将静态资源发送给用户的浏览器就愈发重要了。

如果你看过 Cloudflare 发布「Enhanced HTTP/2 Prioritization」功能的博客「[Better HTTP/2 Prioritization for a Faster Web](https://blog.cloudflare.com/better-http-2-prioritization-for-a-faster-web/)」的话……什么？你还没有看过？快去看！如果你实在不想看的话，直接看最重要的结果就好了：

![](https://pic.skk.moe/blog/how-to-make-a-fast-blog/load-asset-compare.gif)

> 由于各个浏览器加载各个资源时分配的带宽和优先级不同，因此在相同的弱网环境下，用户使用 Chrome 时打开网页的速度最快，Firefox 次之，Safari 和 Edge with EdgeHTML 最慢。

秉持着「我永远比浏览器聪明」的理念，我一定要 ~亲自部署~ 介入控制所有资源的加载顺序。

### 确保首屏资源优先级

为了避免 Blocking Script 阻碍页面加载和渲染，我将所有不那么重要的 JS 全部使用异步加载，确保首屏资源的最高优先级；对于直接影响渲染的 JavaScript（如 [暗色模式](https://blog.skk.moe/post/hello-darkmode-my-old-friend/) 相关）则是直接内联在 HTML 的 `<head>` 中，Parse HTML 阶段就可以执行；媒体资源如图片、第三方资源如 Disqus、CodeSandBox 则是全部做了 lazyload。因此我保证了不论在任何浏览器上加载的首屏资源只有 HTML 和 CSS，不论浏览器如何为分配带宽和优先级都不会显著影响白屏时间。

### 延迟非关键资源加载

确保非关键资源不会和首屏资源抢带宽的方法就是 lazyload。

我使用的库是 [`vanilla-lazyload`](https://github.com/verlok/vanilla-lazyload)，支持图片、`background-image`、iframe 的 lazyload。了解更多关于如何使 lazyload 的图片能够被 RSS 阅读器和爬虫抓取、以及如何避免 lazyload 的图片加载导致的 Layout Shift、Reflow 和重绘的内容，请查看我的另一篇文章「[图片 lazyload 的学问和在 Hexo 上的最佳实践](https://blog.skk.moe/post/img-lazyload-hexo/)」。至于笨重的 Disqus 评论，我也启用了 lazyload 策略，可以参考我的这篇文章「[使 Disqus 不再拖累性能和页面加载](https://blog.skk.moe/post/prevent-disqus-from-slowing-your-site/)」。

### 加速 CSS 递送

HTML 文档本身自然是优先级最高的资源、也是最早加载的资源。为了确保作为首屏资源的 CSS 也能尽快到达浏览器，我还使用了 HTTP/2 Server Push，当浏览器请求 HTML 时将 HTML 和 CSS 的响应一起发送给浏览器，从而节省 1 个 RTT。你可以查看我的另一篇文章「[静态资源递送优化：HTTP/2 和 Server Push](https://blog.skk.moe/post/http2-server-push/#HTTP-2-Server-Push)」中的「HTTP/2 Server Push」和「HTTP/3 Server Push」章节了解更多关于 Server Push 的内容。

### 确保第三方 JavaScript 异步加载

`<script>` 标签的 `async` 和 `defer` 属性的兼容性其实不差（`async` 属性兼容性为 IE 10+，`defer` 属性的兼容性为 IE 6+）。如果你还不知道这两个属性有什么用，看看下图就知道了。

![](https://img10.360buyimg.com/ddimg/jfs/t1/181544/4/22284/13259/62579dadEdfbf0208/b64d2bcd1d022d01.png)

上图描绘了一个规范定义的理想状况：

-   `defer` 属性即当 HTML 解析到时开始加载，同时一定会在 DOMContentLoaded 事件触发前执行，因此本质上也是 Blocking Script。
-   `async` 属性即当 HTML 解析到时开始加载，然后无视 DOM、渲染、Load 事件，只要加载一完成就会开始执行。

规范很美好，但是现实很残酷：

-   虽然 `<script defer>` 兼容 IE 6+，但是直到 IE 10 之前，`defer` 属性的实现其实 [相当 Buggy](https://github.com/h5bp/lazyweb-requests/issues/42)，包括是否加载、加载顺序、是否执行、执行顺序等一系列问题
-   Firefox 可能会在 DOMContentLoaded 事件触发后才开始执行 `<script defer>` 中的脚本，参见 [Bugzilla #688580](https://bugzilla.mozilla.org/show_bug.cgi?id=688580)
-   由于 `defer` 不是一个合法的 XHTML 属性，因此对于 DOCTYPE 声明了 XHTML 的页面，WebKit 会无视 `script defer` 并将其视为一个同步的 Blocking Script，参见 [Chormium Bug #611136](https://bugs.chromium.org/p/chromium/issues/detail?id=611136) 和 [Chormium Bug #874749](https://bugs.chromium.org/p/chromium/issues/detail?id=874749)，这点其实还好

各个浏览器对于 `<script defer>` 的实现是如此大相径庭而又如此令人大跌眼镜，以至于不使用 `<script defer>` 已经成为了最佳实践。为了确保 JavaScript 只在 DOMContentLoaded 事件触发之前执行？直接把 `<script>` 标签放在 `</body>` 之前吧！

`<script async>` 的实现也好不到哪里去：

-   `<script async>` 依然会阻碍 window 的 Load 事件
-   Blink 会将 `<script async>` 的加载优先级会被提高

对于大部分现代浏览器来说，确保异步加载的做法其实是在操作 DOM 动态插入 `<script async>` 标签：

```
function loadScript(url, cb, isMoudule) {
  var script = document.createElement('script');
  script.src = url;
  if (cb) script.onload = cb;
  if (isMoudule) script.type = 'module';
  script.async = true;
  document.body.appendChild(script);
}
```

对于博客依赖的外部脚本（如 Vanilla-Lazyload 库和 [Cloudflare Workers Async Google Analytics](https://blog.skk.moe/post/cloudflare-workers-cfga/)），我都是使用上述的 `loadScript` 函数加载。为了改善在现代浏览器上的性能，我还使用 `<link rel="preload" as="script">` 标签，以较低的优先级提前加载脚本。相比直接使用 `<script async>`，动态插入脚本和 `preload` 的组合可以避免阻碍 Load 事件。

## 避免 JavaScript 阻碍页面渲染

虽然将同步 JavaScript 放在 HTML 片段的后面、`</body>` 前面，但是几乎在所有浏览器上，DOM 渲染总是在同步的 JavaScript 执行完成之后才渲染。你可以用下面 HTML 片段做个测试：

```
<!DOCTYPE html>
<html>
<head></head>
<body>
  <h1>Sukka</h1>
  <p>with a Big Foxtail</p>
  <p id="js-complete"></p>
  <script>
    const start = new Date();
    while(new Date() - start < 10000) {}
    document.getElementById('js-complete').textContent = 'JavaScript Executed.';
  </script>
</body>
</html>
```

浏览器会持续白屏十秒钟，之后「Sukka with a Big Foxtail」和「JavaScript Executed.」字样才会显示出来。如果这是实际生产环境中的网站，这十秒内用户除了白色的屏幕以外什么都看不见，毫无疑问他们会直接关掉页面。

这里不介绍各个浏览器的渲染机制，只简单介绍一个行之有效的 Hack，将耗时函数包在一个 `setTimeout(() => {}, 0)` 之中，即可将耗时函数踢出主调用栈、扔到 Event Loop 的末尾：

```
<!DOCTYPE html>
<html>
<head></head>
<body>
  <h1>Sukka</h1>
  <p>with a Big Foxtail</p>
  <p id="js-complete"></p>
  <script>
    setTimeout(() => {
      const start = new Date();
      while(new Date() - start < 10000) {}
      document.getElementById('js-complete').textContent = 'JavaScript Executed.';
    }, 0);
  </script>
</body>
</html>
```

现在「Sukka with a Big Foxtail」会直接显示在页面上，「JavaScript Executed.」字样则会在十秒钟之后显示出来。

## 优化访问统计

不论是 Google Analytics、百度统计还是自建 Matomo 访问统计，都需要引入一段体积不小的外源 JS。而我只需要统计页面标题、URL、访问来源等基本信息即可，因此我实现了一个简单的统计后端、部署在 Cloudflare Workes 上，异步地将数据发回 Google Analytics。网站只需要引入一段不超过 1 KiB 的 JavaScript 文件，丝毫不影响页面性能。

你可以在我的「[使用 Cloudflare Workers 加速 Google Analytics](https://blog.skk.moe/post/cloudflare-workers-cfga/)」这篇文章中了解更多相关内容。

## 改善 DOM 操作性能

由于我的博客是静态的，因此很多可以交给在后端的任务只能被放在前端执行。比如「本文更新于 xx 天之前，文中所描述的信息可能已发生改变」的横幅会在距离文章最后更新日期 180 天后展示，这一功能，动态博客可以直接在后端输出最终 DOM，而静态博客只能在浏览器内比对更新时间和当前时间、然后判断是否将隐藏的横幅显示出来。

除此以外，我的博客涉及 DOM 操作的功能还有移动端下文章页面的文章目录（点击屏幕右下角的按钮展开文章目录、接着点击屏幕任意区域可关闭文章目录）、以及暗色模式的切换。触发这些功能时都会引起重绘和回流。

因此，我将这些 DOM 操作全部包装成函数，然后作为回调函数传入 `window.requestAnimationFrame`。这本来是优化动画的函数 —— 浏览器会在页面下一次绘制时执行回调。

效果妙不可言，切换暗色模式涉及到页面数百个元素的 `color`、`background-color` 属性更改，原本点击按钮后需要 150ms 才做出响应，现在只要 77ms。

关于 `window.requestAnimationFrame` 可以参考 [MDN 上的文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)，如果你需要编写一个涉及数十上百个 DOM 操作的动画，可以考虑使用 [fastdom](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame) 这个库。

## 优化 CSS 渲染路径

通常来说，计算样式的第一步是创建一组匹配选择器，即浏览器计算出给指定元素应用哪些类、伪选择器和 ID；第二步涉及到从匹配选择器中获取所有样式规则，并计算出此元素的最终样式。

与外行人想象中的不同，CSS 选择器的匹配顺序并不是从左往右、而是从右往左。但是应对复杂的 CSS 选择器规则时，不论哪种查询顺序都捉襟见肘。如果对 CSS 选择器规则进行优化，可以大大减少浏览器的计算量。

> 从右往左的匹配顺序可以尽早过滤无关样式规则、从而有效地避免回溯。如果想要知道什么是回溯，可以参考我翻译的「[Cloudflare 在 2019 年 7 月 2 日宕机的技术细节](https://blog.skk.moe/post/how-regex-caused-cf-global-502/)」一文中的「附录：关于正则表达式回溯」章节。

如果给出如下的 CSS 选择器：

```
.box:nth-last-child(-n+1) h1 .title {
  /* styles */
}
```

那么浏览器要做的事情是：

-   寻找所有包含 `title` 类的元素
-   在这些元素中，筛选出其父元素是 `<h1>` 标签的元素
-   接着再筛选出父元素是其父元素中第奇数个子元素的元素
-   对经过筛选后的元素应用样式

头昏脑涨？毫无疑问，这种选择器规则的性能十分低下。

Steve Souders（Google Web 工程师，曾经写过三本 Web 性能为主题的 O'Reilly）曾于 2009 年总结了 CSS 选择器性能排行：

1.  ID 选择器（`#id`）
2.  类选择器（`.className`）
3.  标签选择器（`div` `h1` `p`）
4.  相邻选择器（`h1+p`）
5.  子选择器（`ul > li`）
6.  后代选择器（`li a`）
7.  通配符选择器（`*`）
8.  属性选择器（`a[rel="external"]`）
9.  伪类选择器（`a:hover` `li:nth-child`）

由于从右往左匹配，因此即使是同一类型的选择器之间的性能也有差异 —— 后代选择器中，最右侧是 ID 的选择器的性能就优于最右侧是类的选择器。

> 正是在 10 年前、硬件和软件性能都捉襟见肘的时代，才诞生出了像雅虎 35 条军规和 CSS 选择器性能排行这样的性能优化原则。即使 10 年过去了，硬件性能已经突飞猛进，这些传统的性能优化依然不能落下。

基于以上基础，我将不必要的后代选择器精简为 ID 选择器和类选择器、将所有的 `:not(:last-child)` 拆分为不包含伪类和使用 `:last-child` 伪类的两条样式。经过重写后，我不仅进一步精简了 CSS 的体积，渲染时 Style & Layout 用时还减少了 150ms。

## 后记

曾经我在 V2EX 看到过一个小白，用宝塔面板和 WordPress 建站、装了十几个老掉牙的 WP-Rocket 之类的插件 “优化” 和 “防御” 网站，还写了十篇系列文章分享到 V2EX，自然被数十个人 diss 了（包括我在内）。但是他反驳我的话却让我印象深刻：

> 你的博客是很快，但是你从来不写这方面的教程，只写了一篇文章 [说你的博客有多快](https://blog.skk.moe/post/how-fast-is-my-blog/)。像你这样不分享教程的人有什么资格看不起我们这些小白？

之前我的文章更多是零散的介绍一些技巧，这次我将之前的文章都串起来，融会贯通写一篇集大成者以飨读者。