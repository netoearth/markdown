**Matrix 首页推荐** 

[Matrix](https://sspai.com/matrix) 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。我们会不定期挑选 Matrix 最优质的文章，展示来自用户的最真实的体验和观点。   
文章代表作者个人观点，少数派仅对标题和排版略作修改。

___

## 前言

毋庸置疑，如今的 Chrome 及其他基于 Chromium 的 Blink 内核浏览器因其速度快、网页兼容性好的优点，以绝对的优势占据了浏览器市场的主导地位。但与此同时，因为 Chrome 与 Chromium 被一家号称「不作恶」但处处作恶的公司所拥有，所以经常被诟病存在隐私问题。许多想要逃离 Chrome 统治的人都转向了更加自由的 Firefox 浏览器，尽管其母公司 Mozilla 近年来也是骚操作不断，但 Firefox 仍然是除 Chromium 的 Blink 和 Safari 的 WebKit 外唯一选择。

我曾经也很喜欢用 Chrome（或者说是 Chromium），使用 ArchLinux 作为主力系统的我，官方软件源中使用开源 Chromium 代码编译出的 Chromium 浏览器在当时体验基本与谷歌官方的闭源 Chrome 一致。促使我转向使用 Firefox 的契机开始于一年多前，[谷歌限制了第三方浏览器对其数据同步 API 的使用](https://blog.chromium.org/2021/01/limiting-private-API-availability-in.html)，这样一来各家发行版自行编译的 Chromium 浏览器都失去了账户同步功能，这在当时引起了很多不满。从那时开始，我就开始尝试使用 Firefox 替代 Chromium 作为我的主力浏览器，并最终彻底淘汰掉了 Chromium。

无独有偶，[谷歌又宣布将会在 2022 年底彻底禁用 Manifest V2 规范](https://www.theregister.com/2022/09/08/ad_blockers_chrome_manifest_v3/)（后来又推迟了半年），作为替代者的 Manifest V3 规范，用谷歌的话说是为了「保护用户隐私」，限制了浏览器扩展的很多权限，其中影响最大的是禁用了 WebRequest API，绝大多数去广告相关的扩展都要依赖这个 API 来工作，谷歌虽然开放了新的 API 供扩展使用，但使用了新 API 的扩展拦截广告能力会大打折扣。另外一边，[Firefox 则保证自己的 Manifest V3 规范将会保留对 WebRequest API 的兼容](https://blog.mozilla.org/addons/2022/05/18/manifest-v3-in-firefox-recap-next-steps/)。

综合以上两个背景，相信会有很多人对 Firefox 产生兴趣，或者转向使用 Firefox。于是我写下这篇文章，分享一下我在使用 Firefox 的这一年多来学习到的配置方法与使用技巧。

___

注：没有特别说明的话，本文提及的 Firefox 指的是 Mozilla 公司的原版 Firefox 浏览器，而不是国内谋智公司的所谓「特供版」火狐浏览器。

## 下载与安装

和 Chrome 的版本发行规律类似，官方的 Firefox 分为 [稳定版](https://www.mozilla.org/zh-CN/firefox/new/)、[测试版（beta）](https://www.mozilla.org/zh-CN/firefox/channel/desktop/#beta)、和 [每夜版（nightly）](https://www.mozilla.org/zh-CN/firefox/channel/desktop/#nightly)，但和 Chrome 不同的是，Firefox 还有长期支持的 [ESR 版本](https://www.mozilla.org/zh-CN/firefox/enterprise/)（版本较落后，但会长期维护）和面向开发者的 [Developer Edition](https://www.mozilla.org/zh-CN/firefox/developer/)（基于 beta 版，多了一些开发者工具）。

另外，因为是开源项目，Firefox 还存在很多非官方的 fork（分叉版），其中值得一提的有以下几个：

-   [Tor Browser](https://www.torproject.org/download/)：大名鼎鼎的 Tor 浏览器，基于 Firefox ESR 版本，默认修改了很多高级设置以增强隐私性和安全性，同时集成了 Tor 网络增强匿名性，不过因为网络环境的原因，在国内几乎不可用;
-   [Librewolf](https://librewolf.net/)：基于稳定版 Firefox，移除了原版 Firefox 中不严格符合自由软件定义的代码，禁用了遥测、数据收集和 DRM，修改了很多高级设置以增强隐私性和安全性，并默认集成了 uBlock Origin 扩展；
-   [GNU IceCat](https://www.gnu.org/software/gnuzilla/)：GNU 的 Firefox，出于回避使用专有品牌名称考虑换了名字，移除了所有的不严格符合自由软件定义部分，并默认集成了一些 GNU 自己开发的扩展以禁用网页中的不严格自由部分，从而保证「完全的自由」。IceCat 不提供 Windows 和 MacOS 版本；
-   [Waterfox](https://www.waterfox.net/download/)：基于 Firefox ESR，宣称隐私友好，但是被卖给了一家广告公司；而且它的版本发行比较奇怪，分为 Classic（基于超级古老的 Firefox ESR 56，但是仍然在维护更新）和 Current 版本（基于最新的 ESR 版本），版本号都以 G 为前缀（应该代表的是 Generation），例如 G5 对应 Firefox ESR 102，G4 对应 Firefox ESR 91，G3 对应 Firefox ESR 78。

这么多的版本，到底应该选哪个呢？我个人的建议是：如果你是个隐私狂人，想要最高等级的隐私保护，推荐直接用 Tor Browser 和 Librewolf，并且下面的配置部分也可以跳过，因为这两个浏览器已经提供了最佳保护设置；如果你是自由软件运动和 Richard Stallman 的坚定支持者，那么这里推荐 IceCat。不过这两种人一般都不需要我这种教程了吧。

![](https://cdn.sspai.com/2022/11/07/article/e346a1c0d1aa5ea6555e5e36966947cf?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

**对于普通人，想要保证正常的网页浏览体验，我还是比较推荐用官方版本的 Firefox。**

不过值得一提的是，Mozilla 的骚操作之一，直接从官网下载的 Firefox 的安装包都含有[唯一识别码](https://www.ghacks.net/2022/03/17/each-firefox-download-has-a-unique-identifier/)，可以被用来追踪用户，如果介意的话，可以去 Mozilla 的 [FTP 站点](https://ftp.mozilla.org/pub/firefox/) 下载安装包。

对于 Linux 用户，由于 Firefox 几乎存在于所有发行版的官方软件仓库中，甚至是绝大多数发行版的系统预装软件，没有预装的也可以使用包管理器快捷安装，除了软件本体，还需要安装对应的语言包，我使用的 Archlinux，官方稳定版 Firefox 的简体中文语言包包名是 `firefox-i18n-zh-cn`，其他发行版请自行寻找。

## 推荐配置

### 备份配置文件

首先，数据安全最重要，在 Firefox 地址栏中输入 `about:profiles` 再回车，就可以看到下图的界面，正常情况下这里只有一个配置文件，如果没有重命名过的话，配置文件名字是 default，同时会有两个文件夹，一般是一串乱码加配置文件名。第一个文件夹存储着浏览器的各种数据，比较重要，第二个文件夹存储的是缓存文件，可以不用关心。点击旁边的 「打开目录」 就可以在文件管理器用打开文件夹。我们要做的是将第一个文件夹复制到别处，如果不小心把浏览器折腾坏了，不知道如何恢复的话，只需删除配置文件夹，并将之前备份的配置文件夹复制回去，就可以恢复到之前的状态了。

![](https://cdn.sspai.com/2022/11/07/article/d0d12567ecbf218893e77263fad419d9?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

另外你也可以点击上方的 「创建新配置文件」 再创建一个新的配置文件夹并设置为默认配置文件。这样下次打开 Firefox 就会默认读取新配置文件，不会影响到旧的配置文件。

### 个性化设置

对于外观每个人喜好不同，我只在这里分享一下我个人的配置，仅供参考。

首先是主题配色，在最新的 Firefox 106 版本中，新添加了 Colorways 主题配色，在第一次启动浏览器时弹出的向导界面中就会推荐。

![](https://cdn.sspai.com/2022/11/07/article/94b88c412d10aa4a83003c012a06e62e?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

后续也可以点击浏览器左上角的 logo 进入 Firefox view 界面或地址栏输入 `about:addons`进入扩展管理页面修改 Colorways 配色，此外 Firefox 也内置了好几个主题配色，想要更多主题配色可以通过 Firefox 的 [扩展商店](https://addons.mozilla.org/zh-CN/firefox/themes/) 下载。我个人使用的是 [Dracula Dark Theme](https://addons.mozilla.org/zh-CN/firefox/addon/dracula-dark-colorscheme/)。

![](https://cdn.sspai.com/2022/11/07/article/20a06eb3742d5eea6a12cd4dc7358c56?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

![](https://cdn.sspai.com/2022/11/07/article/919491331d6c9da7f5177e42b0d6c451?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

然后是浏览器的页面布局，鼠标在浏览器标签栏空白处右键，在右键菜单中点击 「定制工具栏」，便可进入定制 Firefox 界面，工具栏上不想要的元素，只需用鼠标将其拖动放到中间区域，相对的，想要添加的元素也只需鼠标拖动到想要的位置，修改完点击右下角的完成即可。我个人习惯去掉地址栏两边的空白，在左上角添加一个主页按钮，将不常用的扩展放入折叠菜单，还有前文提到的的 Firefox view，这个功能是在 106 版本新加的，方便多设备同步，但是默认在左上角感觉很不协调，我就给挪到右上角了。

![](https://cdn.sspai.com/2022/11/07/article/84b08f758e75b84f1ce0f2572f14e522?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

最后是主页内容，在地址栏输入  `about:preferences` 回车可以进入设置页面，左侧点击主页，在 Firefox 主页内容部分取消勾选除了网络搜索的所有部分，这样打开浏览器就是一个清爽的主页了。

### 隐私设置

虽然 Firefox 经常以隐私保护为卖点宣传，但官方版本的 Firefox 在不做任何设置的情况下，并没有比它的竞争对手 Chrome 强上太多，一样会有遥测，一样会有数据收集。但 Firefox 的优势在于它可配置的选项更多，遥测和数据收集也都可以通过一定的手段关闭。本节将会介绍一些基本的隐私相关设置项，但具体如何配置要取决于个人的使用场景。

-   「设置」>「常规」中，找到 「采用数字版权管理（DRM）的内容」：一些音频视频网站，比如 Spotify 和 Netflix ，提供的内容可能存在版权保护，经过了数字加密（EME），浏览器想要播放这类内容，就需要安装 Widevine 插件，打开这个选项浏览器就可以自动安装 Widevine 插件。但是问题在于，DRM 与开源自由的理念相悖，如果介意这一点，可以将这个选项关闭。
-   「启用基于 HTTPS 的 DNS」：启用后所有的 DNS 查询都将会由 DNS-over-HTTPS 提供商处理，而不会把查询结果泄漏给本地运营商。Firefox 默认内置了 Cloudflare 和 NextDNS 两个提供商，你也可以自行添加其他的服务商。需要注意的是，DNS-over-HTTPS 的速度要明显慢于普通 DNS，开启后可能会拖慢网页加载速度。
-   「设置」>「搜索」，选择你常用的的搜索引擎，Firefox 默认内置了 google、百度、维基百科、Bing 和 DuckDuckGo 几个搜索引擎，想要添加其他的搜索引擎，可以从官方扩展商店以 [扩展](https://addons.mozilla.org/zh-CN/firefox/extensions/category/search-tools/) 的形式安装，或者在搜索引擎网址主页右键地址栏添加。

![](https://cdn.sspai.com/2022/11/07/article/8ed8515cdeb5895b3540f47d5cbafe6c?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

-   「搜索建议」：建议关闭，因为搜索建议是由搜索引擎提供的，有可能被用来收集隐私信息。
-   「设置」>「隐私与安全」，找到增强性隐私保护，默认的标准模式不会拦截正常窗口的跟踪性内容，这里推荐选择为严格，虽然 Firefox 会提示你 「可能导致某些网站异常」，但我基本还没遇到过，如果真的遇见了网站异常，也可以添加例外。
-   「向网站发出『请勿跟踪』信号，示明您不想被跟踪」：建议选择为 「一律发送」，遵循这一设置的网站将会停止跟踪用户信息，但不是所有网站都会遵循这一设置，不过聊胜于无。
-   「Cookie 和网站数据」：如果不想让浏览器保存 Cookie 和网站数据，可以勾选「关闭 Firefox 时删除 Cookie 与网站数据」，再在 「管理例外」 中配置不想被删除数据的网站，但是这个管理例外我个人觉得比较难用，网址不支持通配符，我更喜欢用 Cookie Auto Delete 扩展，详见下文 [扩展推荐](https://sspai.com/post/76688#%E6%89%A9%E5%B1%95%E6%8E%A8%E8%8D%90) 部分。
-   「登录信息与密码」和 「表单与自动填写」：我个人使用的是单独的密码管理器，所以这项全关了，如果想要用 Firefox 管理密码，可以打开。
-   「历史记录」：如果担心电脑被盗后浏览历史泄漏，可以选择不记录历史，我个人觉得记录历史还是挺方便的。
-   「Firefox 数据收集与使用」：建议全部取消勾选，不用多说。
-   「欺诈内容和危险软件防护」：可能有点反直觉，如果想要保护隐私，建议把这项关闭，因为「危险与诈骗内容」是提交给 Google 识别的。看来在这里隐私和安全不可兼得吧。
-   「查询 OCSP 响应服务器，以确认证书当前是否有效」：建议打开，校验 SSL 证书是否有效，增加安全性，但如果网络环境较差 OCSP 服务器连接不稳定，可能会拖慢网页加载速度，甚至会无法加载网页。
-   「HTTPS-Only 模式」：建议打开，现在绝大多数网站都已经支持了 HTTPS，这项打开后可以将未加密，不安全的 HTTP 升级为更安全的 HTTPS，对于极少数不支持 HTTPS 的网站，可以手动退回 HTTP。

如果你看到了这里并且跟随着我的设置一步步来，那么在尽可能少地影响基本的网页浏览体验的前提下，你的浏览器已经比绝大多数人的浏览器更加隐私与安全了。如果你此时已经十分满意，大可以直接退出这篇文章；如果你想要更高层面的隐私保护，并且做好了会影响网页浏览体验的心理准备，那么可以继续往下看。

### 深入高级设置

在 Firefox 的地址栏输入 `about:config` 再回车，会弹出一个警示页面，点击接受风险并继续，就可以进入高级设置了。

比如说要是我想禁用 `about:config` 的警示页面，可以在搜索框中输入 `browser.aboutConfig.showWarning`，只有一个结果，点击右边的箭头，true 变为 false。再重新打开 `about:config`，你就会发现，警示页面没有了。

![](https://cdn.sspai.com/2022/11/07/article/733cdd2e741c7eedb9b3273c34ab57de?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

让我们暂时先把 `browser.aboutConfig.showWarning` 调回 `true`，也就是不禁用警示页面。还记得 [前文](https://sspai.com/post/76688#%E5%A4%87%E4%BB%BD%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)提到的配置文件目录吗，地址栏输入 `about:profiles`，找到当前的配置文件目录，打开目录，在里面新建一个文本文档，内容如下：

```
user_pref("browser.aboutConfig.showWarning", false);
```

将文档重命名为 `user.js`，之后重启浏览器，再次进入 `about:config` 界面，你会发现警示界面又没有了，搜索 `browser.aboutConfig.showWarning`，发现它的值又变成了 false。

至此，你已经对 Firefox 的高级设置有了一定的了解，Firefox 的高级设置既可以在 `about:config` 界面设置，也可以用 `user.js` 文件设置，接下来就可以根据自身需要对 Firefox 的高级选项做修改了。

### Arkenfox

这时你可能会问：`about:config` 里面有那么多配置，怎么知道要改哪一个？要怎么改？事实证明，永远不要重复造轮子，在 GitHub 上，有一个名为[Arkenfox](https://github.com/arkenfox/user.js) 的项目。Arkenfox 是一个积极维护的，旨在增强 Firefox 隐私性与安全性的 `user.js` 列表，禁用了绝大多数的遥测项目，并修改了许多隐私相关的设置项，用户只需下载项目提供的 [user.js](https://raw.githubusercontent.com/arkenfox/user.js/master/user.js) 文件，放进配置文件夹里面，重启浏览器，就大功告成了。

**修改这些高级选项极有可能导致网页故障，所以推荐再安装一个备用浏览器，或者在** `**about:profiles**` **界面中新建一个配置文件。**

### user-overrides.js

这时你可能又会问，这个 `user.js` 配置不可能适合所有人吧，如果需要修改怎么办？如果日后 Arkenfox 的配置项有变动，又要怎么跟随项目更新呢？

这点 Arkenfox 的维护者也想到了，对于想要自行修改的配置项，不推荐直接修改 `user.js` 文件，Arkenfox 提供了一个 updater 脚本，Linux 和 MacOS 用户可以使用 [updater.sh](https://raw.githubusercontent.com/arkenfox/user.js/master/updater.sh)，Windows 用户可以使用 [updater.bat](https://raw.githubusercontent.com/arkenfox/user.js/master/updater.bat)，将需要修改的配置项保存进文本，重命名为 `user-overrides.js` ，同 updater 脚本一起放进配置文件夹，运行脚本，会自动下载最新的 `user.js`，并将其与 `user-overrides.js` 合并，保存为新的 `user.js`文件。以后如果 `user-overrides.js` 有改动，或者要更新 Arkenfox，只需重新运行 updater 脚本即可。

另外，Arkenfox 还提供了 [perfsCleaner.sh](https://raw.githubusercontent.com/arkenfox/user.js/master/prefsCleaner.sh) 和 [perfsCleaner.bat](https://raw.githubusercontent.com/arkenfox/user.js/master/prefsCleaner.bat)脚本，推荐在更新 Arkenfox 之前运行，这个脚本会将所有配置项重置，防止出现玄学问题。

你可以在 [这里](https://github.com/arkenfox/user.js/wiki/3.2-Overrides-%5BCommon%5D) 和 [这里](https://github.com/arkenfox/user.js/issues/1080) 了解常用的 overrides，以下是我推荐修改的 overrides：

**浏览器启动页面修改：**0 为空白页面，1 为主页，2 为最后浏览的页面，3 为恢复先前的浏览状态。Arkenfox 修改为了 0，我个人修改为了 1；

```
user_pref("browser.startup.page", 1);
```

**浏览器主页修改：**Arkenfox 修改为了 `about:blank` 空白页，我个人修改为了 `about:home`。

```
user_pref("browser.startup.homepage", "about:home");
```

**新标签页修改：**Arkenfox 修改为了空白页，我个人修改回了默认主页。

```
user_pref("browser.newtabpage.enabled", true);
```

**设置网页的偏好语言：**Arkenfox 修改为了英语，我改回了默认；

```
user_pref("intl.accept_languages", "zh-CN, zh, zh-TW, zh-HK, en-US, en");
```

**强制浏览器语言为英语：**Arkenfox 修改为了 `true`，我改回了 `false`；

```
user_pref("javascript.use_us_english_locale", false);
```

**禁用 IPv6：**arkenfox 修改为了 `true`，若有访问 ipv6 网站的需求，可修改为 `false`；

```
user_pref("network.dns.disableIPv6", false);
```

**使用地址栏进行网络搜索：**`false` 为禁用，地址栏只可以用于解析 URL，此后可在「设置」>「搜索」，勾选「添加搜索栏到工具栏」，使用单独的搜索栏进行网络搜索。`true` 为启用，指输入的不是正确 URL 时会跳转到搜索引擎进行搜索，缺点是有可能将输入的 URL 泄漏给搜索引擎，我个人还是觉得启用了方便一些；

```
user_pref("keyword.enabled", true);
```

**记录浏览历史：**Arkenfox 修改为了 `false`，我修改回了 `true`；

```
user_pref("browser.formfill.enable", true);
```

**阻止使用不安全加密方式的网站：**会导致某些网站无法访问，若发现有些网站出现问题，可尝将此项设为 false；

```
user_pref("security.ssl.require_safe_negotiation", false);
```

若上一项设置为了 `false`，可选择将此项设为 `true`，若访问使用不安全加密方式的网站时，会在地址栏的锁头图标上显示一个感叹号用于提醒；

```
user_pref("security.ssl.treat_unsafe_negotiation_as_broken", true);
```

**启用 Public Key Pinning：**Arkenfox 设置为了 `2` 强制启用，若使用杀毒软件，可能会受到影响，如果是的话，可以改回默认 `1`；

```
user_pref("security.cert_pinning.enforcement_level", 2);
```

**禁用跨站标识：**比如你从少数派点击 www.baidu.com 的链接点击访问百度网站，则百度可以识别到此次访问来自 `sspai.com`，这两项设置为 `2` 可禁止此行为，但会影响到很多网站的功能，比如一些视频网站无法播放视频，若影响到了使用，可改为 `0`；

```
user_pref("network.http.referer.XOriginPolicy", 0);
user_pref("network.http.referer.XOriginTrimmingPolicy", 0);
```

**禁用 DRM：**在前文提及过，有需要可启用；

```
user_pref("media.eme.enabled", true);
```

**在关闭浏览器时删除浏览历史：**有需要可关闭；

```
user_pref("privacy.sanitize.sanitizeOnShutdown", false);
```

若上一项设置为了 `true`，即自动删除历史，通过下面几项可选择删除的类别；

```
user_pref("privacy.clearOnShutdown.cache", true); // 缓存
user_pref("privacy.clearOnShutdown.downloads", true); // 下载记录
user_pref("privacy.clearOnShutdown.formdata", true); // 网站数据
user_pref("privacy.clearOnShutdown.history", true); // 历史
user_pref("privacy.clearOnShutdown.sessions", true); // 打开的标签页状态
user_pref("privacy.clearOnShutdown.cookies", true); // Cookies
user_pref("privacy.clearOnShutdown.offlineApps", true); // 网站数据
```

阻止指纹识别，网站可以识别到访问者的浏览器版本，系统版本，语言等等，并利用这些数据建立起可识别的指纹以便进行跟踪，打开此项可以伪装部分参数，使网站更加难以建立可识别的指纹，此功能极有可能影响到网页浏览，并会会影响到很多浏览器的功能，比如时区被锁定在 UTC0，网站总是偏向于使用亮色主题等等，若介意的话，可以关闭。

```
user_pref("privacy.resistFingerprinting", false);
```

**RFP letterboxing：**打开此项会使网页外部有一个边框（实际上是与当前窗口大小最接近的「常见」窗口大小），进一步阻止网站通过屏幕分辨率采集设备指纹，介意的话可以关闭。

```
user_pref("privacy.resistFingerprinting.letterboxing", false);
```

**禁用 WebGL：**`true` 为禁用，会极大地影响性能，建议设置为 `false`；

```
user_pref("webgl.disabled", false);
```

**启用 http3：**http3 使用 udp 连接，理论上更快，已知目前部分 google 的网站和使用 Cloudflare CDN 的网站都使用了 http3，但实际上很多运营商会对 udp 流量限速，导致 http3 的实际体验更差，建议禁用。

```
user_pref("network.http.http3.enable", false);
```

你也可以订阅 Arkenfox 的 [RSS Feed](https://github.com/arkenfox/user.js/releases.atom) 获取 Arkenfox 的更新日志。

### 扩展推荐

通过修改以上的参数，基本可以避免出现绝大多数问题，若想要更加舒适的浏览体验，还推荐安装一些扩展。

**若是比较看重隐私的话，则安装扩展应该遵循最少原则，只安装自己需要的扩展，尽量不要安装多余的扩展，因为安装的扩展越多，越容易建立起可识别的指纹。**

**uBlock** [**Origin**](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin)**：**按照 uBlock Origin 开发者的说法，这个扩展并不是广告拦截扩展，而是一个高效地请求过滤工具，只是恰好起到了广告过滤的作用。不过要是推荐 Firefox 的去广告扩展，还是首推 uBlock Origin，相比于其他的的去广告扩展，它支持更多的过滤规则，且占用的资源极低。

uBlock Origin 已经内置了很多去广告规则，可以根据个人需求自行勾选.除了默认启用的规则，我个人推荐：

-   在 「隐私」 分类中勾选 AdGuard Tracking Protection 和 AdGuard URL Tracking Protection，这两个规则提供了很多隐私增强相关的过滤项；
-   [EasyList China](https://easylist-downloads.adblockplus.org/easylistchina.txt) 和 [CJX's Annoyance List](https://raw.githubusercontent.com/cjx82630/cjxlist/master/cjx-annoyance.txt)：两个针对中文互联网环境的规则；
-   [Actually Legitimate](https://raw.githubusercontent.com/DandelionSprout/adfilt/master/LegitimateURLShortener.txt) URL [Shortener Tool](https://raw.githubusercontent.com/DandelionSprout/adfilt/master/LegitimateURLShortener.txt)：这个规则可以去除 URL 中的一些无用的跟踪内容，有点类似于 [Clear](https://addons.mozilla.org/en-US/firefox/addon/clearurls/)URL[s](https://addons.mozilla.org/en-US/firefox/addon/clearurls/) 这个扩展的功能，所以 ClearURLs 扩展就不需要单独安装了。

另外值得一提的是，由于某些广告商的投诉，在国内网络环境下，Firefox 扩展商店中的一些去广告扩展已经无法安装，扩展商店主页也无法访问，请自行寻找方法，例如部分 Linux 发行版用户可以使用系统的包管理器从软件源安装，比如 Archlinux 可以用 `pacman -S firefox-uBlock-origin` 命令为官方版的 Firefox 安装 uBlock Origin 扩展。

[**CanvasBlocker**](https://addons.mozilla.org/en-US/firefox/addon/canvasblocker)**：**如果你在 user-overrides.js 中禁用了阻止指纹识别，则可以安装这个扩展作为替代，安装完好像也无需进行多余的设置，这个扩展提供了一些指纹保护措施，虽说保护效果不如浏览器原生的阻止指纹识别效果好，但对网站的影响更小。

[**Cookie AutoDelete**](https://addons.mozilla.org/en-US/firefox/addon/cookie-autodelete)**：**顾名思义，自动删除 cookie，安装后在扩展设置页面勾选自动清理，对于不想要清理的网站在表达式列表中添加白名单即可，这个扩展相对于浏览器自带的自动清理功能，优点在于白名单支持通配符，比如想要对 google.com 的所有子域名添加白名单，只需添加一条 \*.google.com 即可。

[**Skip Redirect**](https://addons.mozilla.org/en-US/firefox/addon/skip-redirect)**：**链接自动跳转，一些网站在点击站外链接时，会经过一些中间页面，有时还需手动确认，有时甚至还会拦截它们认为 「危险」的链接，这个扩展可以解决这个问题，直接跳转到最终页面，省去了很多麻烦。

**网页翻译：**从 Chrome 切换到 Firefox 后最大的痛点，就是没有内置翻译功能，虽然 Firefox 有自家的 [翻译工具](https://addons.mozilla.org/en-US/firefox/addon/firefox-translations/)，但是还在测试阶段，且支持的语言很少，我也试过很多翻译扩展，感觉都不太好用，目前在用的是 [TWP](https://addons.mozilla.org/en-US/firefox/addon/traduzir-paginas-web/)，感觉相对来说比较好用，它在地址栏右上角添加了一个类似 Chrome 的翻译按钮，支持整页翻译，不过我很少用，我最常用的还是它的划词翻译功能，网页选取需要翻译的内容，就会弹出一个按钮，翻译选中的内容。

![](https://cdn.sspai.com/2022/11/07/article/da560b4514d3c1c578c0ed4bb82da23b?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

还有一些我自用的扩展，可以根据自己需要安装，大部分大家都比较熟悉，就不一一介绍了。

-   [Tampermonkey](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey) 或 [Violentmonkey](https://addons.mozilla.org/en-Us/firefox/addon/violentmonkey)：安装脚本；
-   [Bitwarden](https://addons.mozilla.org/en-US/firefox/addon/bitwarden-password-manager)：密码管理器；
-   [Dark Reader](https://addons.mozilla.org/en-US/firefox/addon/darkreader/)：深色模式；
-   [uBlacklist](https://addons.mozilla.org/en-US/firefox/addon/ublacklist/)：屏蔽 google 搜索结果中的一些低质量网站；
-   [Vimium C](https://addons.mozilla.org/en-US/firefox/addon/vimium-c)：使用 vim 快捷键浏览网页；
-   [SponsorBlock](https://addons.mozilla.org/en-US/firefox/addon/sponsorblock/)：自动跳过 Youtube 视频中的赞助内容；
-   [Return Youtube Dislike](https://addons.mozilla.org/en-US/firefox/addon/return-youtube-dislikes/)：显示 Youtube 视频的点踩数量。

iOS 端的 Firefox 我没用过，不作评价；而且 iOS 端的浏览器因为 Apple 要求必须使用 WebKit 内核，所以基本上全都是 Safari 的套壳，纠结选择哪个浏览器意义不大。这里主要讨论的是 Android 版本。

Android 版本的 Firefox 被很多人诟病，也确实存在很多问题，比如功能缺失，官方版的 Firefox Android 不能访问 `about:config` 界面修改高级选项，扩展安装也受限制。但是相比于 Firefox，我个人觉得 Android 版本的 Chrome 更加难用，而且完全不支持扩展。

虽然 Android 版 Firefox 支持安装扩展，但是只能安装官方提供的 [少量扩展](https://addons.mozilla.org/en-US/android/)，如果想要安装列表以外的扩展怎么办呢？其实也有办法，Android 版 Firefox 的 [Nightly](https://play.google.com/store/apps/details?id=org.mozilla.fenix) 版本提供了安装额外扩展的方式，不过比较麻烦。

首先，在电脑上访问 [Firefox 扩展商店](https://addons.mozilla.org/)，登录自己的帐号，鼠标放到右上角自己的昵称处，点击 「查看我的收藏集」，若没有收藏集就新建一个，点击进入收藏集，复制网页的 URL，后面会用到。

再回到主页，搜索自己喜欢的扩展，滚动到最下方，选择添加到自己刚才创建的收藏集，

来到 Android 端，打开 Firefox nightly，打开设置，找到最下方 「关于 Firefox」，点击进入，多次点击 Firefox logo，直到提示 「已启用调试菜单」。

回到设置页面，就会发现在原来的 「附加组件」下方多出了一个 「自定义附加组件收藏集」，点开，就会提示输入用户 ID 和收藏集名称和之前复制的收藏集 URL。

打个比方，如果是 `https://addons.mozilla.org/zh-CN/firefox/collections/123456789/name`（只是打比方，并不是真实链接），那么用户 ID 就是 `123456789`，收藏集名称就是 `name`。

填好后，保存重启应用，点开附加组件页面，等它刷新就能发现，想要安装的扩展就已经在列表里面了。不过扩展安装后可能不会正常工作，请自行测试。

![](https://cdn.sspai.com/2022/11/07/b036e6cc6924ddc383ee5de2fcbe3cf9.jpg?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

但是就算是 Nightly 版本的 Firefox，也没办法进入 `about:config` 页面。这就要提到第三方的 Fork 版本了。

首先是 [Iceraven](https://github.com/fork-maintainers/iceraven-browser)，基于 Firefox Android 的开源代码 [fenix](https://github.com/mozilla-mobile/fenix) 构建，你可以从 [Github release](https://github.com/fork-maintainers/iceraven-browser/releases) 下载安装包，默认集成了一个很大的扩展列表，绝大部分常用的扩展都能找到，省去了自己添加收藏集的麻烦了，同时也可以访问 `about:config` 修改高级选项。但是 Iceraven 的维护者对于版本更新很不积极，经常落后官方版本很多。

[Fennec F-Droid](https://f-droid.org/zh_Hans/packages/org.mozilla.fennec_fdroid/) 是 [F-Droid](https://f-droid.org/zh_Hans/) 商店基于 Fenix 代码构建的浏览器，更新进度基本与官方一致，可能稍有落后，构建时移除了非自由部分和遥测，功能与官方版基本一样，不同的是支持进入 about:config 界面修改高级选项，同时也支持自定义附加组件收藏集。值得一提的是，不知什么原因，Fennec 安装扩展后，扩展的设置页面默认都是英文的，有些扩展可以在自己的设置页面修改语言，比如 Tampermonkey，有些就改不了。

虽然 Iceraven 和 Fennec 可以通过 `about:config` 修改高级设置，Arkenfox 修改了上百个选项，照着 `user.js` 在小屏幕上一个个修改很是难受，有没有什么更好的方法呢？

那么这就要提到 [Mull](https://f-droid.org/zh_Hans/packages/us.spotco.fennec_dos/) 了，这个浏览器基于上文提到的 Fennec，但是已经默认集成了 Arkenfox 的高级设置，少数的需要覆盖的选项可以自行修改，省去了很多麻烦。

不过值得一提的是，和 Arkenfox 一样，Mull 默认启用了阻止指纹识别的选项，在是在移动端上开启这一选项后会使界面刷新率锁在 60Hz，这是个已知 bug，如果使用高刷手机且比较介意的话可以关闭。另外我还发现 Violentmonkey 扩展在 Mull 上不能正常工作，Tampermonkey 就可以正常工作，这两个扩展在 Fennec 上都可以正常使用的，不知道是什么原因。

## 结语

以上基本就是我对于 Firefox 的使用心得，其实还有一些没有讲到，比如通过自定义 CSS 文件来修改 Firefox 外观，我没有深入研究过，就不细讲了，这里只分享一个包含很多现成的 CSS 的 [Github Repo](https://github.com/MrOtherGuy/firefox-csshacks)。感兴趣可以自己研究，文章到这里已经接近一万字了，如果文章发出去，真的有人看到这里的话，我很是感激。

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀