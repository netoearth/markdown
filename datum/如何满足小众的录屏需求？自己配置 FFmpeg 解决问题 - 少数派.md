开始自我监控后，录屏工具的重要性急速提升，我遇到的问题也越来越多。

疫情封控两个月后，我对自己开始 24 ...

经过几天的自我监控后，我对于自己的活动有了更清晰的认知，方...

![](https://cdn.sspai.com/2022/05/22/9a850394d70ef92ba3c50d4274986908.jpg)

因为我录屏主要是为了自我监控，所以我需要的帧率并不用很高，甚至越低越好，分辨率同样不必和屏幕一致，能看清我在做什么即可。最初，我用了免费开源的 [VLC](https://www.videolan.org/vlc/)，这也是我文章里采用的方案。它能调节输出视频的编码、帧率、格式，但操作麻烦不说，还不能同时录屏和摄像头，暂停录制容易程序崩溃。

然后，我试了 [OBS](https://obsproject.com/)，它的录制功能极其强大，可以任意添加摄像头、文字、图像等，但输出限制多，生成的视频过大。同时，OBS 不支持录制画面与直播画面分开，而我平常习惯边开直播边工作，这令我只能放弃 OBS 录屏。

接着，我遇到 7.8k Star 的 [Capture](https://github.com/MathewSachin/Captura/releases/tag/v8.0.0)，它的自由度较高，能自定义叠加元素，然而项目已于 2018 年停止更新，使用时经常碰到莫名其妙的报错，很不稳定。

免费的不行，那收费的会不会好点呢？

我用 [Bandicam](https://www.bandicam.cn/) 录了一周的视频。与 Capture 相比，Bandicam 的稳定性好了很多，不会突然崩溃，还能降噪，内录扬声器音频，但它时不时丢失摄像头，导致无法自动进行录屏。

再后，我测试了其他几款录屏应用：

-   相机：Windows 自带应用，录制方便，但输出选项少，限制多。
-   [FlashBack Express](https://www.flashbackrecorder.com/zh/express/)：能调节帧率，镜像，虚化背景，但免费版只支持 2 小时内的录制。
-   [Mirillis Action!](https://mirillis.com/zh/products/action.html)：高帧率录制游戏，自动分割视频，自定义叠加元素，但输入帧率不能自由调整，最低只能 15 帧，试用期 30 天。
-   [oCam](https://ohsoft.net/eng/ocam/intro.php?cate=1002)：打着免费招牌但有弹窗广告，且输出视频偏大。
-   [ShareX](https://getsharex.com/)：免费开源强大的截图软件，具备录屏功能，能调节编码和帧率，但只能单一录屏或录像。

我一共试了 9 款录屏软件，体验都不算好，软件普遍存在无法自定义画面、不兼容、稳定性低的问题。再加上自我监控方案的单次录制时间在 12 小时以上，理想的帧率（0.02 帧）远超出应用最低 15-30 帧的下限。因此，我需要另找一款稳定能兼容自由度高，又能自由定制录屏方案的工具，最终找到的是这些录屏工具共同的父母：FFmpeg。

FFmpeg 是处理多媒体内容（如音频、视频、字幕和相关元数据）的库和工具的集合，支持在 Linux、macOS 和 Windows 全平台运行。它提供了录制、转换以及流化音视频的完整解决方案。

之前那些录屏、视频处理工具几乎都是基于 FFmpeg 而开发的。FFmpeg 能实现它们所有的功能，同时具备超高的稳定性和兼容性。现成的录屏应用与 FFmpeg 相比，优势只在于其美观的界面和简单易上手的录制方案。

如果你想跳出软件的限制，自由的定制录屏方案，避免莫名其妙的 bug，更加底层的 FFmpeg 反而是更稳定有效的方案。命令行录制看起来复杂，但实际上只需要熟悉十几个参数，你就能定制专属录屏方案，个人体验感觉比熟悉 Bandicam 的软件界面更简单。

下面以我的 Windows 桌面录制方案为例，从多屏幕中指定一个 2k 区域进行录制，并在画面右下角添加 360p 的摄像头录制角度，然后以帧率 0.02 输出监控视频。按 `q` 则停止录制。

![boxcnOTvcNypfpA6wac40fLKKKd](https://cdn.sspai.com/editor/u_/cdii815b34tcqo565olg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

输出画面如图例

## 录屏准备

### 配置 FFmpeg

1.  下载最新版 [FFmpeg](https://github.com/BtbN/FFmpeg-Builds/releases/tag/latest)，Windows 环境选择 `ffmpeg-master-latest-win64-gpl.zip`，GPL 版本包含了所有依赖项。
2.  将 FFmpeg 解压到任意文件夹，比如 `D:\Backup\Libraries\Documents\ffmpeg`。
3.  开始栏搜索「编辑系统环境变量」，点击进入「环境变量」。
4.  新建用户变量 `FFMPEG_HOME`，变量值设为刚才的解压路径 `D:\Backup\Libraries\Documents\ffmpeg`。

![boxcn9Y5dfJbBiiuWteMoOB2xEe](https://cdn.sspai.com/editor/u_/cdii81db34tcqo565om0?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

配置完成后，在终端输入 ffmpeg 即可启动。

![boxcnC51TlAjuqBGQ25WXQwChb3](https://cdn.sspai.com/editor/u_/cdii81lb34tcpvhjl0m0?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

### 配置视频/音频设备

FFmpeg 的录制命令 gdigrab 不支持音频录制，也不支持直接调用摄像头，此时需使用开源的 [screen-capture-recorder-to-video-windows-free](https://github.com/rdp/screen-capture-recorder-to-video-windows-free/releases) 增强 FFmpeg 的录制功能，其最新版本为 0.12.12。

通过命令 `ffmpeg -list_devices true -f dshow -i dummy` 查看支持的 Windows DirectShow 输入设备，采集视频和音频设备，包含设备名称，设备类型等信息<sup href="https://blog.csdn.net/m0_60352504/article/details/126762161" footnote-id="1" data-title="ffmpeg 录屏命令，参考 https://blog.csdn.net/m0_60352504/article/details/126762161">1</sup>

。比如我这里得到了视频设备「V380 FHD Camera」和音频设备「Analogue 1/2 (Audient iD4)」，之后会用到。

![boxcn6Roa6QiY7xhrKVdXdnxxjg](https://cdn.sspai.com/editor/u_/cdii81tb34tcqo565omg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

## 录制屏幕

从坐标 0:0 开始圈定出一个 2560x1440 的屏幕范围，然后以每 50 秒截图 1 帧，输出为 mp4 格式的视频，录制命令为： `ffmpeg -f gdigrab -r 20/1001 -draw_mouse 1 -offset_x 0 -offset_y 0 -video_size 2560x1440 -i desktop -s 1280x720 output.mp4`

以下是录制命令的说明：

-   `-f gdigrab` 使用 FFmpeg 内置屏幕录制命令 [gdigrab](https://ffmpeg.org/ffmpeg-all.html#gdigrab)，录制对象可为全屏、指定范围和指定程序。
-   `-r 20/1001` 帧率为 0.02，每 50 秒录制 1 帧。主流大家喜欢用 `-r 30` 录制，我这因为是每日监测视频，用了超低帧率。
-   `-c:v libx264` 是用于设置视频编解码器，一般可不填使用默认配置，`-c:a` 为音频编码。<sup href="" footnote-id="2" data-title="libx265 编码说明 https://ffmpeg.org/ffmpeg-codecs.html#libx265">2</sup>
-   `-draw_mouse 1` 在 gdigrab 录制的视频中显示鼠标。
-   `-offset_x 0 -offset_y 0 -video_size 2560x1440` 为起始坐标和选定录制范围。坐标可使用截图软件获取，比如我用 Snipaste，点击 F1 后进入截图界面，鼠标经过当前区域就会显示坐标。
-   `-s 1280x720` 用 scale 方法，设置视频分辨率为 720p。
-   `-i desktop` 为输入设备，指代显示屏。
-   `out.mp4` 为输出视频的名字与格式。默认保存在命令运行文件夹，可以在此处设置输出位置，如 `D:\Backup\Libraries\Desktop\out.mp4`。

除上方命令外，FFmpeg 还有许多参数可以设置，比如 `-pix_fmt yuv420p -preset ultrafast` 提升编码速度，`-filter:v "setpts=0.1*PTS"` 减少视频抽样，但 setpts 不是视频加速，对于低帧率的视频影响很小。<sup href="" footnote-id="3" data-title="x265 的 preset 与编码速度、视频画质以及比特率的关联 https://magiclen.org/x265-preset/
  FFmpeg 音视频倍速控制 https://blog.csdn.net/zhying719/article/details/123059209">3</sup>

## 录制摄像头

然后，我们使用上方获取的视频设备，即可用摄像头进行录制，如： `ffmpeg -f dshow -i video="V380 FHD Camera" output.mp4`

如果录屏的同时需要录制音频，则在命令中加入之前获取的音频设备，命令变为： `ffmpeg -f dshow -i audio="Analogue 1/2 (Audient iD4)" -f dshow -i video="V380 FHD Camera" output.mp4`

## 输出视频：画中画

清楚如何用 FFmpeg 录制屏幕、摄像头和音频后，我需要将他们放置于同一画面中，将摄像头画面放在录制画面的右下侧，并用 overlay 方法将其置于屏幕画面的上方，遮挡对应区域。<sup href="" footnote-id="4" data-title="FFmpeg 中 overlay 滤镜用法 - 水印及画中画 https://www.cnblogs.com/leisure_chn/p/10434209.html
  ffmpeg 调整缩放裁剪视频的基础知识 https://blog.csdn.net/guanyijun123/article/details/121270650">4</sup>

综合了以上三步，最终的录制命令为： `ffmpeg -f gdigrab -r 1 -draw_mouse 1 -offset_x 0 -offset_y 0 -video_size 2560x1440 -i desktop -s 1280x720 -b:v 0 -crf 32 output.mp4 -f dshow -i audio="Analogue 1/2 (Audient iD4)" -f dshow -s 640x360 -i video="V380 FHD Camera" -filter_complex "overlay=W-w-1:H-h-1" -y`

-   `-b:v 0 -crf 32` 是将视频比特率设置为最小，同时使用恒定质量，CRF<sup href="" footnote-id="5" data-title="固定码率因子">5</sup> 的范围可以从 0（最佳质量）到 63（最小文件大小）。
-   `overlay=W-w-1:H-h-1` 这是一个坐标，指浮层放在右下角，距离边缘 1px。
-   `-y` 遇到选项时，默认执行 yes 命令，比如覆盖同名的视频文件。

命令中的录制帧率较低，但不会影响同时录制的音频。之后的录屏只需在终端中运行这段命令，就会自动录制屏幕，按 `q` 即可停止录制。使用 FFmpeg 后，我的录屏再也没有莫名其妙的崩溃了。

## 常见问题

### Could not set video options

报错 `Could not set video options`，多是由于录制设置的帧率、分辨率超出设备范围造成的。使用命令 `ffmpeg -f dshow -list_options true -i video="V380 FHD Camera" -loglevel debug` 检查设备的输出属性，调整录制属性。

### real-time buffer

报错 `real-time buffer [xxxxxx] [video input] too full or near too full (181% of size: 3041280 [rtbufsize parameter])! frame dropped!`，解决方案参考 [issue 136](https://github.com/rdp/screen-capture-recorder-to-video-windows-free/issues/136)。我未修复成功，不过也未影响视频录制。

### 摄像头分辨率错误

如果摄像头画面出现裁切，分辨率与预想不同，则检查摄像头录制属性和摄像头应用输出分辨率。例如部分版本的 SplitCam Video Driver 对外场景尺寸被固定为 4:3，导致输出画面被裁剪，只能更换其他视频输入源。

### 录制画面偏移

录制画面比例异常或画幅偏移，这是 Windows 的屏幕缩放造成的，勾选 ffmpeg.exe 的属性「高 DPI 缩放替代」即可解决。

## 后续

如果读了 FFmpeg 的文档，就会发现这个工具异常强大，很多采用 FFmpeg 的工具都没有将它的功能性发挥到极致，用比较普适的功能尽可能地换取了软件操作的易用性。而相应地，如果和我一样，有一个比较小众，甚至特殊的需求，已经打包好的图形界面应用就很有可能力有不逮。这时，FFmpeg 这种底层的命令行工具可能就是唯一选择，而且用了就会发现，它在功能强大的同时还更加稳定，自定义能力也更强。而且如果跨过了起初对于命令行的恐惧，理解和上手其实也不算多难。

而且，FFmpeg 的功能不止录屏，它还有诸如连续截图、视频转帧率改大小等多种玩法，非常强大。

前几天，群里有人分享了快速生成 FFmpeg 命令的工具 [FFmpeg.guide](https://ffmpeg.guide/)。本以为能帮新手快速入门，使用后却感觉完全不实用。FFmpeg 最快入门的方法还是得看[官方文档](https://ffmpeg.org/ffmpeg.html)，也有一些爱好者整理翻译了相关的中文/视频教程。前期会耗费些时间，但只要定制好自己要的代码，之后能一直用，需求不变命令也就不会变。

当然，这篇文章的目的是分享我监控自己的延伸，分享使用 FFmpeg 录屏的入门方法，而非完全掌握，因此只介绍了录屏相关的核心代码。如果有需要，还是推荐研究一下官方文档，或者跟着我做的试一试，说不定就有新收获。

#### 关联阅读

-   [闭关家中，我将监控摄像对准了自己](https://sspai.com/post/73362)
-   [Zoom 会议摸鱼学](https://sspai.com/post/71404)
-   [一日一技 | 妙用 Keynote 实现数据可视化](https://sspai.com/post/76001)

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀