**本期作者**

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Ra5libNH8ibg5UNzb2j2bcHp0mmC8y7KcxKEsLMribzXHrwicInibicFp4fxkBEicUCcNhvc0iacQeic4QDRA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

吴世阳

bilibili开发工程师

主要负责 B 站视频云 WasmPlayer 研发，专注于前端多媒体领域

李晓波

bilibili资深算法工程师

主要从事HEVC、AV1以及VVC编码器和解码器的研发和应用

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Ra5libNH8ibg5UNzb2j2bcHp4pTxnyYCp2kYXPVp7UYC2MDUkI5PCva1SDPjxIH3SVKj8ibou6ouplg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1BMf5Ir754Ra5libNH8ibg5UNzb2j2bcHpsm1bG012BiaH0Oakzh9IPWJibCrLET58P4qtyPiayOicEmibbicwum6gebWg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

谭兆歆

bilibili资深开发工程师

负责B站主站前端团队、Web播放团队，专注于Web播放能力和弹幕能力建设

**背景**

B 站是一家把用户体验放在第一位的公司，在 AVC 编码下我们已经做到了不错的视频播放卡顿率控制，为了进一步的降低卡顿率我们开始了对 HEVC 编码的探索。HEVC 是比 AVC 更加高效的视频压缩标准，可以在同等画质下减少 50% 的文件体积，意味着只要原先一半的带宽下就能流畅播放 HEVC 视频，在更差的网络环境下依旧能得到不错的用户体验。

目前业内 HEVC 编码主要用于各类移动端设备上，Web 端常见浏览器仅在 Safari 、旧版 Edge 和部分魔改的 Chromium 内核浏览器上才支持播放。得益于 Chrome 在 v57 版本支持了 WebAssembly 并在 v68 版本重新开放了 SharedArrayBuffer（WebAssembly 多线程基础） 接口使得在 Web 上实现高性能应用成为了可能，于是我们自研了基于 WebAssembly 实现软解 HEVC 的播放器。我们在 Web 上进行 HEVC 优化就分为了两部分：在支持 HEVC 的浏览器上使用浏览器解码播放 HEVC 和在不支持 HEVC 的 Chromium 内核浏览器上使用基于 WebAssembly 开发的 WasmPlayer 解码播放 HEVC。

WasmPlayer 提供的是软件解码（软解）能力，通俗层面上软件解码是指使用 CPU 进行解码，相对应的硬件解码（硬解）则是使用 GPU 进行解码。软解的兼容性较高，在不支持硬解的设备上也能使用，劣势是相对而言占用更多的 CPU；硬解的兼容性则相对较低，需要硬件厂商提前适配，优势是功耗、CPU 占用低。硬解和软件相辅相成缺一不可，不是所有的设备都支持各类编码的硬解，新编码借助软解方案则可以实现更快的普及，让更多的用户提前使用上优秀的新编码。

**WebAssembly**

WebAssembly 是一个低级的编程语言，主要用于解决 JavaScript 运行性能低下的问题。WebAssembly 的出现并非是为了替代 JavaScript，而是一起协作，为 Web 应用提供更好的性能，并且可以将更多的桌面应用搬到 Web 上，丰富 Web 使用场景。

1995 年 Netscape 推出的 JavaScript 技术，极大丰富了 Web 的使用体验。为了实现一些更强大的功能以及更好的性能，陆续出现了 Java applet、ActiveX 等插件技术，以及多媒体领域的Flash技术。但是它们在可能会悄无声息的访问我们的文件系统，窃取我们的数据，所以安全性得不到保证。2008 年 Google 开发了 NaCl/PNaCl 技术，在独立的沙盒环境中执行，以保证系统和数据安全，但是 NaCl/PNaCl 方案无法直接与 JavaScript 通信，以及开发难度和成本等原因，最终导致 NaCl/PNaCl 无法普及。2010 年由于 NaCl/PNaCl 存在的一些问题，Mozilla 开发了 asm.js 方案，希望可以直接将 C/C++ 代码转换为 JavaScript，并解决 JavaScript 性能差的问题。2013 年 Google 和 Mozilla 开始合作，整合 NaCl/PNaCl 和 asm.js 方案，并于 2015 年由四个浏览器厂商开始联合开发 WebAssembly，最终于2019年正式发布 WebAssembly 标准。目前 WebAssembly 已经发展成为了“浏览器的第二语言”，相关技术也在逐步完善和发展。

从目前公开的资料以及实践我们得出：在视频编解码领域 WebAssembly 的速度是 x86 native 的 1/3~1/2。

**设计方案**

Web 端的视频播放方案一般都是直接使用 `<video>` 标签或使用 `<video>` 标签配合 MSE API（Media Source Extensions API，媒体扩展 API），`<video>` 标签和 MSE API 的设计简单实用，极大的降低了 Web 端多媒体播放的门槛，常见的 flv.js、dash.js、hls.js 都是使用的这两种方案来实现音视频播放。WasmPlayer 设计的出发点是如何融入现有的 Web 端播放器架构，以最小的成本在现有的 Web 端播放器上扩展支持 HEVC 解码和播放，因此实现一套自定义的 `<video>` 标签和 MSE API 接口成为了我们的首选方案。

我们通过 Shadow DOM API  封装了一个 `<bwp-video>` 标签，并实现了一套 BWP MSE API，在需要播放 HEVC 的场景下只需要在原本使用 `<video>` 标签及 MSE API 的位置替换成对应的 `<bwp-video>` 标签及 BWP MSE API 调用即可，可以灵活且基本无缝的在不同的方案间做切换，最低成本的集成到线上业务。作为现有 Web 播放器的一个扩展 SDK 带来的好处是不需要再重复造轮子开发已有的功能，例如：ABR、网络 IO、业务逻辑等，且不需要针对不同场景做额外的业务逻辑适配。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Ra5libNH8ibg5UNzb2j2bcHpPILCFdtg1ystH0Q1KrI3IoqdoqQvGHAkmoxM3YibgoxoBU8JvzoI7MQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

大部分的 WebAssembly 软解方案采用的都是 JavaScript 层主要实现 IO 控制、音视频渲染、音画同步，WebAssembly 层仅实现音视频解码。WasmPlayer 方案是在 JavaScript 层主要实现音视频渲染，WebAssembly 层使用 C/C++ 实现一个完整的播放器包括解封装、音视频解码、重采样、音画同步控制等，WebAssembly 层的内核使用成熟的播放器架构可以做到较稳定可控，解码及音画同步逻辑保持在同一层级以实现更好的同步性控制，得益于 C/C++ 的跨平台红利该播放器内核同时也可应用于 PC 端、移动端及 macOS。我们 WebAssembly 层实现的播放器为自研播放器，解码器使用内部的解码器，解封装与解析器也都为自研，所以我们的包体积可以做到比较小，SDK js 文件大小不到 200 KB， .wasm 文件 600 多 KB，SIMD 版本 .wasm 文件为 900 多 KB。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Ra5libNH8ibg5UNzb2j2bcHp307hBG2cCzbccwHPbOJrZuVXKIOltia0F35iboX8iaPuN77nkrMNXg63g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

**全异步渲染架构**

目前主流的 WebAssembly 播放器方案采用的都是主线程进行音视频渲染，子线程解码，但是主线程进行音视频渲染会导致诸多的问题。大多数的线上场景播放器页面都有不少的业务逻辑且绝大部分的逻辑都是运行在主线程，主线程的负载并不低，此时 WebAssembly 播放器再占用掉一部分的主线程资源就会导致主线程相对卡顿影响到用户交互流畅度。常规的主线程高负载场景有：页面资源载入阶段、高主线程 CPU 开销程序运行时，在该种场景下如果视频在播放中，我们会发现此时主线程经常性的阻塞会导致视频画面渲染卡顿明显、音频播放卡顿断断续续体验较差。

为了解决该问题我们采用的是全异步的音视频渲染，并将 WebAssembly 模块放置到子线程，所有的操作都是异步调用，对主线程的负担降到了最低，整体架构图如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754Ra5libNH8ibg5UNzb2j2bcHp94xXBR4HLL8kjYTrptOkwyCBPFyCUZNhc5qhp4XrKkbWccntmNPg2A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

整体架构可以分为 JavaScript 层和 WebAssembly 层，主线程我们主要实现了自定义的 `<bwp-video>` 标签接口（BwpElement，对应标准接口的 HTMLVideoElement）和 BWP MSE API 接口（BwpMediaSource，对应标准接口的 MediaSource），所有对 WebAssembly 层播放器的操作都会通过 JavaScript 层中处于子线程的 BwpCorePlayer 模块进行调用。按照线程可以划分为主线程的接口和调度层、负责播放控制和视频渲染的 JavaScript 子线程、音频渲染子线程、WebAssembly 内核子线程、视频解码子线程。

### 异步音频渲染

Web 端的音频渲染通常是使用 ScriptProcessorNode 接口或者使用 `<audio>` 标签实现，ScriptProcessorNode 运行在主线程会导致如上文提到的播放卡顿问题，使用 `<audio>` 标签则音视频的同步控制较难处理，比较容易出现音画不同步的情况。我们使用的是 Chrome 在 v64 版本开始支持的 AudioWorkletProcessor 来做音频渲染，WebAssembly 内核解码音频数据完成后会将 PCM 数据送入 AudioWorkletProcessor。由于 AudioWorkletProcessor 运行在专用的子线程所以只要控制一定长度的音频 buffer，不仅可以实现较为平滑的音频渲染还可以抵御一定时长的主线程阻塞导致音频播放卡顿，同时可以减少因主线程阻塞导致的音画不同步的情况。AudioWorkletProcessor 也带来了更细粒度的的音频 sample 数控制，可以做到更低的控制延迟。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

### 异步视频渲染

对于视频视频渲染我们采用的是 WebGL + OffscreenCanvas，OffscreenCanvas 是在 Chrome v69 版本推出的功能，提供了在 web worker 绘制 canvas 的能力，异步模式下在渲染逻辑会在子线程完成并将画面渲染到离屏画布，然后离屏画布会与主线程的画布进行交换在主线程显示离屏渲染的结果，实现异步渲染避免主线程阻塞。WasmPlayer 内核会将视频数据解码为 YUV 格式，并通过 WebGL 将 YUV 数据转换为 RGB 数据并渲染到 canvas，视频 YUV 数据的传输、渲染都直接在同一子线程中完成避免跨子线程的传输，在子线程中执行的渲染循环也不会受到主线程卡顿的影响，渲染帧数可提升 20%+（MacBook Pro (16-inch, 2019)），更为重要的是离屏渲染带来的高稳定性。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

OffscreenCanvas 带来的抗主线程抖动能力可以看如下的 Google OffscreenCanvas 文章示例，当点击 "MAKE ME BUSY" 后会触发主线程阻塞，此时 "Canvas on worker" 依旧能保持流畅的动画，而 “Canvas on main thread” 则会出现卡顿停住的情况：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**SIMD 性能优化**

Chrome v91 版本正式支持了 WebAssembly SIMD，WebAssembly 的性能得到了进一步的提升，WebAssembly 在支持多线程和 SIMD 后性能已经不能同日而语。我们使用 Emscripten 来将 C/C++ 内核代码转换为胶水代码及 .wasm 文件，Emscripten 支持自动将 SSE1, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2 及 128-bit AVX 汇编代码转换为 Wasm SIMD 且支持自动转换多线程代码，使得原先在 x64 和 Arm 平台上我们常用多线程、内存、缓存和汇编优化都可继续使用，这为我们节省了大量的工作。WasmPlayer 会在 Chromium 内核版本 >= v91 的浏览器上启用 SIMD 版本，低于 v91 的版本使用非 SIMD 版本，目前 96%+ 的用户使用的是 SIMD 版本。  

### 每秒解码帧数

软解的性能关乎着用户的实际体验，性能优化和对新技术的探索是我们持续的追求。WasmPlayer 可以在 MacBook Pro (16-inch, 2019) 标准版本上可以实现 4K 60 帧的 HEVC 视频实时解码播放，解码帧数最高可达 100 帧/s，比非 SIMD 版本快 2 倍左右。为了将负载降到最低 WasmPlayer 默认只会调用集成显卡进行视频渲染，只有在用户独立显卡被其他程序启用的情况下才会使用独立显卡进行渲染，YouTube 的全景视频则默认是启用独立显卡，会有较大的开销。

纯解码能力测试数据如下（测试机型：Intel i7-8700@3.20GHz（6C12T）16GB）：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

### CPU 负载

WasmPlayer 的 SIMD 版在中端轻薄本的实际播放场景下性能提高了（CPU 负载降低） 27%～70%，在中高端台式机上也有 31%～58% 的性能提高。以下数据测试条件为播放同一视频不同分辨率帧率相同时间取平均 CPU 负载。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

由上述数据可知 SIMD 版解码 1080P 30 FPS 的 CPU 负载在中端轻薄本（Table 1）为 5.83%，在中高端台式机（Table 2）为 3.20%，整体的负载都比较低，所以 SIMD 版在 1080P 分辨率下对于用户端 CPU 的消耗基本是相对可忽略的了。

**播放策略**

我站的视频编码格式主要分为 AV1、HEVC、AVC 三种，用户可根据喜好和网络条件进行配置，在网页端播放器的设置中可通过设置“播放策略”来调整三种编码的优先级，若没有播放策略选项则播放器只会使用 AVC 编码。目前内置播放策略中针对 HEVC、AV1 有清晰度或码率上限限制，并非所有清晰度均会使用，播放策略会一直优先保障用户的观看体验。针对以下特殊清晰度，目前仅能通过 HEVC 或 AV1 编码的稿件进行观看：HDR10 ( HEVC )、Dolby Vision ( HEVC )、8K (HEVC 、AV1 )

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

默认使用“默认”播放策略，该策略会根据码率、清晰度、异常检测来自动选择最优的编码格式进行播放。默认播放策略的大体处理逻辑如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### WasmPlayer 启用策略

WasmPlayer 并不会忽视用户的基本硬件配置无规则的启用，WasmPlayer 只会在 CPU 逻辑核心大于等于 4 且经过接口及配置验证的机器上启用，并根据分辨率划分不同的最低配置要求，例如：1080P 高码率视频最低的 CPU 逻辑核心数要求为 8、1080P 60 帧视频最低 CPU 逻辑核心数要求为 16。基本的配置检测和限制是保障播放体验的第一步；第二步我们支持了自动无缝降级，在视频播放卡顿、音画不同步（同步操作也无效的情况）、异常的场景下会进行自动的无缝降级操作，自动降级到 H.264/AVC 编码，保障可用性及用户体验；第三步是在用户短时间内遇到持续的卡顿情况下将会自动禁用 WasmPlayer 直到后续的新版本发布才会重新启用。

### 编码方式辨别

在播放器右键菜单中的“视频统计信息”可查看当前策略使用的视频编码，红框为视频编码格式

1.  AV1 编码
    
    ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
    
2.  浏览器原生 HEVC 硬解/软解
    
    ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
    
3.  WasmPlayer 解码播放 HEVC 编码，“Player Type” 字段为 “DashPlayer + WasmPlayer” 即为使用 WasmPlayer
    
    ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
    
4.  AVC 编码
    
    ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
    

**总结与展望**

随着 WebAssembly 的高速发展，相信 Web 上会出现越来越多的高性能跨平台移植的方案，例如：AutoCAD，Unity 游戏等，以往受限于单线程、JavaScript 性能层面的技术瓶颈也都将会有更多的解决方案，Wasmer 的出现也给 WebAssembly 在非 Web 平台带来了大放异彩的机会。期望随着 WebCodecs 的出现浏览器可以逐渐赋予开发者更多更底层操作多媒体的能力，例如：支持自定义的解码器、支持自定义协议解析器等。

___