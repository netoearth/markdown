> **本文作者：**朱思达 飞书技术团队

### **背景简介**

什么是 HEVC ？简单说就是一种比 H264 压缩效率更高的现代视频编码格式，它支持 8K，支持 HDR，支持广色域，支持最高 16bit 的色彩深度，最高 YUV444 的色彩抽样，总之一句话，是一种用来取代现有 H264 的更高效、现代的视频编码格式，且目前已经被各类硬件广泛支持。

然而因为版权和技术派别等原因，这种格式一直没有被浏览器很好支持，尤其是目前市占率最高的 Chrome，一月初看到了一条 B 站用户吐槽 HEVC 解码性能/发热问题的新闻（感谢 B 站在 HEVC WASM 解码方案上的探索），考虑到这也是困扰业界很久的问题，大量依赖 HEVC 的 Web 项目均被迫产出了各种各样 Workaround 方案，但效果一直都不是最理想的，心想不如帮 Chromium 实现一下 HEVC 硬解吧。

本文简述了 Web 解码方案现状，介绍了作者为 Chromium 浏览器实现 & 完善硬解过程中遇到的问题和实现原理，并在文末附加了测试结果，预编译版本供参考，希望可以解决 FrontEnd 苦 HEVC 久矣的问题。

也可以提前下载 Chrome Canary（[https://www.google.com/chrome/canary/](https://link.zhihu.com/?target=https%3A//www.google.com/chrome/canary/)） ，体验 HEVC 硬解功能（ChromeOS、Android、Mac、Windows 需要添加启动参数 --enable-features=PlatformHEVCDecoderSupport，Linux 版本暂未支持）。

### **主流设备早已支持且广泛使用**

在 2015 年，苹果的 iPhone6s 就已经在其 A9 芯片内首次实现了 HEVC 硬解能力，同年，Intel 在第六代 Skylake 的 HD500 系列核显上，NVIDIA 在 GTX900 系列独显上，也先后支持了 HEVC 硬解。

在 2017 年发布的 iOS11, macOS 10.13 上，苹果继续完成了其 VideoToolbox 编解码框架对 HEVC 编解码能力的支持，微软也发布了 HEVC Video Extension 作为 Windows PC 环境 HEVC 解码的能力对标。

从此 HEVC 成为苹果，安卓默认视频格式，成为绝大多数单反 / 无人机 / 摄像设备的主推格式。

直到今年，也就是 2022 年，iPhone 已经出到了 13，芯片技术已经提升到了 5 纳米，然而我们所使用的大部分浏览器依然无法播放 HEVC 视频。

## **硬解的必要性**

### **更低的发热**

所谓硬解，即指使用 GPU 内专用于解码的芯片来处理解码工作，由于 GPU 多核心低频且专一的优势，在解码视频时发热和功耗显著低于 CPU。

### **更好的性能**

通过将 CPU 从繁重的解码工作中解放，可极大程度降低系统卡顿。

且 GPU 天生适合进行图形解码工作，解码性能秒杀 CPU，视频分辨率越高，显卡解码越可以做到不掉帧输出，因此“永远不要指望单纯靠 CPU 软解可以流畅播放 8K 60 帧的 HEVC 视频”。

### **总结**

HEVC 是目前桌面端或手机端播放器最主流的编码格式，考虑到其编码复杂度高，解码更耗费资源，因此为其实现硬解非常必要。

## **HEVC 解码的方案**

### **浏览器解码现状**

首先先来看看 Web 侧解码的现状：

### **Windows**

![](https://pic1.zhimg.com/v2-6e4f7932290c28012667905b03d33980_b.jpg)

### **macOS**

![](https://pic4.zhimg.com/v2-0caf2a7965d56683d25ff89216d638f7_b.jpg)

目前业内常用的 Web HEVC 解码方案大致可以分为两种：“换浏览器” 或 “WASM 软解”，他们各自有各自的优势和使用场景。

### **浏览器-Edge (硬解，仅 Windows）**

Chromium 内核的 Edge 在 Windows 系统下，额外支持了硬解 HEVC 视频，但必须满足如下条件：

1.  操作系统版本必须为 Windows 10 1709（16299.0）及以后版本。
2.  安装付费的 HEVC 视频扩展或免费的来自设备制造商的 HEVC 视频扩展且版本号必须大于等于 1.0.50361.0（由于一个存在了一年半以上的 Bug，老版本存在抖动的 Bug，Issue：[https://techcommunity.microsoft.com/t5/discussions/hevc-video-decoding-broken-with-b-frames/td-p/2077247/page/4](https://link.zhihu.com/?target=https%3A//techcommunity.microsoft.com/t5/discussions/hevc-video-decoding-broken-with-b-frames/td-p/2077247/page/4)）。

![](https://pic2.zhimg.com/v2-a6cb2582ba5cf77f23628e756ebbfe95_b.jpg)

3\. 版本号必须大于等于 Edge 99 。

![](https://pic1.zhimg.com/v2-5cdfa7f0338575d3b837090e2f1841b8_b.jpg)

在安装插件后，进入 edge://gpu 页面，可以查看 Edge 对于 HEVC 硬解支持的 Profile：

![](https://pic3.zhimg.com/v2-daba528819223ff5fb772d388605e372_b.jpg)

出现上图所示的字样，则证明硬解开启成功。

指标：

1.  分辨率最高支持 8192px \* 8192px。
2.  支持 HEVC Main / Main10 / Main Still Picture Profile。

优势：

1.  在显卡支持的情况，性能是最好的。
2.  HTMLVideoElement、MSE 等原生 API 的直接支持。

劣势：

1.  不支持 Windows 8 和老版本 Windows 10。
2.  需要手动装插件。
3.  HDR 支持不够好。

### **浏览器-Safari (硬解，仅 macOS）**

由于 Apple 是 HEVC 标准的主要推动者，因此早在 17 年的 Safari 11 即完成了 HEVC 视频硬解的支持，无需安装任何插件开箱即用。

指标：

1.  分辨率最高支持 8192px \* 8192px。
2.  支持 HEVC Main / Main10 Profile，M1+ 机型支持部分 HEVC Rext Profile。

优势：

1.  在显卡支持的情况，性能是最好的。
2.  HTMLVideoElement、MSE 等原生 API 的直接支持。
3.  开箱即用，无需装插件。
4.  HDR 支持最好（比如：杜比视界 Profile5，杜比全景声）。

劣势：

1.  生态不足，缺乏大量 Chromium 内核下“可用、好用的”插件。
2.  Safari 俗称“下一个 IE”，其浏览器 API 兼容性与实现，相比 Chromium 仍有差距。
3.  部分 HEVC 视频莫名其妙无法播放，哪怕视频本身没问题。

### **前端解码-WASM（软解，任何平台）**

此类方案绝大部分基于 WASM + FFMPEG 编译实现，支持所有支持 WASM 的浏览器。

指标：

1.  支持 FFMPEG 支持的所有分辨率和 Profile。

优势：

1.  不挑浏览器，是纯前端的技术实现。

劣势：

1.  需要依赖所在版本浏览器 WASM 的稳定性。
2.  不支持硬解，因为软解+性能损耗的缘故，性能有其天花板，4K 以上视频即使使用 5950X 这样的顶级 CPU 也会卡顿掉帧。
3.  非 HTML Video Element、MSE、EME 原生 API，需要手动用 js 初始化视频播放，使用有成本。

### **浏览器-本文方案（硬 / 软解，Windows / macOS / Linux）**

本文尝试直接为 Chromium 实现硬解，因为尽管 Safari 和 Edge 均已经实现了 HEVC 硬解，但它们均为闭源软件，无法被各种开源框架集成，而因为 Chromium 是开源的，这可以确保所有人可自行编译支持 Windows / macOS / Linux 硬解的 Chromium / Electron / CEF，考虑到实现原理部分较长，**因此如果你感兴趣，可直接下载预编译版本（[https://github.com/StaZhu/enable-chromium-hevc-hardware-decoding/releases](https://link.zhihu.com/?target=https%3A//github.com/StaZhu/enable-chromium-hevc-hardware-decoding/releases)）进行测试（未来会被包含在 Chrome 正式版本内，预编译版本可供大家尝鲜提前试用，也可下载 Chrome Canary），或跳到测评部分查看与 Edge / Safari 的对比。**

## **HEVC 硬解的实现原理**

正是因为如上瓶颈，“让专业的人做专业的事”这句话同样适用视频解码，GPU 硬解是很有必要的。GPU 解码的存在正是为了让解码工作可以充分利用显卡内部专用芯片，分担 CPU 解码时的压力，因此支持更多格式的硬解能力，已然成为众多显卡厂商的一大卖点。

首先我们需要做一些调研，研究下目前硬解框架是如何存在，并支持哪些“系统” or “GPU”。

下表来自 FFMPEG 项目对不同解码框架硬解支持情况的总结（来源：[https://trac.ffmpeg.org/wiki/HWAccelIntro](https://link.zhihu.com/?target=https%3A//trac.ffmpeg.org/wiki/HWAccelIntro)）

![](https://pic3.zhimg.com/v2-08ac1fd860cc249f86bdba47f39cf352_b.jpg)

硬解框架的支持情况，表格内容来自 FFmpeg 官网

可以看到硬解框架五花八门，不同的显卡厂商和设备有各自的专用解码框架，操作系统也有定义好的通用解码框架，由于显卡厂商众多，因此大部分播放器一般均基于通用框架实现硬解，少部分播放器在人力充裕的情况可能会为了更好的性能（显卡厂商自己的框架一般比通用框架性能更好，但也不绝对）额外对专用框架二次实现。

其中 Windows 平台通用的解码框架有 Media Foundation, Direct3D 11, DXVA2, 以及 OpenCL。macOS 平台通用的解码框架只有一个，也就是苹果自己的 VideoToolbox。Linux 平台的通用解码框架有 VAAPI 和 OpenCL。

显然，对于 Chrome 而言，为了更好的兼容性和稳定性，基于通用硬解框架实现硬解，更符合最小成本最大收益的目标，并提升了可维护性。

### **理解 Chromium 解码流程**

根据 Chromium Media 模块简介可知，浏览器将音视频播放一共抽象成三种类型，我们比较常见的有：Video Element 标签，MSE API。此外还有支持加密视频播放的 EME API，这三种在底层又存在多种复用关系。

![](https://pic1.zhimg.com/v2-db1f0f62c5bc1a8676f39d1b4092fbb8_b.jpg)

Chromium 的解码流程，图片来自 Chromium 代码仓库

那么到了最底层的解码模块，整体逻辑大概可以简述为：

1.  浏览器会从列表中依次按照顺序查找 Decoder，通常来说优先级最高的是硬解 Decoder, 然后会尝试软解 Decoder。
2.  如有命中其中的某个 Decoder 则执行后续解码逻辑。
3.  如没有命中的 Decoder，则解码失败，中止。

因此，为了实现 HEVC 硬解，我们首先需要找到各个平台的通用硬解 Decoder：

-   对于 Windows，根据操作系统以及显卡驱动版本，分为两种：D3D11VideoDecoder 和 VDAVideoDecoder，前者在大于 Windows8 且支持 D3D11 的系统默认被使用，后者则在前者不被使用时（比如 Windows 7）作为 Backup 方案被使用。
-   对于 macOS，为 VDAVideoDecoder。
-   对于 Linux，为 VAAPIVideoDecoder。

### **macOS 的硬解**

在了解了大致背景后，便可以开始探索实现 HEVC 硬解实现了，考虑到 Apple 其最新 Apple Silicon 芯片专门实现了**支持 H.264、HEVC 和 ProRes 的专用编解码媒体处理引擎，**看在 Apple 这么努力的份上，我首先挑选了 macOS 平台来进行尝试 。

### **FFMPEG 方案的尝试**

虽然 Chrome 没有直接实现 HEVC 解码能力，**但由于其实现了 FFMpegVideoDecoder，因此本质上任何 FFMPEG 可以播的视频，只要利用修改 Chromium 的方式为其添加 FFMPEG 解码器的入口，理论上均可以实现播放，**此方案其实是本文硬解实现前开源社区最广为流传的一种方案，@斯杰的文章（[https://www.infoq.cn/article/s65bFDPWzdfP9CQ6Wbw6](https://link.zhihu.com/?target=https%3A//www.infoq.cn/article/s65bFDPWzdfP9CQ6Wbw6)）内已有详尽介绍，由于当时的版本是基于 Chromium 79，目前最新的 Chromium 版本号为 104，因此里面的一些实现有所变动，但整体逻辑并没有明显改变，通过修改 Chromium 104 依然可以实现软解。

优点有很多：由于是 CPU 软解且使用行业最标准的 FFMPEG 解码，最终结果是：不挑系统，容错性好，支持任何 CPU 架构、操作系统，性能虽比不过硬解，但依然比前端 WASM 方案性能更好，且原生支持 MSE 和 Video Element。

缺点也很明显：普通的四核笔记本电脑，即使分辨率只有 1080P，在快进或快退时也会感到明显的卡顿，同时伴随比较高的 CPU 占用，抢占渲染进程 CPU 资源，另外这种方法是否有版权有待评估，但可以确定一点，使用平台提供的解码是合规且没有版权风险的。

当分辨率达到 4K 甚至 8K 级别，8 核甚至更多核的 CPU 也会卡到掉帧。

![](https://pic1.zhimg.com/v2-3de0da1b28450673b6e10a9bf3528448_b.jpg)

FFMPEG 的解码流程，图片来自知乎 @我是小北挖哈哈

根据 FFMPEG 的解码流程如上图（参考：[https://zhuanlan.zhihu.com/p/168240163?Futm\_source=wechat\_session&utm\_medium=social&utm\_oi=29396161265664](https://zhuanlan.zhihu.com/p/168240163?Futm_source=wechat_session&utm_medium=social&utm_oi=29396161265664)），可知道，FFMPEG 除了实现了软解，其实已经完整实现了硬解功能，然而 Chromium 的 FFMpegVideoDecoder 并不支持硬解，因此，我们的同学，首先尝试 FFMpegVideoDecoder 内尝试配置 `hw_device_ctx`，以开启其硬解能力，具体步骤如下：

开启硬解宏:

```
// third_party/ffmpeg/chromium/config/Chrome/mac/x64/config.h

#define CONFIG_VIDEOTOOLBOX 1
#define CONFIG_HEVC_VIDEOTOOLBOX_HWACCEL 1
#define HAVE_KCMVIDEOCODECTYPE_HEVC 1
```

设置硬件上下文:

```
// media/filters/ffmpeg_video_decoder.cc -> FFmpegVideoDecoder::ConfigureDecoder(const VideoDecoderConfig& config, bool low_delay)

if (decode_nalus_)
    codec_context_->flags2 |= AV_CODEC_FLAG2_CHUNKS;

+ if (codec_context_->codec_id == AVCodecID::AV_CODEC_ID_HEVC) {
+     AVBufferRef *hw_device_ctx = NULL;
+     int err;
+     if ((err = av_hwdevice_ctx_create(&hw_device_ctx, AV_HWDEVICE_TYPE_VIDEOTOOLBOX, NULL, NULL, 0)) >= 0) {
+         codec_context_->hw_device_ctx = av_buffer_ref(hw_device_ctx);
+     }
+ }

const AVCodec* codec = avcodec_find_decoder(codec_context_->codec_id);
```

取出解码数据:

```
// media/ffmpeg/ffmpeg_common.cc -> AVPixelFormatToVideoPixelFormat(AVPixelFormat pixel_format)

case AV_PIX_FMT_YUV420P:
case AV_PIX_FMT_YUVJ420P:
case AV_PIX_FMT_VIDEOTOOLBOX: // hwaccel
  return PIXEL_FORMAT_I420;
```

将硬件解码得到的数据取出，即 av\_hwframe\_transfer\_data 函数:

```
// media/ffmpeg/ffmpeg_decoding_loop.cc

FFmpegDecodingLoop::DecodeStatus FFmpegDecodingLoop::DecodePacket(const AVPacket* packet, FrameReadyCB frame_ready_cb) {
+    AVFrame* tmp_frame = NULL;
+    AVFrame* sw_frame = av_frame_alloc();
    bool sent_packet = false, frames_remaining = true, decoder_error = false;
    while (!sent_packet || frames_remaining) {
        ......
+        if (frame_.get()->format == AV_PIX_FMT_VIDEOTOOLBOX) {
+            int ret = av_hwframe_transfer_data(sw_frame, frame_.get(), 0);
+            tmp_frame = sw_frame;
+        } else {
+            tmp_frame = frame_.get();
+        }
+        const bool frame_processing_success = frame_ready_cb.Run(tmp_frame);
+        av_frame_unref(tmp_frame);
-        const bool frame_processing_success = frame_ready_cb.Run(frame_.get());
        av_frame_unref(frame_.get());
        if (!frame_processing_success)
            return DecodeStatus::kFrameProcessingFailed;
        }

    return decoder_error ? DecodeStatus::kDecodeFrameFailed : DecodeStatus::kOkay;
}
```

如上，经过多次尝试后，通过活动监视器可以观察到点击< Video >标签播放按钮时 VTDecoderXPCService 进程（Videotoolbox 的解码进程）CPU 占有率有所上升，说明调用 VideoToolbox 硬件解码模块成功，但视频白屏说明解码失败。

探索过程中，阅读 Chromium Media 模块的文档后发现，**使用 FFMpegVideoDecoder 不支持在 Sandboxed 的进程调用 VT 硬解框架，**为了避免在错误的道路上投入过多精力，遂放弃。

### **在 GPU 进程实现**

上面的方式行不通，说明得换一种思路，需要看看正统的 H264 硬解流程是怎样的，通过使用 Chrome 的搜索引擎（[https://source.chromium.org/](https://link.zhihu.com/?target=https%3A//source.chromium.org/)），发现 macOS 的 H264 硬解实现均位于`vt_video_decoder_accelerator.cc`这个文件内。

### **VideoToolbox 简介**

由 FFmpeg 介绍可知，如我们想在 macOS 实现 HEVC 硬解，则一定需要使用苹果提供的媒体解码框架 VideoToolbox 来完成。

> VideoToolbox is a low-level framework that provides direct access to hardware encoders and decoders. It provides services for video compression and decompression, and for conversion between raster image formats stored in CoreVideo pixel buffers. These services are provided in the form of session objects (compression, decompression, and pixel transfer), which are vended as Core Foundation (CF) types. Apps that don't need direct access to hardware encoders and decoders should not need to use VideoToolbox directly.

根据 Apple Developer 网站介绍（[https://developer.apple.com/documentation/videotoolbox](https://link.zhihu.com/?target=https%3A//developer.apple.com/documentation/videotoolbox)）可知，VideoToolbox 是苹果提供的直接用来进行编解码的底层框架，要实现硬解，大体解码流程可以理解为：Chromium -> VDAVideoDecoder -> VideoToolbox -> GPU -> VideoToolbox -> VDAVideoDecoder -> Chromium。

因此我们的目标就是正确按照 VideoToolbox 要求的方式，提交 Image Buffer，并等待 VT 将解码后的数据回传。

### **添加 Supported Profile**

根据 Chromium 解码流程 可知，Chromium 对于特定 Codec 的视频首先会尝试查找硬解 Decoder，如硬解 Decoder 不支持，则继续向后查找 Fallback 的软解 Decoder。

通过观察可发现，在 macOS 下，某种编码格式是否支持硬解，取决于硬解 Decoder 内的 SupportProfiles 是否包含这种编码格式，其代码如下：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

// 这个数组内包含了所有可能支持的Profile，但是否真正支持并不取决于这里
constexpr VideoCodecProfile kSupportedProfiles[] = {
    H264PROFILE_BASELINE, H264PROFILE_EXTENDED, H264PROFILE_MAIN,
    H264PROFILE_HIGH,

    // macOS 11以上，会尝试对这两种格式进行硬解
    VP9PROFILE_PROFILE0, VP9PROFILE_PROFILE2,

    // macOS 11以上，支持的最主流的HEVC Main / Main10 Profile, 以及
    // Main Still Picture / Main Rext 的硬、软解
    // (Apple Silicon 机型支持硬解HEVC Rext, Intel 机型支持软解HEVC Rext)
    // These are only supported on macOS 11+.
    HEVCPROFILE_MAIN, HEVCPROFILE_MAIN10, HEVCPROFILE_MAIN_STILL_PICTURE,
    HEVCPROFILE_REXT,

    // TODO(sandersd): Hi10p fails during
    // CMVideoFormatDescriptionCreateFromH264ParameterSets with
    // kCMFormatDescriptionError_InvalidParameter.
    //
    // H264PROFILE_HIGH10PROFILE,

    // TODO(sandersd): Find and test media with these profiles before enabling.
    //
    // H264PROFILE_SCALABLEBASELINE,
    // H264PROFILE_SCALABLEHIGH,
    // H264PROFILE_STEREOHIGH,
    // H264PROFILE_MULTIVIEWHIGH,
};
```

### **Session 预热与引导逻辑**

实现硬解，需要在 Sandboxed 的进程启用前创建解码 Session 预热，并根据系统版本与支持情况决定最终是否启用硬解：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc
bool InitializeVideoToolbox() {
  // 在GPU主进程调用时立刻执行，以确保Sandboxed/非Sandoxed进程均可硬解
  static const bool succeeded = InitializeVideoToolboxInternal();
  return succeeded;
}

// 在GPU Sandbox启用前通过创建Videotoolbox的Decompression Session预热，确保Sandboxed/非Sandoxed进程均可硬解
bool InitializeVideoToolboxInternal() {
  VTDecompressionOutputCallbackRecord callback = {0};
  base::ScopedCFTypeRef<VTDecompressionSessionRef> session;
  gfx::Size configured_size;

  // 创建H264硬解Session
  const std::vector<uint8_t> sps_h264_normal = {
      0x67, 0x64, 0x00, 0x1e, 0xac, 0xd9, 0x80, 0xd4, 0x3d, 0xa1, 0x00, 0x00,
      0x03, 0x00, 0x01, 0x00, 0x00, 0x03, 0x00, 0x30, 0x8f, 0x16, 0x2d, 0x9a};
  const std::vector<uint8_t> pps_h264_normal = {0x68, 0xe9, 0x7b, 0xcb};
  if (!CreateVideoToolboxSession(
          CreateVideoFormatH264(sps_h264_normal, std::vector<uint8_t>(),
                                pps_h264_normal),
          /*require_hardware=*/true, /*is_hbd=*/false, &callback, &session,
          &configured_size)) {
    // 如果H264硬解Session创建失败，直接禁用整个硬解模块
    DVLOG(1) << "Hardware H264 decoding with VideoToolbox is not supported";
    return false;
  }

  session.reset();

  // 创建H264软解Session
  // 总结下，如果这台设备连H264硬/软解都不支持，则直接禁用硬解，解码完全走FFMpegVideoDecoder的软解
  const std::vector<uint8_t> sps_h264_small = {
      0x67, 0x64, 0x00, 0x0a, 0xac, 0xd9, 0x89, 0x7e, 0x22, 0x10, 0x00,
      0x00, 0x3e, 0x90, 0x00, 0x0e, 0xa6, 0x08, 0xf1, 0x22, 0x59, 0xa0};
  const std::vector<uint8_t> pps_h264_small = {0x68, 0xe9, 0x79, 0x72, 0xc0};
  if (!CreateVideoToolboxSession(
          CreateVideoFormatH264(sps_h264_small, std::vector<uint8_t>(),
                                pps_h264_small),
          /*require_hardware=*/false, /*is_hbd=*/false, &callback, &session,
          &configured_size)) {
    DVLOG(1) << "Software H264 decoding with VideoToolbox is not supported";
    // 如果H264软解 Decompression Session创建失败，直接禁用整个硬解模块
    return false;
  }

  session.reset();

  if (__builtin_available(macOS 11.0, *)) {
    VTRegisterSupplementalVideoDecoderIfAvailable(kCMVideoCodecType_VP9);

    // 当系统大于等于macOS Big Sur时，尝试创建VP9硬解Session
    if (!CreateVideoToolboxSession(
            CreateVideoFormatVP9(VideoColorSpace::REC709(), VP9PROFILE_PROFILE0,
                                 absl::nullopt, gfx::Size(720, 480)),
            /*require_hardware=*/true, /*is_hbd=*/false, &callback, &session,
            &configured_size)) {
      DVLOG(1) << "Hardware VP9 decoding with VideoToolbox is not supported";
      // 如果创建session失败，说明不支持VP9硬解，跳过，但保持H264可继续硬解
    }
  }

// 按照Chromium的要求HEVC硬解相关的逻辑，均需要依赖ENABLE_HEVC_PARSER_AND_HW_DECODER宏定义开关，只有开启开关后才会将代码引入
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  // 即使编译时开启了HEVC硬解宏
  // 当启动时传入`--enable-features=PlatformHEVCDecoderSupport`可启用HEVC硬解
  if (base::FeatureList::IsEnabled(media::kPlatformHEVCDecoderSupport)) {
    // 这里限制了至少是Big Sur系统的原因是，Catalina及以下系统使用
    // CMVideoFormatDescriptionCreateFromHEVCParameterSets API创建解码Session
    // 会失败
    // 注：macOS自身问题，苹果承诺了10.13及以上系统即可使用这个API，然，实测结果并卵
    // 但VLC和FFmpeg等使用的CMVideoFormatDescriptionCreate可以正常创建
    // 但，这与硬解模块实现的风格和结构不符
    if (__builtin_available(macOS 11.0, *)) {
      session.reset();

      // 创建HEVC硬解Session
      // vps/sps/pps提取自bear-1280x720-hevc.mp4
      const std::vector<uint8_t> vps_hevc_normal = {
          0x40, 0x01, 0x0c, 0x01, 0xff, 0xff, 0x01, 0x60,
          0x00, 0x00, 0x03, 0x00, 0x90, 0x00, 0x00, 0x03,
          0x00, 0x00, 0x03, 0x00, 0x5d, 0x95, 0x98, 0x09};

      const std::vector<uint8_t> sps_hevc_normal = {
          0x42, 0x01, 0x01, 0x01, 0x60, 0x00, 0x00, 0x03, 0x00, 0x90, 0x00,
          0x00, 0x03, 0x00, 0x00, 0x03, 0x00, 0x5d, 0xa0, 0x02, 0x80, 0x80,
          0x2d, 0x16, 0x59, 0x59, 0xa4, 0x93, 0x2b, 0xc0, 0x5a, 0x70, 0x80,
          0x00, 0x01, 0xf4, 0x80, 0x00, 0x3a, 0x98, 0x04};

      const std::vector<uint8_t> pps_hevc_normal = {0x44, 0x01, 0xc1, 0x72,
                                                    0xb4, 0x62, 0x40};

      if (!CreateVideoToolboxSession(
              CreateVideoFormatHEVC(vps_hevc_normal, sps_hevc_normal,
                                    pps_hevc_normal),
              /*require_hardware=*/true, /*is_hbd=*/false, &callback, &session,
              &configured_size)) {
        DVLOG(1) << "Hardware HEVC decoding with VideoToolbox is not supported";
        // 同VP9逻辑，HEVC硬解预热失败不会禁用H264硬解能力
      }

      session.reset();

      // 创建HEVC软解Session
      // vps/sps/pps提取自bear-320x240-v_frag-hevc.mp4
      const std::vector<uint8_t> vps_hevc_small = {
          0x40, 0x01, 0x0c, 0x01, 0xff, 0xff, 0x01, 0x60,
          0x00, 0x00, 0x03, 0x00, 0x90, 0x00, 0x00, 0x03,
          0x00, 0x00, 0x03, 0x00, 0x3c, 0x95, 0x98, 0x09};

      const std::vector<uint8_t> sps_hevc_small = {
          0x42, 0x01, 0x01, 0x01, 0x60, 0x00, 0x00, 0x03, 0x00, 0x90,
          0x00, 0x00, 0x03, 0x00, 0x00, 0x03, 0x00, 0x3c, 0xa0, 0x0a,
          0x08, 0x0f, 0x16, 0x59, 0x59, 0xa4, 0x93, 0x2b, 0xc0, 0x40,
          0x40, 0x00, 0x00, 0xfa, 0x40, 0x00, 0x1d, 0x4c, 0x02};

      const std::vector<uint8_t> pps_hevc_small = {0x44, 0x01, 0xc1, 0x72,
                                                   0xb4, 0x62, 0x40};

      if (!CreateVideoToolboxSession(
              CreateVideoFormatHEVC(vps_hevc_small, sps_hevc_small,
                                    pps_hevc_small),
              /*require_hardware=*/false, /*is_hbd=*/false, &callback, &session,
              &configured_size)) {
        DVLOG(1) << "Software HEVC decoding with VideoToolbox is not supported";

        //  同VP9逻辑，HEVC软解预热失败不会禁用H264硬解能力
      }
    }
  }
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  return true;
}

// 实际的最终判断逻辑
VideoDecodeAccelerator::SupportedProfiles
VTVideoDecodeAccelerator::GetSupportedProfiles(
    const gpu::GpuDriverBugWorkarounds& workarounds) {
  SupportedProfiles profiles;
  // H264硬/软解不支持时，禁用硬解模块
  if (!InitializeVideoToolbox())
    return profiles;

  for (const auto& supported_profile : kSupportedProfiles) {
    // 目前仅支持VP9 PROFILE0、2两种Profile
    if (supported_profile == VP9PROFILE_PROFILE0 ||
        supported_profile == VP9PROFILE_PROFILE2) {
      // 所有GPU模块的解码都会先读取依赖GPU Workaround
      // 比如需要禁用特定型号或厂商的GPU对特定Codec的硬解支持
      // 则可利用GPU Workaround下发禁用配置
      if (workarounds.disable_accelerated_vp9_decode)
        continue;
      if (!base::mac::IsAtLeastOS11())
        // 系统版本不支持VP9硬解，跳过
        continue;
      if (__builtin_available(macOS 10.13, *)) {
        if ((supported_profile == VP9PROFILE_PROFILE0 ||
             supported_profile == VP9PROFILE_PROFILE2) &&
            !VTIsHardwareDecodeSupported(kCMVideoCodecType_VP9)) {
          // Profile不支持，或操作系统不支持VP9硬解，跳过
          continue;
        }

        // 经过GPU workaround、操作系统版本、Profile、以及OS是否支持VP9硬解检查，最终确认支持VP9硬解，并接管解码权限
      } else {
        // 系统版本不支持VP9硬解，跳过
        continue;
      }
    }

    // 目前支持HEVC Main、Main10、MSP、Rext四种Profile
    if (supported_profile == HEVCPROFILE_MAIN ||
        supported_profile == HEVCPROFILE_MAIN10 ||
        supported_profile == HEVCPROFILE_MAIN_STILL_PICTURE ||
        supported_profile == HEVCPROFILE_REXT) {
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
      if (!workarounds.disable_accelerated_hevc_decode &&
          base::FeatureList::IsEnabled(kPlatformHEVCDecoderSupport)) {
        if (__builtin_available(macOS 11.0, *)) {
          // 经过GPU workaround、操作系统版本、Profile，编译开关，启动开关检查，最终确认支持HEVC硬解（软解我们也使用Videotoolbox来做，原因后面说），并接管解码权限
          SupportedProfile profile;
          profile.profile = supported_profile;
          profile.min_resolution.SetSize(16, 16);
          // HEVC最大可支持8k  
          profile.max_resolution.SetSize(8192, 8192);
          profiles.push_back(profile);
        }
      }<br/>#endif  //  BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
      continue;
    }

    // H264和VP9最大支持4k
    SupportedProfile profile;
    profile.profile = supported_profile;
    profile.min_resolution.SetSize(16, 16);
    profile.max_resolution.SetSize(4096, 4096);
    profiles.push_back(profile);
  }
  return profiles;
}
```

如上，经过 GPU workaround、操作系统版本、Profile、编译开关、启动开关检查，最终如果校验通过，则 HEVC 解码逻辑会由 VideoToolbox 接管，并由 VTDecoderXPCService 进程最终实际负责解码。

### **理解 HEVC 的 NALU 类型**

NALU (network abstraction layer unit)，即网络抽象层单元，是 H.264 / AVC 和 HEVC 视频编码标准的核心定义，按白话理解，就是 H264 / HEVC 为不同的视频单元定义了的不同的类型（参考），感兴趣可自行百科，这里不再赘述。对于 H264，存在 32 种，其中保留 Nalu 有 8 种。到了 HEVC，被扩展到了 64 种，保留 Nalu 有 16 种。

![](https://pic2.zhimg.com/v2-e1361a0a621e13e04ccf92b9ab9b6d4d_b.jpg)

H264 的 Nalu Unit 组成，图片来自 Apple

```
// media/video/h265_nalu_parser.h

enum Type {
    TRAIL_N = 0,  // coded slice segment of a non TSA(Temporal Sub-layer Access)
                  // trailing picture
    TRAIL_R = 1,  // coded slice segment of a non TSA(Temporal Sub-layer Access)
                  // trailing picture
    TSA_N = 2,    // coded slice segment of a TSA(Temporal Sub-layer Access)
                  // trailing picture
    TSA_R = 3,    // coded slice segment of a TSA(Temporal Sub-layer Access)
                  // trailing picture
    STSA_N = 4,   // coded slice segment of a STSA(Step-wise Temporal Sub-layer
                  // Access) trailing picture
    STSA_R = 5,   // coded slice segment of a STSA(Step-wise Temporal Sub-layer
                  // Access) trailing picture
    RADL_N = 6,   // coded slice segment of a RADL(Random Access Decodable
                  // Leading) leading picture
    RADL_R = 7,   // coded slice segment of a RADL(Random Access Decodable
                  // Leading) leading picture
    RASL_N = 8,   // coded slice segment of a RASL(Random Access Skipped
                  // Leading)L leading picture
    RASL_R = 9,  // coded slice segment of a RASL(Random Access Skipped Leading)
                 // leading picture
    RSV_VCL_N10 = 10,     // reserved non-IRAP SLNR VCL
    RSV_VCL_R11 = 11,     // reserved non-IRAP sub-layer reference VCL
    RSV_VCL_N12 = 12,     // reserved non-IRAP SLNR VCL
    RSV_VCL_R13 = 13,     // reserved non-IRAP sub-layer reference VCL
    RSV_VCL_N14 = 14,     // reserved non-IRAP SLNR VCL
    RSV_VCL_R15 = 15,     // reserved non-IRAP sub-layer reference VCL
    BLA_W_LP = 16,        // coded slice segment of a BLA IRAP picture
    BLA_W_RADL = 17,      // coded slice segment of a BLA IRAP picture
    BLA_N_LP = 18,        // coded slice segment of a BLA IRAP picture
    IDR_W_RADL = 19,      // coded slice segment of an IDR IRAP picture
    IDR_N_LP = 20,        // coded slice segment of an IDR IRAP picture
    CRA_NUT = 21,         // coded slice segment of a CRA IRAP picture
    RSV_IRAP_VCL22 = 22,  // reserved IRAP(intra random access point) VCL
    RSV_IRAP_VCL23 = 23,  // reserved IRAP(intra random access point) VCL
    RSV_VCL24 = 24,       // reserved non-IRAP VCL
    RSV_VCL25 = 25,       // reserved non-IRAP VCL
    RSV_VCL26 = 26,       // reserved non-IRAP VCL
    RSV_VCL27 = 27,       // reserved non-IRAP VCL
    RSV_VCL28 = 28,       // reserved non-IRAP VCL
    RSV_VCL29 = 29,       // reserved non-IRAP VCL
    RSV_VCL30 = 30,       // reserved non-IRAP VCL
    RSV_VCL31 = 31,       // reserved non-IRAP VCL
    VPS_NUT = 32,         // vps(video parameter sets)
    SPS_NUT = 33,         // sps(sequence parameter sets)
    PPS_NUT = 34,         // pps(picture parameter sets)
    AUD_NUT = 35,         // access unit delimiter
    EOS_NUT = 36,         // end of sequence
    EOB_NUT = 37,         // end of bitstream
    FD_NUT = 38,          // filter Data
    PREFIX_SEI_NUT = 39,  // sei
    SUFFIX_SEI_NUT = 40,  // sei
    RSV_NVCL41 = 41,      // reserve
    RSV_NVCL42 = 42,      // reserve
    RSV_NVCL43 = 43,      // reserve
    RSV_NVCL44 = 44,      // reserve
    RSV_NVCL45 = 45,      // reserve
    RSV_NVCL46 = 46,      // reserve
    RSV_NVCL47 = 47,      // reserve
    UNSPEC48 = 48,        // unspecified
    UNSPEC49 = 49,        // unspecified
    UNSPEC50 = 50,        // unspecified
    UNSPEC51 = 51,        // unspecified
    UNSPEC52 = 52,        // unspecified
    UNSPEC53 = 53,        // unspecified
    UNSPEC54 = 54,        // unspecified
    UNSPEC55 = 55,        // unspecified
    UNSPEC56 = 56,        // unspecified
    UNSPEC57 = 57,        // unspecified
    UNSPEC58 = 58,        // unspecified
    UNSPEC59 = 59,        // unspecified
    UNSPEC60 = 60,        // unspecified
    UNSPEC61 = 61,        // unspecified
    UNSPEC62 = 62,        // unspecified
    UNSPEC63 = 63,        // unspecified
  };
```

### **解析 SPS / PPS / VPS**

如想实现 HEVC 解码，首先需要拿到视频的元数据，这就需要通过解析 NALU 类型为 32 (`VPS_NUT`), 33 (`SPS_NUT`), 34 (`PPS_NUT`)的 Nalu Header 来获取。

举个最基本的例子，如果我们希望获取视频的宽高，则需要解析`SPS_NUT`的 Nalu Header，并取`sps->pic_width_in_luma_samples`的值，以此类推。

通常媒体开发会使用一个叫做`StreamAnalyzer`的工具（链接：[https://www.elecard.com/zh/products/video-analysis/stream-analyzer](https://link.zhihu.com/?target=https%3A//www.elecard.com/zh/products/video-analysis/stream-analyzer)）快速解析视频 Nalu Header，我们要做的事其实和这个软件做的差不多：

![](https://pic1.zhimg.com/v2-de986e8daf5af6279131ee1acfca56ac_b.jpg)

Stream Analyzer 解析 Nalu Header 示意

可以看到，`SPS_NUT`的 Nalu Header 解析后的数据如截图右侧区域显示，感谢 Elecard 开发的这款好用的工具，有了它对我们实现 VPS 解析有很大帮助。

观察 Chromium 的代码结构发现 @Jeffery Kardatzke 大佬已经于 2020 年底完成 Linux 平台 Vappi HEVC 的硬解加速实现，和 H265 Nalu Parse 的大部分逻辑实现，由于 Linux 平台硬解并不需要提取 VPS 参数，因为大佬没有实现 VPS 解析，但根据 Apple Developer 的说明，若我们使用`CMVideoFormatDescriptionCreateFromHEVCParameterSets` API 创建解码 session，需要提供 VPS, SPS, PPS 三种类型的 Nalu Data，因此实现 macOS 硬解的很大一部分工作即是完成 VPS NALU 的 Header 解析：

首先，参考 `T-REC-H.265-202108-I`，以及 FFMPEG 定义好的 `H265RawVPS Struct Reference`，我们需要定义好要解析的 VPS 结构体类型：

```
// media/video/h265_parser.h

// 定义H265VPS的结构体
struct MEDIA_EXPORT H265VPS {
  H265VPS();

  int vps_video_parameter_set_id; // 即vps_id，稍后需要用到
  bool vps_base_layer_internal_flag;
  bool vps_base_layer_available_flag;
  int vps_max_layers_minus1;
  int vps_max_sub_layers_minus1;
  bool vps_temporal_id_nesting_flag;
  H265ProfileTierLevel profile_tier_level;
  int vps_max_dec_pic_buffering_minus1[kMaxSubLayers]; // 稍后需要用到
  int vps_max_num_reorder_pics[kMaxSubLayers]; // 稍后需要用到
  int vps_max_latency_increase_plus1[kMaxSubLayers];
  int vps_max_layer_id;
  int vps_num_layer_sets_minus1;
  bool vps_timing_info_present_flag;

  // 剩余部分我们不需要，因此暂未实现解析逻辑
};
```

接着，我们需要完成 `H265VPS` 的解析逻辑：

```
// media/video/h265_parser.cc

// 解析VPS逻辑
H265Parser::Result H265Parser::ParseVPS(int* vps_id) {
  DVLOG(4) << "Parsing VPS";
  Result res = kOk;

  DCHECK(vps_id);
  *vps_id = -1;

  std::unique_ptr<H265VPS> vps = std::make_unique<H265VPS>();

  // 读4Bit
  READ_BITS_OR_RETURN(4, &vps->vps_video_parameter_set_id);
  // 校验读取结果是否为0-16区间内的值
  IN_RANGE_OR_RETURN(vps->vps_video_parameter_set_id, 0, 16);
  READ_BOOL_OR_RETURN(&vps->vps_base_layer_internal_flag);
  READ_BOOL_OR_RETURN(&vps->vps_base_layer_available_flag);
  READ_BITS_OR_RETURN(6, &vps->vps_max_layers_minus1);
  IN_RANGE_OR_RETURN(vps->vps_max_layers_minus1, 0, 62);
  READ_BITS_OR_RETURN(3, &vps->vps_max_sub_layers_minus1);
  IN_RANGE_OR_RETURN(vps->vps_max_sub_layers_minus1, 0, 7);
  READ_BOOL_OR_RETURN(&vps->vps_temporal_id_nesting_flag);
  SKIP_BITS_OR_RETURN(16);  // 跳过vps_reserved_0xffff_16bits
  res = ParseProfileTierLevel(true, vps->vps_max_sub_layers_minus1,
                              &vps->profile_tier_level);
  if (res != kOk) {
    return res;
  }

  bool vps_sub_layer_ordering_info_present_flag;
  READ_BOOL_OR_RETURN(&vps_sub_layer_ordering_info_present_flag);

  for (int i = vps_sub_layer_ordering_info_present_flag
                   ? 0
                   : vps->vps_max_sub_layers_minus1;
       i <= vps->vps_max_sub_layers_minus1; ++i) {
    READ_UE_OR_RETURN(&vps->vps_max_dec_pic_buffering_minus1[i]);
    IN_RANGE_OR_RETURN(vps->vps_max_dec_pic_buffering_minus1[i], 0, 15);
    READ_UE_OR_RETURN(&vps->vps_max_num_reorder_pics[i]);
    IN_RANGE_OR_RETURN(vps->vps_max_num_reorder_pics[i], 0,
                       vps->vps_max_dec_pic_buffering_minus1[i]);
    if (i > 0) {
      TRUE_OR_RETURN(vps->vps_max_dec_pic_buffering_minus1[i] >=
                     vps->vps_max_dec_pic_buffering_minus1[i - 1]);
      TRUE_OR_RETURN(vps->vps_max_num_reorder_pics[i] >=
                     vps->vps_max_num_reorder_pics[i - 1]);
    }
    READ_UE_OR_RETURN(&vps->vps_max_latency_increase_plus1[i]);
  }
  if (!vps_sub_layer_ordering_info_present_flag) {
    for (int i = 0; i < vps->vps_max_sub_layers_minus1; ++i) {
      vps->vps_max_dec_pic_buffering_minus1[i] =
          vps->vps_max_dec_pic_buffering_minus1[vps->vps_max_sub_layers_minus1];
      vps->vps_max_num_reorder_pics[i] =
          vps->vps_max_num_reorder_pics[vps->vps_max_sub_layers_minus1];
      vps->vps_max_latency_increase_plus1[i] =
          vps->vps_max_latency_increase_plus1[vps->vps_max_sub_layers_minus1];
    }
  }

  READ_BITS_OR_RETURN(6, &vps->vps_max_layer_id);
  IN_RANGE_OR_RETURN(vps->vps_max_layer_id, 0, 62);
  READ_UE_OR_RETURN(&vps->vps_num_layer_sets_minus1);
  IN_RANGE_OR_RETURN(vps->vps_num_layer_sets_minus1, 0, 1023);

  *vps_id = vps->vps_video_parameter_set_id;
  // 如果存在相同vps_id的vps，则直接替换
  active_vps_[*vps_id] = std::move(vps);

  return res;
}

// 获取VPS逻辑
const H265VPS* H265Parser::GetVPS(int vps_id) const {
  auto it = active_vps_.find(vps_id);
  if (it == active_vps_.end()) {
    DVLOG(1) << "Requested a nonexistent VPS id " << vps_id;
    return nullptr;
  }

  return it->second.get();
}
```

完善编写 Unit Test 和 Fuzzer Test：

```
// media/video/h265_parser_unittest.cc

TEST_F(H265ParserTest, VpsParsing) {
  LoadParserFile("bear.hevc");
  H265NALU target_nalu;
  EXPECT_TRUE(ParseNalusUntilNut(&target_nalu, H265NALU::VPS_NUT));
  int vps_id;
  EXPECT_EQ(H265Parser::kOk, parser_.ParseVPS(&vps_id));
  const H265VPS* vps = parser_.GetVPS(vps_id);
  EXPECT_TRUE(!!vps);
  EXPECT_TRUE(vps->vps_base_layer_internal_flag);
  EXPECT_TRUE(vps->vps_base_layer_available_flag);
  EXPECT_EQ(vps->vps_max_layers_minus1, 0);
  EXPECT_EQ(vps->vps_max_sub_layers_minus1, 0);
  EXPECT_TRUE(vps->vps_temporal_id_nesting_flag);
  EXPECT_EQ(vps->profile_tier_level.general_profile_idc, 1);
  EXPECT_EQ(vps->profile_tier_level.general_level_idc, 60);
  EXPECT_EQ(vps->vps_max_dec_pic_buffering_minus1[0], 4);
  EXPECT_EQ(vps->vps_max_num_reorder_pics[0], 2);
  EXPECT_EQ(vps->vps_max_latency_increase_plus1[0], 0);
  for (int i = 1; i < kMaxSubLayers; ++i) {
    EXPECT_EQ(vps->vps_max_dec_pic_buffering_minus1[i], 0);
    EXPECT_EQ(vps->vps_max_num_reorder_pics[i], 0);
    EXPECT_EQ(vps->vps_max_latency_increase_plus1[i], 0);
  }
  EXPECT_EQ(vps->vps_max_layer_id, 0);
  EXPECT_EQ(vps->vps_num_layer_sets_minus1, 0);
  EXPECT_FALSE(vps->vps_timing_info_present_flag);
}

// media/video/h265_parser_fuzzertest.cc

case media::H265NALU::VPS_NUT:
  int vps_id;
  res = parser.ParseVPS(&vps_id);
  break;
```

由于 FFMPEG 已经实现了 VPS 的解析逻辑，因此这里大部分逻辑与 FFMPEG 保持一致即可，经过 UnitTest 测试（编译步骤：`autoninja -C out/Release64 media_unittests`) 确认无问题，对照 StreamAnalyzer 同样无问题后，完成 VPS 解析逻辑实现。

这里跳过 SPS, PPS, SliceHeader 的解析逻辑，因为代码量过大且琐碎，感兴趣可参考 [http://h265\_parser.cc](https://link.zhihu.com/?target=http%3A//h265_parser.cc)（[https://source.chromium.org/chromium/chromium/src/+/main:media/video/h265\_parser.cc](https://link.zhihu.com/?target=https%3A//source.chromium.org/chromium/chromium/src/%2B/main%3Amedia/video/h265_parser.cc)）

### **计算 POC (Picture Order Count)**

我们知道 H264 / HEVC 视频帧类型大体上有三种：I 帧，P 帧，B 帧，其中 I 帧又称全帧压缩编码帧，为整个 GOP（一个存在了 I，P，B 的帧组）内的第一帧，解码无需参考其他帧，P 帧又称前向预测编码帧，解码需要参考前面的 I，P 帧解码，B 帧又称双向预测内插编码帧，解码需要参考前面的 I、P 帧和后面的 P 帧。

一共存在的这三种帧，他们在编码时不一定会按顺序写入视频流，因此在解码时为了获取不同帧的正确顺序，需要计算图片的顺序即 POC。

![](https://pic1.zhimg.com/v2-ccc3fbd147f6bdcd3ce7bb3d3eb8201c_b.jpg)

（StreamEye 解析后的 GOP POC 结果示意）

如上图 StreamEye 解析结果所示，POC 呈现：0 -> 4 -> 2 -> 1 -> 3 -> 8 -> 6 ... 规律。

不同帧的出现顺序对于解码来说至关重要，因此我们需要在不同帧解码后对帧按 POC 重新排序，最终确保解码图像按照实际顺序呈现给用户：0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8。

苹果的 VideoToolbox 并不会给我们实现这部分逻辑，因此我们需要自行计算 POC 顺序，并在之后重排序，代码实现如下：

```
// media/video/h265_poc.h

// Copyright 2022 The Chromium Authors. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.

#ifndef MEDIA_VIDEO_H265_POC_H_
#define MEDIA_VIDEO_H265_POC_H_

#include <stdint.h>

#include "third_party/abseil-cpp/absl/types/optional.h"

namespace media {

struct H265SPS;
struct H265PPS;
struct H265SliceHeader;

class MEDIA_EXPORT H265POC {
 public:
  H265POC();

  H265POC(const H265POC&) = delete;
  H265POC& operator=(const H265POC&) = delete;

  ~H265POC();

  // 根据SPS和PPS以及解析好的SliceHeader信息计算POC
  int32_t ComputePicOrderCnt(const H265SPS* sps,
                             const H265PPS* pps,
                             const H265SliceHeader& slice_hdr);
  void Reset();

 private:
  int32_t ref_pic_order_cnt_msb_;
  int32_t ref_pic_order_cnt_lsb_;
  // 是否为解码过程的首张图
  bool first_picture_;
};

}  // namespace media

#endif  // MEDIA_VIDEO_H265_POC_H_
```

POC 的计算逻辑：

```
// media/video/h265_poc.cc

// Copyright 2022 The Chromium Authors. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.

#include <stddef.h>

#include <algorithm>

#include "base/cxx17_backports.h"
#include "base/logging.h"
#include "media/video/h265_parser.h"
#include "media/video/h265_poc.h"

namespace media {

H265POC::H265POC() {
  Reset();
}

H265POC::~H265POC() = default;

void H265POC::Reset() {
  ref_pic_order_cnt_msb_ = 0;
  ref_pic_order_cnt_lsb_ = 0;
  first_picture_ = true;
}

// 如下逻辑所示，我们需要按照HEVC Spec的规范计算POC
//（这里我参考了Jeffery Kardatzke在H265Decoder的实现逻辑）
int32_t H265POC::ComputePicOrderCnt(const H265SPS* sps,
                                    const H265PPS* pps,
                                    const H265SliceHeader& slice_hdr) {
  int32_t pic_order_cnt = 0;
  int32_t max_pic_order_cnt_lsb =
      1 << (sps->log2_max_pic_order_cnt_lsb_minus4 + 4);
  int32_t pic_order_cnt_msb;
  int32_t no_rasl_output_flag;
  // Calculate POC for current picture.
  if (slice_hdr.irap_pic) {
    // 8.1.3
    no_rasl_output_flag = (slice_hdr.nal_unit_type >= H265NALU::BLA_W_LP &&
                           slice_hdr.nal_unit_type <= H265NALU::IDR_N_LP) ||
                          first_picture_;
  } else {
    no_rasl_output_flag = false;
  }

  if (!slice_hdr.irap_pic || !no_rasl_output_flag) {
    int32_t prev_pic_order_cnt_lsb = ref_pic_order_cnt_lsb_;
    int32_t prev_pic_order_cnt_msb = ref_pic_order_cnt_msb_;

    if ((slice_hdr.slice_pic_order_cnt_lsb < prev_pic_order_cnt_lsb) &&
        ((prev_pic_order_cnt_lsb - slice_hdr.slice_pic_order_cnt_lsb) >=
         (max_pic_order_cnt_lsb / 2))) {
      pic_order_cnt_msb = prev_pic_order_cnt_msb + max_pic_order_cnt_lsb;
    } else if ((slice_hdr.slice_pic_order_cnt_lsb > prev_pic_order_cnt_lsb) &&
               ((slice_hdr.slice_pic_order_cnt_lsb - prev_pic_order_cnt_lsb) >
                (max_pic_order_cnt_lsb / 2))) {
      pic_order_cnt_msb = prev_pic_order_cnt_msb - max_pic_order_cnt_lsb;
    } else {
      pic_order_cnt_msb = prev_pic_order_cnt_msb;
    }
  } else {
    pic_order_cnt_msb = 0;
  }

  // 8.3.1 Decoding process for picture order count.
  if (!pps->temporal_id && (slice_hdr.nal_unit_type < H265NALU::RADL_N ||
                            slice_hdr.nal_unit_type > H265NALU::RSV_VCL_N14)) {
    ref_pic_order_cnt_lsb_ = slice_hdr.slice_pic_order_cnt_lsb;
    ref_pic_order_cnt_msb_ = pic_order_cnt_msb;
  }

  pic_order_cnt = pic_order_cnt_msb + slice_hdr.slice_pic_order_cnt_lsb;
  first_picture_ = false;

  return pic_order_cnt;
}

}  // namespace media
```

### **计算 MaxReorderCount**

计算 POC 并解码后，为了确保视频帧按照正确的顺序展示给用户，需要对视频帧进行 Reorder 重排序，我们可以观察 H264 的最大 Reorder 数计算逻辑，发现很复杂：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

// H264最大Reorder数的计算逻辑
int32_t ComputeH264ReorderWindow(const H264SPS* sps) {
  // When |pic_order_cnt_type| == 2, decode order always matches presentation
  // order.
  // TODO(sandersd): For |pic_order_cnt_type| == 1, analyze the delta cycle to
  // find the minimum required reorder window.
  if (sps->pic_order_cnt_type == 2)
    return 0;

  int max_dpb_mbs = H264LevelToMaxDpbMbs(sps->GetIndicatedLevel());
  int max_dpb_frames =
      max_dpb_mbs / ((sps->pic_width_in_mbs_minus1 + 1) *
                     (sps->pic_height_in_map_units_minus1 + 1));
  max_dpb_frames = std::clamp(max_dpb_frames, 0, 16);

  // See AVC spec section E.2.1 definition of |max_num_reorder_frames|.
  if (sps->vui_parameters_present_flag && sps->bitstream_restriction_flag) {
    return std::min(sps->max_num_reorder_frames, max_dpb_frames);
  } else if (sps->constraint_set3_flag) {
    if (sps->profile_idc == 44 || sps->profile_idc == 86 ||
        sps->profile_idc == 100 || sps->profile_idc == 110 ||
        sps->profile_idc == 122 || sps->profile_idc == 244) {
      return 0;
    }
  }
  return max_dpb_frames;
}
```

幸运的是 HEVC 相比 H264 不需要如此繁杂的计算，HEVC 在编码时已经提前将最大 Reorder 数算好了，我们只需按如下方式获取：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

// HEVC最大Reorder数的计算逻辑
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
int32_t ComputeHEVCReorderWindow(const H265VPS* vps) {
  int32_t vps_max_sub_layers_minus1 = vps->vps_max_sub_layers_minus1;
  return vps->vps_max_num_reorder_pics[vps_max_sub_layers_minus1];
}
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
```

计算好 Reorder 数和 POC 后，继续复用 H264 的 Reorder 逻辑，即可正确完成排序。

### **提取并缓存 SPS / PPS / VPS**

下面我们正式开始解码逻辑实现，首先，需要提取 SPS / PPS / VPS，并对其解析，缓存:

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

switch (nalu.nal_unit_type) {
    // 跳过
    ...
    // 解析SPS
    case H265NALU::SPS_NUT: {
        int sps_id = -1;
        result = hevc_parser_.ParseSPS(&sps_id);
        if (result == H265Parser::kUnsupportedStream) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR, "Unsupported SPS");
            NotifyError(PLATFORM_FAILURE, SFT_UNSUPPORTED_STREAM);
            return;
        }
        if (result != H265Parser::kOk) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR, "Could not parse SPS");
            NotifyError(UNREADABLE_INPUT, SFT_INVALID_STREAM);
            return;
        }
        // 按照sps_id缓存SPS的nalu data
        seen_sps_[sps_id].assign(nalu.data, nalu.data + nalu.size);
        break;
    }
    // 解析PPS
    case H265NALU::PPS_NUT: {
        int pps_id = -1;
        result = hevc_parser_.ParsePPS(nalu, &pps_id);
        if (result == H265Parser::kUnsupportedStream) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR, "Unsupported PPS");
            NotifyError(PLATFORM_FAILURE, SFT_UNSUPPORTED_STREAM);
            return;
        }
        if (result == H265Parser::kMissingParameterSet) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR,
                            "Missing SPS from what was parsed");
            NotifyError(PLATFORM_FAILURE, SFT_INVALID_STREAM);
            return;
        }
        if (result != H265Parser::kOk) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR, "Could not parse PPS");
            NotifyError(UNREADABLE_INPUT, SFT_INVALID_STREAM);
            return;
        }
        // 按照pps_id缓存PPS的nalu data
        seen_pps_[pps_id].assign(nalu.data, nalu.data + nalu.size);
        // 将PPS同样作为提交到VT的一部分，这可以解决同一个GOP下不同帧引用不同PPS的问题
        nalus.push_back(nalu);
        data_size += kNALUHeaderLength + nalu.size;
        break;
    }
    // 解析VPS
    case H265NALU::VPS_NUT: {
        int vps_id = -1;
        result = hevc_parser_.ParseVPS(&vps_id);
        if (result == H265Parser::kUnsupportedStream) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR, "Unsupported VPS");
            NotifyError(PLATFORM_FAILURE, SFT_UNSUPPORTED_STREAM);
            return;
        }
        if (result != H265Parser::kOk) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR, "Could not parse VPS");
            NotifyError(UNREADABLE_INPUT, SFT_INVALID_STREAM);
            return;
        }
        // 按照vps_id缓存VPS的nalu data
        seen_vps_[vps_id].assign(nalu.data, nalu.data + nalu.size);
        break;
    }
    // 跳过
    ...
}
```

### **创建解码 Format 和 Session**

根据解析后的 VPS，SPS，PPS，我们可以创建解码 Format：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
// 使用vps，sps，pps创建解码Format（CMFormatDescriptionRef）
base::ScopedCFTypeRef<CMFormatDescriptionRef> CreateVideoFormatHEVC(
    const std::vector<uint8_t>& vps,
    const std::vector<uint8_t>& sps,
    const std::vector<uint8_t>& pps) {
  DCHECK(!vps.empty());
  DCHECK(!sps.empty());
  DCHECK(!pps.empty());

  // Build the configuration records.
  std::vector<const uint8_t*> nalu_data_ptrs;
  std::vector<size_t> nalu_data_sizes;
  nalu_data_ptrs.reserve(3);
  nalu_data_sizes.reserve(3);
  nalu_data_ptrs.push_back(&vps.front());
  nalu_data_sizes.push_back(vps.size());
  nalu_data_ptrs.push_back(&sps.front());
  nalu_data_sizes.push_back(sps.size());
  nalu_data_ptrs.push_back(&pps.front());
  nalu_data_sizes.push_back(pps.size());

  // 这里有一个关键点，即，在一个 GOP 内可能存在 >= 2 的引用情况、
  // 比如I帧引用了 pps_id 为 0 的 pps，P帧引用了 pps_id 为 1 的 pps
  // 这种场景经过本人测试，解决方法有两个：
  // 方法1：把两个PPS都传进来，以此创建 CMFormatDescriptionRef（此时nalu_data_ptrs数组长度为4）
  // 方法2（本文选用的方法）：仍然只传一个PPS，但把 PPS 的 Nalu Data 提交到 VT，VT 会自动查找到PPS的引用关系，并处理这种情况，见"vt_video_decode_accelerator_mac.cc;l=1380"
  base::ScopedCFTypeRef<CMFormatDescriptionRef> format;
  if (__builtin_available(macOS 11.0, *)) {
    OSStatus status = CMVideoFormatDescriptionCreateFromHEVCParameterSets(
        kCFAllocatorDefault,
        nalu_data_ptrs.size(),     // parameter_set_count
        &nalu_data_ptrs.front(),   // &parameter_set_pointers
        &nalu_data_sizes.front(),  // &parameter_set_sizes
        kNALUHeaderLength,         // nal_unit_header_length
        extensions, format.InitializeInto());
    OSSTATUS_LOG_IF(WARNING, status != noErr, status)
        << "CMVideoFormatDescriptionCreateFromHEVCParameterSets()";
  }
  return format;
}
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
```

![](https://pic4.zhimg.com/v2-1a035a6eb705552a71bab5f39cbcc9a3_b.jpg)

VideoToolbox 的解码流程，图片来自 Apple

在创建解码 Format 后，继续创建解码 Session：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

bool VTVideoDecodeAccelerator::ConfigureDecoder() {
  DVLOG(2) << __func__;
  DCHECK(decoder_task_runner_->RunsTasksInCurrentSequence());

  base::ScopedCFTypeRef<CMFormatDescriptionRef> format;
  switch (codec_) {
    case VideoCodec::kH264:
      format = CreateVideoFormatH264(active_sps_, active_spsext_, active_pps_);
      break;
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
    case VideoCodec::kHEVC:
      // 创建CMFormatDescriptionRef
      format = CreateVideoFormatHEVC(active_vps_, active_sps_, active_pps_);
      break;
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
    case VideoCodec::kVP9:
      format = CreateVideoFormatVP9(
          cc_detector_->GetColorSpace(config_.container_color_space),
          config_.profile, config_.hdr_metadata,
          cc_detector_->GetCodedSize(config_.initial_expected_coded_size));
      break;
    default:
      NOTREACHED() << "Unsupported codec.";
  }

  if (!format) {
    NotifyError(PLATFORM_FAILURE, SFT_PLATFORM_ERROR);
    return false;
  }

  if (!FinishDelayedFrames())
    return false;

  format_ = format;
  session_.reset();

  // 利用创建好的解码format创建解码session
  // 如果是VP9，则强制请求硬解解码
  // 如果是HEVC，由于一些可能的原因，我们选择不强制硬解解码（让VT自己选最适合的解码方式）
  // 可能的原因有：
  // 1. GPU不支持硬解（此时我们希望使用VT软解）
  // 2. 解码的Profile不受支持（比如M1支持HEVC Rext硬解，而Intel/AMD GPU不支持，此时希望软解）
  // 3. GPU繁忙，资源不足，此时希望软解
  const bool require_hardware = config_.profile == VP9PROFILE_PROFILE0 ||
                                config_.profile == VP9PROFILE_PROFILE2;

  // 可能是HDR视频，因此希望输出pix_fmt是
  // kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange
  const bool is_hbd = config_.profile == VP9PROFILE_PROFILE2 ||
                      config_.profile == HEVCPROFILE_MAIN10 ||
                      config_.profile == HEVCPROFILE_REXT;
  // 创建解码Session
  if (!CreateVideoToolboxSession(format_, require_hardware, is_hbd, &callback_,
                                 &session_, &configured_size_)) {
    NotifyError(PLATFORM_FAILURE, SFT_PLATFORM_ERROR);
    return false;
  }

  // Report whether hardware decode is being used.
  bool using_hardware = false;
  base::ScopedCFTypeRef<CFBooleanRef> cf_using_hardware;
  if (VTSessionCopyProperty(
          session_,
          // kVTDecompressionPropertyKey_UsingHardwareAcceleratedVideoDecoder
          CFSTR("UsingHardwareAcceleratedVideoDecoder"), kCFAllocatorDefault,
          cf_using_hardware.InitializeInto()) == 0) {
    using_hardware = CFBooleanGetValue(cf_using_hardware);
  }
  UMA_HISTOGRAM_BOOLEAN("Media.VTVDA.HardwareAccelerated", using_hardware);

  if (codec_ == VideoCodec::kVP9 && !vp9_bsf_)
    vp9_bsf_ = std::make_unique<VP9SuperFrameBitstreamFilter>();

  // Record that the configuration change is complete.
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  configured_vps_ = active_vps_;
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  configured_sps_ = active_sps_;
  configured_spsext_ = active_spsext_;
  configured_pps_ = active_pps_;
  return true;
}
```

创建解码 Session 的逻辑：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

// 利用CMFormatDescriptionRef创建VTDecompressionSession
bool CreateVideoToolboxSession(
    const CMFormatDescriptionRef format,
    bool require_hardware,
    bool is_hbd,
    const VTDecompressionOutputCallbackRecord* callback,
    base::ScopedCFTypeRef<VTDecompressionSessionRef>* session,
    gfx::Size* configured_size) {
  // Prepare VideoToolbox configuration dictionaries.
  base::ScopedCFTypeRef<CFMutableDictionaryRef> decoder_config(
      CFDictionaryCreateMutable(kCFAllocatorDefault,
                                1,  // capacity
                                &kCFTypeDictionaryKeyCallBacks,
                                &kCFTypeDictionaryValueCallBacks));
  if (!decoder_config) {
    DLOG(ERROR) << "Failed to create CFMutableDictionary";
    return false;
  }

  CFDictionarySetValue(
      decoder_config,
      kVTVideoDecoderSpecification_EnableHardwareAcceleratedVideoDecoder,
      kCFBooleanTrue);
  CFDictionarySetValue(
      decoder_config,
      kVTVideoDecoderSpecification_RequireHardwareAcceleratedVideoDecoder,
      require_hardware ? kCFBooleanTrue : kCFBooleanFalse);

  CGRect visible_rect = CMVideoFormatDescriptionGetCleanAperture(format, true);
  CMVideoDimensions visible_dimensions = {
      base::ClampFloor(visible_rect.size.width),
      base::ClampFloor(visible_rect.size.height)};
  base::ScopedCFTypeRef<CFMutableDictionaryRef> image_config(
      BuildImageConfig(visible_dimensions, is_hbd));
  if (!image_config) {
    DLOG(ERROR) << "Failed to create decoder image configuration";
    return false;
  }

  // 创建解码Session的最终逻辑
  OSStatus status = VTDecompressionSessionCreate(
      kCFAllocatorDefault,
      format,          // 我们创建好的CMFormatDescriptionRef
      decoder_config,  // video_decoder_specification
      image_config,    // destination_image_buffer_attributes
      callback,        // output_callback
      session->InitializeInto());
  if (status != noErr) {
    OSSTATUS_DLOG(WARNING, status) << "VTDecompressionSessionCreate()";
    return false;
  }

  *configured_size =
      gfx::Size(visible_rect.size.width, visible_rect.size.height);

  return true;
}
```

### **提取视频帧并解码**

这一步开始我们就要开始正式解码了，解码前首先需要提取视频帧的 SliceHeader，并从缓存中拿到到该帧引用的 SPS，PPS，VPS，计算 POC 和最大 Reorder 数。

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

switch (nalu.nal_unit_type) {
    case H265NALU::BLA_W_LP:
    case H265NALU::BLA_W_RADL:
    case H265NALU::BLA_N_LP:
    case H265NALU::IDR_W_RADL:
    case H265NALU::IDR_N_LP:
    case H265NALU::TRAIL_N:
    case H265NALU::TRAIL_R:
    case H265NALU::TSA_N:
    case H265NALU::TSA_R:
    case H265NALU::STSA_N:
    case H265NALU::STSA_R:
    case H265NALU::RADL_N:
    case H265NALU::RADL_R:
    case H265NALU::RASL_N:
    case H265NALU::RASL_R:
    case H265NALU::CRA_NUT: {
        // 针对视频帧提取SliceHeader
        curr_slice_hdr.reset(new H265SliceHeader());
        result = hevc_parser_.ParseSliceHeader(nalu, curr_slice_hdr.get(),
                                                last_slice_hdr.get());

        if (result == H265Parser::kMissingParameterSet) {
            curr_slice_hdr.reset();
            last_slice_hdr.reset();
            WriteToMediaLog(MediaLogMessageLevel::kERROR,
                            "Missing PPS when parsing slice header");
            continue;
        }

        if (result != H265Parser::kOk) {
            curr_slice_hdr.reset();
            last_slice_hdr.reset();
            WriteToMediaLog(MediaLogMessageLevel::kERROR,
                            "Could not parse slice header");
            NotifyError(UNREADABLE_INPUT, SFT_INVALID_STREAM);
            return;
        }

        // 这里是一个Workaround，一些iOS设备拍摄的视频如果在Seek过程首个关键帧是CRA帧，
        // 那么下一个帧如果是一个RASL帧，则会立即报kVTVideoDecoderBadDataErr的错误，
        // 因此我们需要判断总输出帧数是否大于5，否则跳过这些RASL帧
        if (output_count_for_cra_rasl_workaround_ < kMinOutputsBeforeRASL &&
            (nalu.nal_unit_type == H265NALU::RASL_N ||
                nalu.nal_unit_type == H265NALU::RASL_R)) {
            continue;
        }

        // 根据SliceHeader内的pps_id，拿到缓存的pps nalu data
        const H265PPS* pps =
            hevc_parser_.GetPPS(curr_slice_hdr->slice_pic_parameter_set_id);
        if (!pps) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR,
                            "Missing PPS referenced by slice");
            NotifyError(UNREADABLE_INPUT, SFT_INVALID_STREAM);
            return;
        }

        // 根据PPS内的sps_id，拿到缓存的sps nalu data
        const H265SPS* sps = hevc_parser_.GetSPS(pps->pps_seq_parameter_set_id);
        if (!sps) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR,
                            "Missing SPS referenced by PPS");
            NotifyError(UNREADABLE_INPUT, SFT_INVALID_STREAM);
            return;
        }

        // 根据VPS内的vps_id，拿到缓存的vps nalu data
        const H265VPS* vps =
            hevc_parser_.GetVPS(sps->sps_video_parameter_set_id);
        if (!vps) {
            WriteToMediaLog(MediaLogMessageLevel::kERROR,
                            "Missing VPS referenced by SPS");
            NotifyError(UNREADABLE_INPUT, SFT_INVALID_STREAM);
            return;
        }

        // 记录一下当前激活的sps/vps/pps
        DCHECK(seen_pps_.count(curr_slice_hdr->slice_pic_parameter_set_id));
        DCHECK(seen_sps_.count(pps->pps_seq_parameter_set_id));
        DCHECK(seen_vps_.count(sps->sps_video_parameter_set_id));
        active_vps_ = seen_vps_[sps->sps_video_parameter_set_id];
        active_sps_ = seen_sps_[pps->pps_seq_parameter_set_id];
        active_pps_ = seen_pps_[curr_slice_hdr->slice_pic_parameter_set_id];

        // 计算POC
        int32_t pic_order_cnt =
            hevc_poc_.ComputePicOrderCnt(sps, pps, *curr_slice_hdr.get());

        frame->has_slice = true;
        // 是否为IDR（这里其实为IRAP）
        frame->is_idr = nalu.nal_unit_type >= H265NALU::BLA_W_LP &&
                        nalu.nal_unit_type <= H265NALU::RSV_IRAP_VCL23;
        frame->pic_order_cnt = pic_order_cnt;
        // 计算最大Reorder数
        frame->reorder_window = ComputeHEVCReorderWindow(vps);

        // 存储上一帧的SliceHeader
        last_slice_hdr.swap(curr_slice_hdr);
        curr_slice_hdr.reset();
        [[fallthrough]];
    }
    default:
        nalus.push_back(nalu);
        data_size += kNALUHeaderLength + nalu.size;
        break;
}
```

### **检测视频参数是否发生变化**

![](https://pic4.zhimg.com/v2-c3a7d2e44c49991d37512bec7d34aadf_b.jpg)

H264 视频帧的 SPS、PPS 引用关系，图片来自 Apple

如果视频帧引用的 VPS，PPS，SPS 任一发生变化，则按照 VideoToolbox 的要求，需要重新配置解码 Session：

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

if (frame->is_idr &&
      (configured_vps_ != active_vps_ || configured_sps_ != active_sps_ ||
       configured_pps_ != active_pps_)) {

    // 这里是一些校验逻辑
    ...

    // 这里重新创建解码format，并重新配置解码session
    if (!ConfigureDecoder()) {
      return;
    }
}
```

### **设置输出视频帧的目标像素格式**

鉴于我们需要支持 HDR，因此需要判断一下视频是否为 Main10 / Rext Profile，并调整输出为`gfx::BufferFormat::P010`

```
// media/gpu/mac/vt_video_decode_accelerator_mac.cc

const gfx::BufferFormat buffer_format =
      config_.profile == VP9PROFILE_PROFILE2 ||
              config_.profile == HEVCPROFILE_MAIN10 ||
              config_.profile == HEVCPROFILE_REXT
          ? gfx::BufferFormat::P010
          : gfx::BufferFormat::YUV_420_BIPLANAR;
```

### **总结**

在上述步骤后，硬解关键流程基本完工，目前代码已合入 Chromium 104（main 分支），macOS 平台具体实现过程和代码 Diff 可以追溯 Crbug（[https://bugs.chromium.org/p/chromium/issues/detail?id=1300444](https://link.zhihu.com/?target=https%3A//bugs.chromium.org/p/chromium/issues/detail%3Fid%3D1300444)）。

### **Windows 的硬解**

有了 macOS 硬解的开发经验，尝试 Windows 硬解相对变得容易了一些，尽管也踩了一些坑。

### **Media Foundation 方案的尝试**

文章开头已经介绍了，实际上在 Windows 平台，如果你可以安装 `HEVC视频扩展`，则是可以在 Edge 浏览器硬解 HEVC 的，因此我最初的思路也是和 Edge 一样，通过引导 `HEVC视频扩展`，完成硬解支持。

首先，使用 Edge，打开任意 HEVC 视频，发现，Edge 使用 VDAVideoDecoder 进行 HEVC 硬解：

![](https://pic2.zhimg.com/v2-0ce614a33b0fdab56b6cfb2285dc1485_b.jpg)

因此尝试搜索发现 Windows 平台 VDAVideoDecoder 代码实现均位于`dxva_video_decode_accelerator_win.cc`文件内，继续寻找蛛丝马迹，发现，在开源的 Chromium 项目内，并不存在 HEVC 硬解相关的任何实现，这说明 Edge 是自己基于某个时期的 Chromium Media 模块“魔改”出来的 HEVC 硬解支持，同时在这个过程发现了个有趣的现象：

1.  Edge 完全不使用 `D3D11VideoDecoder` 进行解码，而是使用 `VDAVideoDecoder` 解码，我猜测其目的是为了推广自家的 Media Foundation。
2.  Edge 的色彩处理有问题 (Edge 102)，与 Chrome 不一样的点在于，比如对于 Transfer 是 PQ 的 HDR 视频，Edge 并没有对其进行 Tone Mapping，导致 PQ 视频在 Edge 下看起来有“过曝”的问题。

![](https://pic4.zhimg.com/v2-22dc165e1fcadfad3da1c6f36992457b_b.jpg)

3\. Edge 解码 AV1 需要装 `AV1 Video Extension` ，才可解码 AV1（软解+硬解)，而 Chrome 由于实现了 AV1 的 D3D11VA 硬解，以及 `Gav1VideoDecoder` 和 `DAV1dVideoDecoder` 软解 Decoder，不需要安装 AV1 插件也可在受支持的显卡硬解，不受支持的显卡软解。

![](https://pic3.zhimg.com/v2-c2859226268b05d6283f2911528da7e6_b.jpg)

好吧，尽管没少吐槽 Edge，但是他确实是 Windows 平台唯一支持 HEVC 硬解的浏览器（当然，马上就不是了）。

接着看 `dxva_video_decode_accelerator_win.cc` 的实现，从上述 Edge 解码需要安装 AV1 插件的逻辑反推，如果我们照着 AV1 的方式实现 HEVC，是否可行？答案是肯定的。

观察 Supported Profile，然后将我们需要支持的 HEVCPROFILE\_MAIN、HEVCPROFILE\_MAIN10 加入：

```
// media/gpu/windows/dxva_video_decode_accelerator_win.cc

// 我们可以看到与macOS类似，VDAVideoDecoder支持的格式都被放到了Supported Profiles内
// 如下，一目了然，VDAVideoDecoder原始支持H264，VP8，VP9，AV1
constexpr VideoCodecProfile kSupportedProfiles[] = {
    H264PROFILE_BASELINE,    H264PROFILE_MAIN,        H264PROFILE_HIGH,
    VP8PROFILE_ANY,          VP9PROFILE_PROFILE0,     VP9PROFILE_PROFILE2,
    AV1PROFILE_PROFILE_MAIN, AV1PROFILE_PROFILE_HIGH, AV1PROFILE_PROFILE_PRO,
    // 添加我们需要支持的两种Profile
    HEVCPROFILE_MAIN,        HEVCPROFILE_MAIN10,
 };
```

之后按照 AV1 的逻辑，加入 HEVC Codec，同时值得一提的是必须在调用 SetOutput 方法前设置分辨率（这块坑了我大概一天的时间 Debug），代码如下：

```
// media/gpu/windows/dxva_video_decode_accelerator_win.cc

...
if (config.profile == VP9PROFILE_PROFILE2 ||
      config.profile == VP9PROFILE_PROFILE3 ||
      config.profile == H264PROFILE_HIGH10PROFILE) {
    // Input file has more than 8 bits per channel.
    use_fp16_ = true;
    decoder_output_p010_or_p016_ = true;
    // the OS for VP9 which is why it works and AV1 doesn't.
    HRESULT hr = CreateAV1Decoder(IID_PPV_ARGS(&decoder_));
    RETURN_ON_HR_FAILURE(hr, "Failed to create decoder instance", false);
  } else if (profile >= HEVCPROFILE_MAIN && profile <= HEVCPROFILE_MAIN10) {

    codec_ = kCodecHEVC;
    clsid = CLSID_MSH265DecoderMFT;
    // 这里必须提前设置分辨率，否则SetOutput会失败
    using_ms_vpx_mft_ = true;

    // 经过各种探索发现只有1.0.31823版本的HEVC视频扩展没有抖动问题，
    // 其他情况包括最新版本（1.0.51361.0）的HEVC视频扩展，依然存在解码跳帧问题
    // 显然如果希望1.0.51361版本插件正常解码，需要额外的配置，但无法从官网文档找到配置方法
    HRESULT hr = CreateHEVCDecoder(IID_PPV_ARGS(&decoder_));
    RETURN_ON_HR_FAILURE(hr, "Failed to create hevc decoder instance", false);
  } else {
    if (!decoder_dll)
      RETURN_ON_FAILURE(false, "Unsupported codec.", false);
  HRESULT hr = MFCreateMediaType(&media_type);
  RETURN_ON_HR_FAILURE(hr, "MFCreateMediaType failed", false);

  // 设置主类型，参考：https://docs.microsoft.com/en-us/windows/win32/medfound/mf-mt-major-type-attribute
  hr = media_type->SetGUID(MF_MT_MAJOR_TYPE, MFMediaType_Video);
  RETURN_ON_HR_FAILURE(hr, "Failed to set major input type", false);

  if (codec_ == kCodecH264) {
  // 设置辅类型，参考：https://docs.microsoft.com/en-us/windows/win32/medfound/video-subtype-guids
  if (codec_ == kCodecHEVC) {
    hr = media_type->SetGUID(MF_MT_SUBTYPE, MFVideoFormat_HEVC);
  } else if (codec_ == kCodecH264) {
    hr = media_type->SetGUID(MF_MT_SUBTYPE, MFVideoFormat_H264);
  } else if (codec_ == kCodecVP9) {
    hr = media_type->SetGUID(MF_MT_SUBTYPE, MEDIASUBTYPE_VP90);
```

接着实现一下 HEVCDecoder 的获取逻辑：

```
// media/gpu/windows/dxva_video_decode_accelerator_win.cc

// 不同于H264等格式，由于HEVC是以可选插件形式支持的，因此直接读取DLL的方式并不可行
// 参考AV1的实现方法，以及微软的官方文档，使用::MFTEnumEx方法，最终可以拿到HEVCDecoder
HRESULT CreateHEVCDecoder(const IID& iid, void** object) {
  MFT_REGISTER_TYPE_INFO type_info = {MFMediaType_Video, MFVideoFormat_HEVC};

  base::win::ScopedCoMem<IMFActivate*> acts;
  UINT32 acts_num;
  HRESULT hr =
      ::MFTEnumEx(MFT_CATEGORY_VIDEO_DECODER, MFT_ENUM_FLAG_SORTANDFILTER,
                  &type_info, nullptr, &acts, &acts_num);
  if (FAILED(hr))
    return hr;

  if (acts_num < 1)
    return E_FAIL;

  hr = acts[0]->ActivateObject(iid, object);
  for (UINT32 i = 0; i < acts_num; ++i)
    acts[i]->Release();
  return hr;
}
```

同时将 HEVC Main, Main10 Profile 加入到 `supported_profile_helpers.cc`:

```
// media/gpu/windows/supported_profile_helpers.cc

// 对Windows10 1709以上版本添加HEVC硬解支持：
if (base::win::GetVersion() >= base::win::Version::WIN10_RS2) {
    if (profile_id == D3D11_DECODER_PROFILE_HEVC_VLD_MAIN) {
        supported_resolutions[HEVCPROFILE_MAIN] = GetResolutionsForGUID(
            video_device.Get(), profile_id, kModernResolutions, DXGI_FORMAT_NV12);
        continue;
    }
    if (profile_id == D3D11_DECODER_PROFILE_HEVC_VLD_MAIN10) {
        supported_resolutions[HEVCPROFILE_MAIN10] = GetResolutionsForGUID(
            video_device.Get(), profile_id, kModernResolutions, DXGI_FORMAT_P010);
        continue;
    }
}
```

最后，还需要修改一下引导逻辑，强制让 HEVC 编码格式使用 VDAVideoDecoder 而不是 D3D11VideoDecoder：

```
// media/mojo/services/gpu_mojo_media_client_win.cc

...
// 强制令HEVC编码格式的视频使用VDAVideoDecoder解码
std::unique_ptr<VideoDecoder> CreatePlatformVideoDecoder(
    const VideoDecoderTraits& traits) {
   if (!ShouldUseD3D11VideoDecoder(*traits.gpu_workarounds) || (
      config.profile() >= HEVCPROFILE_MAIN &&
      config.profile() <= HEVCPROFILE_MAIN10)) {
    if (traits.gpu_workarounds->disable_dxva_video_decoder)
      return nullptr;
    return VdaVideoDecoder::Create(
...
```

后面省略了一些透传 Profile 的代码。

在上述步骤执行后，一切大功告成，代码实现基本完成，`HEVC视频扩展`帮我们处理了大部分的解码逻辑，所以实现过程相当简单。

但，问题来了！由于`HEVC视频扩展`插件在 1.0.31823 之后的版本存在抖动问题，而 1.0.50361 虽然解决了抖动的问题，但其官网文档并没有明确详述如何配置 Decoder 解决该问题（注：欢迎贡献配置方法），因此，如果我们需要用`HEVC视频扩展`的方案，则必须限制用户本地强行使用 1.0.31823 版本。

为此我尝试写过 nsh 脚本，试图在用户电脑存在非 31823 版本`HEVC视频扩展`的情况，强制卸载并重装 1.0.31823 版本的 `HEVC视频扩展`，但，因为 Windows 商店会默认对 Appx 扩展自动更新，这导致如果希望用户电脑不更新`HEVC视频扩展`， 则必须强迫用户关闭 `Windows 商店`的自动更新，这无疑意味着这个方案是个半成品，很可能需要换技术方案了，但我们的选择真的不多。

然而就在即将放弃的时候，我突然想到为啥不照着 Chromium 把 D3D11VA 的方案实现一遍呢？Media Foundation 绝不是唯一解！时间点是 2022 年的 2 月中旬，我抱着尝试的态度，打开了 [http://source.chromium.org](https://link.zhihu.com/?target=http%3A//source.chromium.org) 这个网站，试图学习下其他格式 D3D11VA 的解码方法，并在 media 文件夹偶然间瞥到了一个叫 `d3d11_h265_accelerator.cc` 的文件，这是啥？怎么可能？Windows 不是没有人实现过 HEVC 硬解么？然后我果断看了下提交时间，发现在 2 月 8 号，这个文件才被合入到 Chromium！感谢作者 @Jianlin Qiu（来自 Intel 的大佬），把 Windows 的 D3D11 硬解加速实现的差不多。

### **使用 D3D11VA 硬解**

Trace 了下 @Jianlin Qiu 实现相关的 crbug（[https://bugs.chromium.org/p/chromium/issues/detail?id=1286132#c18](https://link.zhihu.com/?target=https%3A//bugs.chromium.org/p/chromium/issues/detail%3Fid%3D1286132%23c18)），合入其代码，简单做了下测试发现一半的视频可以播（早期版本有些小问题，目前均已解决）。

遂观察其实现逻辑，发现 Windows 的硬解实现逻辑与 macOS 完全不同，在 macOS，尽管我会对 SPS / PPS / VPS / Slice Header 进行 Parse，但是实际上，最终调用`CMVideoFormatDescriptionCreateFromHEVCParameterSets` 方法创建解码 Format 时，传给 VT 的参数是包含了 VPS, SPS, PPS 的 Nalu Data 的数组，也就是说理论上如果我们不计算 POC，不 Reorder，直接将 Nalu Data 塞给 VideoToolbox，也可以解码，只是帧组会抖动罢了。

但到了 Windows 和 Linux 这里，实现起来要麻烦的多。

根据 Mircosoft 官网，可知 D3D11VA 硬解的实际流程可参考这篇文章（[https://docs.microsoft.com/en-us/windows/win32/medfound/supporting-direct3d-11-video-decoding-in-media-foundation#open-a-device-handle](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/windows/win32/medfound/supporting-direct3d-11-video-decoding-in-media-foundation%23open-a-device-handle)），总结起来其实主要工作在于对每一个视频帧的图片参数拼装。

### **GPU 是否支持硬解检测**

GPU 是否支持硬解，这里的逻辑，首先假定默认是不支持 HEVC Main / Main10 的，然后调用 D3D11 Device 提供的 GetVideoDecoderConfig 方法，拿到支持的 Codec 列表，若列表中存在 HEVC 则认为支持：

```
// media/gpu/windows/d3d11_video_decoder.cc

D3D11_VIDEO_DECODER_CONFIG dec_config = {};
bool found = false;

for (UINT i = 0; i < config_count; i++) {
// 调用该方法，d3d11会返回其所支持的全部codec类型
hr = video_device_->GetVideoDecoderConfig(
    decoder_configurator_->DecoderDescriptor(), i, &dec_config);
if (FAILED(hr))
    return {D3D11Status::Codes::kGetDecoderConfigFailed, hr};

if (dec_config.ConfigBitstreamRaw == 1 &&
    (config_.codec() == VideoCodec::kVP9 ||
        config_.codec() == VideoCodec::kAV1 ||
        config_.codec() == VideoCodec::kHEVC)) {
    // DXVA HEVC, VP9, and AV1 specifications say ConfigBitstreamRaw
    // "shall be 1".
    // 如果类型中有HEVC类型，且ConfigBitstreamRaw == 1，则显卡支持硬解
    found = true;
    break;
}

if (config_.codec() == VideoCodec::kH264 &&
    dec_config.ConfigBitstreamRaw == 2) {
    // ConfigBitstreamRaw == 2 means the decoder uses DXVA_Slice_H264_Short.
    found = true;
    break;
}
}
if (!found)
    return D3D11Status::Codes::kDecoderUnsupportedConfig;
```

### **理解 DXVA HEVC Spec**

如上流程可知，根据 DXVA HEVC Spec，要正确实现解码，需要自己提前解析好其要的 Picture Params，以 HEVC 的 Picture Params 为例，结构体如下，每一个参数都不能缺少，这本身工作量就不小，但好在 @Jeffery 大佬在实现 Linux 的 H265 Decoder 和 H265 Parser 时已经完成了大部分工作，因此 @Jianlin 大佬的工作主要是如何正确的将这些已经 Parse 好的 Params 拼装和计算，并塞给 D3D11。

```
// 诚然，如果每个软件都要实现一遍拼装逻辑，成本高的离谱
// 相比macOS的API设计，DXVA规范设计的非常复杂
// 但我相信其一定有自己的理由

typedef struct _DXVA_PicEntry_HEVC {
  union {
    struct {
      UCHAR  Index7Bits  :7;
      UCHAR  AssociatedFlag   :1;
    };
    UCHAR  bPicEntry;
  };
} DXVA_PicEntry_HEVC, *PDXVA_PicEntry_HEVC;

typedef struct _DXVA_PicParams_HEVC {
  USHORT             PicWidthInMinCbsY;
  USHORT             PicHeightInMinCbsY;
  union {
    struct {
      USHORT chroma_format_idc  :2;
      USHORT separate_colour_plane_flag   :1;
      USHORT bit_depth_luma_minus8   :3;
      USHORT bit_depth_chroma_minus8  :3;
      USHORT log2_max_pic_order_cnt_lsb_minus4  :4;
      USHORT NoPicReorderingFlag   :1;
      USHORT  NoBiPredFlag   :1;
      USHORT ReservedBits1     :1;
    };
    USHORT  wFormatAndSequenceInfoFlags;
  };
  DXVA_PicEntry_HEVC CurrPic;
  UCHAR              sps_max_dec_pic_buffering_minus1;
  UCHAR              log2_min_luma_coding_block_size_minus3;
  UCHAR              log2_diff_max_min_luma_coding_block_size;
  UCHAR              log2_min_transform_block_size_minus2;
  UCHAR              log2_diff_max_min_transform_block_size;
  UCHAR              max_transform_hierarchy_depth_inter;
  UCHAR              max_transform_hierarchy_depth_intra;
  UCHAR              num_short_term_ref_pic_sets;
  UCHAR              num_long_term_ref_pics_sps;
  UCHAR              num_ref_idx_l0_default_active_minus1;
  UCHAR              num_ref_idx_l1_default_active_minus1;
  CHAR               init_qp_minus26;
  UCHAR              ucNumDeltaPocsOfRefRpsIdx;
  USHORT             wNumBitsForShortTermRPSInSlice;
  USHORT             ReservedBits2;
  union {
    struct {
      UINT32 scaling_list_enabled_flag  :1;
      UINT32 amp_enabled_flag  :1;
      UINT32 sample_adaptive_offset_enabled_flag  :1;
      UINT32 pcm_enabled_flag   :1;
      UINT32 pcm_sample_bit_depth_luma_minus1   :4;
      UINT32 pcm_sample_bit_depth_chroma_minus1     :4;
      UINT32 log2_min_pcm_luma_coding_block_size_minus3    :2;
      UINT32 log2_diff_max_min_pcm_luma_coding_block_size  :2;
      UINT32 pcm_loop_filter_disabled_flag  :1;
      UINT32 long_term_ref_pics_present_flag   :1;
      UINT32 sps_temporal_mvp_enabled_flag  :1;
      UINT32 strong_intra_smoothing_enabled_flag   :1;
      UINT32 dependent_slice_segments_enabled_flag    :1;
      UINT32 output_flag_present_flag   :1;
      UINT32 num_extra_slice_header_bits    :3;
      UINT32 sign_data_hiding_enabled_flag  :1;
      UINT32 cabac_init_present_flag  :1;
      UINT32 ReservedBits3    :5;
    };
    UINT32             dwCodingParamToolFlags;
    union {
      struct {
        UINT32 constrained_intra_pred_flag  :1;
        UINT32 transform_skip_enabled_flag  :1;
        UINT32 cu_qp_delta_enabled_flag  :1;
        UINT32 pps_slice_chroma_qp_offsets_present_flag  :1;
        UINT32 weighted_pred_flag  :1;
        UINT32 weighted_bipred_flag  :1;
        UINT32 transquant_bypass_enabled_flag  :1;
        UINT32 tiles_enabled_flag   :1;
        UINT32 entropy_coding_sync_enabled_flag   :1;
        UINT32 uniform_spacing_flag    :1;
        UINT32 loop_filter_across_tiles_enabled_flag   :1;
        UINT32 pps_loop_filter_across_slices_enabled_flag  :1;
        UINT32 deblocking_filter_override_enabled_flag  :1;
        UINT32 pps_deblocking_filter_disabled_flag  :1;
        UINT32 lists_modification_present_flag  :1;
        UINT32 slice_segment_header_extension_present_flag  :1;
        UINT32 IrapPicFlag  :1;
        UINT32 IdrPicFlag     :1;
        UINT32 IntraPicFlag   :1;
        UINT32 ReservedBits4     :13;
      };
      UINT32   dwCodingSettingPicturePropertyFlags;
    };
    CHAR               pps_cb_qp_offset;
    CHAR               pps_cr_qp_offset;
    UCHAR              num_tile_columns_minus1;
    UCHAR              num_tile_rows_minus1;
    USHORT             column_width_minus1[19];
    USHORT             row_height_minus1[21];
    UCHAR              diff_cu_qp_delta_depth;
    CHAR               pps_beta_offset_div2;
    CHAR               pps_tc_offset_div2;
    UCHAR              log2_parallel_merge_level_minus2;
    INT                CurrPicOrderCntVal;
    DXVA_PicEntry_HEVC RefPicList[15];
    UCHAR              ReservedBits5;
    INT                PicOrderCntValList[15];
    UCHAR              RefPicSetStCurrBefore[8];
    UCHAR              RefPicSetStCurrAfter[8];
    UCHAR              RefPicSetLtCurr[8];
    USHORT             ReservedBits6;
    USHORT             ReservedBits7;
    UINT               StatusReportFeedbackNumber;
  };
} DXVA_PicParams_HEVC, *PDXVA_PicParams_HEVC;
```

### **填充默认 Picture Params**

实现硬解加速本身不需要实现解码逻辑，因此其实 `H265Accelerator` 本身的功能主要在于拼装 DXVA 所要的 Picture Params，并正确提交。这个过程，首先需要填充默认的 Picture Params：

```
// media/gpu/windows/d3d11_h265_accelerator.cc

void D3D11H265Accelerator::FillPicParamsWithConstants(
    DXVA_PicParams_HEVC* pic) {
  // According to DXVA spec section 2.2, this optional 1-bit flag
  // has no meaning when used for CurrPic so always configure to 0.
  pic->CurrPic.AssociatedFlag = 0;

  // num_tile_columns_minus1 and num_tile_rows_minus1 will only
  // be set if tiles are enabled. Set to 0 by default.
  pic->num_tile_columns_minus1 = 0;
  pic->num_tile_rows_minus1 = 0;

  // Host decoder may set this to 1 if sps_max_num_reorder_pics is 0,
  // but there is no requirement that NoPicReorderingFlag must be
  // derived from it. So we always set it to 0 here.
  pic->NoPicReorderingFlag = 0;

  // Must be set to 0 in absence of indication whether B slices are used
  // or not, and it does not affect the decoding process.
  pic->NoBiPredFlag = 0;

  // Shall be set to 0 and accelerators shall ignore its value.
  pic->ReservedBits1 = 0;

  // Bit field added to enable DWORD alignment and should be set to 0.
  pic->ReservedBits2 = 0;

  // Should always be set to 0.
  pic->ReservedBits3 = 0;

  // Should be set to 0 and ignored by accelerators
  pic->ReservedBits4 = 0;

  // Should always be set to 0.
  pic->ReservedBits5 = 0;

  // Should always be set to 0.
  pic->ReservedBits6 = 0;

  // Should always be set to 0.
  pic->ReservedBits7 = 0;
}
```

### **从 SPS 等位置提取 Picture Params**

下面基本都是一些枯燥的流程, 利用 H265 Parser 解析后的结果，去填充 Picture Params：

```
// media/gpu/windows/d3d11_h265_accelerator.cc

#define ARG_SEL(_1, _2, NAME, ...) NAME
#define SPS_TO_PP1(a) pic_param->a = sps->a;
#define SPS_TO_PP2(a, b) pic_param->a = sps->b;
#define SPS_TO_PP(...) ARG_SEL(__VA_ARGS__, SPS_TO_PP2, SPS_TO_PP1)(__VA_ARGS__)
void D3D11H265Accelerator::PicParamsFromSPS(DXVA_PicParams_HEVC* pic_param,
                                            const H265SPS* sps) {
  // Refer to formula 7-14 and 7-16 of HEVC spec.
  int min_cb_log2_size_y = sps->log2_min_luma_coding_block_size_minus3 + 3;
  pic_param->PicWidthInMinCbsY =
      sps->pic_width_in_luma_samples >> min_cb_log2_size_y;
  pic_param->PicHeightInMinCbsY =
      sps->pic_height_in_luma_samples >> min_cb_log2_size_y;
  // wFormatAndSequenceInfoFlags from SPS
  SPS_TO_PP(chroma_format_idc);
  SPS_TO_PP(separate_colour_plane_flag);
  SPS_TO_PP(bit_depth_luma_minus8);
  SPS_TO_PP(bit_depth_chroma_minus8);
  SPS_TO_PP(log2_max_pic_order_cnt_lsb_minus4);

  // HEVC DXVA spec does not clearly state which slot
  // in sps->sps_max_dec_pic_buffering_minus1 should
  // be used here. However section A4.1 of HEVC spec
  // requires the slot of highest tid to be used for
  // indicating the maximum DPB size if level is not
  // 8.5.
  int highest_tid = sps->sps_max_sub_layers_minus1;
  pic_param->sps_max_dec_pic_buffering_minus1 =
      sps->sps_max_dec_pic_buffering_minus1[highest_tid];

  SPS_TO_PP(log2_min_luma_coding_block_size_minus3);
  SPS_TO_PP(log2_diff_max_min_luma_coding_block_size);

  // DXVA spec names them differently with HEVC spec.
  SPS_TO_PP(log2_min_transform_block_size_minus2,
            log2_min_luma_transform_block_size_minus2);
  SPS_TO_PP(log2_diff_max_min_transform_block_size,
            log2_diff_max_min_luma_transform_block_size);

  SPS_TO_PP(max_transform_hierarchy_depth_inter);
  SPS_TO_PP(max_transform_hierarchy_depth_intra);
  SPS_TO_PP(num_short_term_ref_pic_sets);
  SPS_TO_PP(num_long_term_ref_pics_sps);

  // dwCodingParamToolFlags extracted from SPS
  SPS_TO_PP(scaling_list_enabled_flag);
  SPS_TO_PP(amp_enabled_flag);
  SPS_TO_PP(sample_adaptive_offset_enabled_flag);
  SPS_TO_PP(pcm_enabled_flag);

  // 这里发现过一个bug
  //（fix：https://chromium-review.googlesource.com/c/chromium/src/+/3538144）
  // 部分单反拍出的视频如果这里填充错误会导致花屏
  if (sps->pcm_enabled_flag) {
    SPS_TO_PP(pcm_sample_bit_depth_luma_minus1);
    SPS_TO_PP(pcm_sample_bit_depth_chroma_minus1);
    SPS_TO_PP(log2_min_pcm_luma_coding_block_size_minus3);
    SPS_TO_PP(log2_diff_max_min_pcm_luma_coding_block_size);
    SPS_TO_PP(pcm_loop_filter_disabled_flag);
  }
  SPS_TO_PP(long_term_ref_pics_present_flag);
  SPS_TO_PP(sps_temporal_mvp_enabled_flag);
  SPS_TO_PP(strong_intra_smoothing_enabled_flag);
}
#undef SPS_TO_PP
#undef SPS_TO_PP2
#undef SPS_TO_PP1
```

Picture Params 还需要从 PPS，SliceHeader，以及计算好的 Ref Pic List，Picture 填充，考虑到内容过于繁琐，这里暂时省略，整体思路可以概括为`参数拼装`。

### **处理分辨率，色彩深度的突变**

现实中的实际视频，尤其是在 WebRTC 场景产生的视频，可能存在分辨率或者色彩深度突变的情况，因此，在实际实现 Decoder 的过程中，处理这种情况至关重要，如果处理不好，轻则会导致视频花屏、绿屏，重则会导致 D3D11 device context lost，并最终导致 GPU 进程崩溃。

```
// media/gpu/h265_decoder.cc

switch (curr_nalu_->nal_unit_type) {
      // 对每个视频帧解码
      case H265NALU::BLA_W_LP:  // fallthrough
      case H265NALU::BLA_W_RADL:
      case H265NALU::BLA_N_LP:
      case H265NALU::IDR_W_RADL:
      case H265NALU::IDR_N_LP:
      case H265NALU::TRAIL_N:
      case H265NALU::TRAIL_R:
      case H265NALU::TSA_N:
      case H265NALU::TSA_R:
      case H265NALU::STSA_N:
      case H265NALU::STSA_R:
      case H265NALU::RADL_N:
      case H265NALU::RADL_R:
      case H265NALU::RASL_N:
      case H265NALU::RASL_R:
      case H265NALU::CRA_NUT:
        if (!curr_slice_hdr_) {
          curr_slice_hdr_.reset(new H265SliceHeader());
          // 对所有视频帧，解析SliceHeader
          par_res = parser_.ParseSliceHeader(*curr_nalu_, curr_slice_hdr_.get(),
                                             last_slice_hdr_.get());
          ....

          state_ = kTryPreprocessCurrentSlice;
          // 这里负责处理检测是否为irap帧 (之前因为使用sps的id去判断是否发生变化，
          // 导致了部分视频崩溃），因此使用irap作为判断条件，如果是irap
          // 则去检查是否该帧引用的分辨率，色彩空间等参数是否发生变化
          if (curr_slice_hdr_->irap_pic) {
            bool need_new_buffers = false;
            if (!ProcessPPS(curr_slice_hdr_->slice_pic_parameter_set_id,
                            &need_new_buffers)) {
              SET_ERROR_AND_RETURN();
            }

            // 如果发生变化，则need_new_buffers赋值true，返回kConfigChange，
            // 并重新创建D3D11Decoder
            if (need_new_buffers) {
              curr_pic_ = nullptr;
              return kConfigChange;
            }
          }
        }

        ....

// 这里是实际的检测逻辑，profile，色深，分辨率若发生变化，则need_new_buffers改为true
bool H265Decoder::ProcessPPS(int pps_id, bool* need_new_buffers) {
  DVLOG(4) << "Processing PPS id:" << pps_id;

  const H265PPS* pps = parser_.GetPPS(pps_id);
  // Slice header parsing already verified this should exist.
  DCHECK(pps);

  const H265SPS* sps = parser_.GetSPS(pps->pps_seq_parameter_set_id);
  // PPS parsing already verified this should exist.
  DCHECK(sps);

  if (need_new_buffers)
    *need_new_buffers = false;

  gfx::Size new_pic_size = sps->GetCodedSize();
  gfx::Rect new_visible_rect = sps->GetVisibleRect();
  if (visible_rect_ != new_visible_rect) {
    DVLOG(2) << "New visible rect: " << new_visible_rect.ToString();
    visible_rect_ = new_visible_rect;
  }
  if (!IsYUV420Sequence(*sps)) {
    DVLOG(1) << "Only YUV 4:2:0 is supported";
    return false;
  }

  // Equation 7-8
  max_pic_order_cnt_lsb_ =
      std::pow(2, sps->log2_max_pic_order_cnt_lsb_minus4 + 4);

  VideoCodecProfile new_profile = H265Parser::ProfileIDCToVideoCodecProfile(
      sps->profile_tier_level.general_profile_idc);
  uint8_t new_bit_depth = 0;
  if (!ParseBitDepth(*sps, new_bit_depth))
    return false;
  if (!IsValidBitDepth(new_bit_depth, new_profile)) {
    DVLOG(1) << "Invalid bit depth=" << base::strict_cast<int>(new_bit_depth)
             << ", profile=" << GetProfileName(new_profile);
    return false;
  }
  if (pic_size_ != new_pic_size || dpb_.max_num_pics() != sps->max_dpb_size ||
      profile_ != new_profile || bit_depth_ != new_bit_depth) {
    if (!Flush())
      return false;
    DVLOG(1) << "Codec profile: " << GetProfileName(new_profile)
             << ", level(x30): " << sps->profile_tier_level.general_level_idc
             << ", DPB size: " << sps->max_dpb_size
             << ", Picture size: " << new_pic_size.ToString()
             << ", bit_depth: " << base::strict_cast<int>(new_bit_depth);
    profile_ = new_profile;
    bit_depth_ = new_bit_depth;
    pic_size_ = new_pic_size;
    dpb_.set_max_num_pics(sps->max_dpb_size);
    if (need_new_buffers)
      *need_new_buffers = true;
  }

  return true;
}
```

可以看到在返回 kConfigChange 后，实际上是重新创建了一个新的 D3D11Decoder，这个过程用户在前端完全无感知，创建速度非常快，整体视频播放不会感受到一丝卡顿，是连贯的，相比 VLC 处理的体验更好。

```
// media/gpu/windows/d3d11_video_decoder.cc

...
} else if (result == media::AcceleratedVideoDecoder::kConfigChange) {
    // 忽略首次变化的情况
    const auto new_bit_depth = accelerated_video_decoder_->GetBitDepth();
    const auto new_profile = accelerated_video_decoder_->GetProfile();
    const auto new_coded_size = accelerated_video_decoder_->GetPicSize();
    if (new_profile == config_.profile() &&
        new_coded_size == config_.coded_size() &&
        new_bit_depth == bit_depth_ && !picture_buffers_.size()) {
        continue;
    }

    // Update the config.
    MEDIA_LOG(INFO, media_log_)
        << "D3D11VideoDecoder config change: profile: "
        << static_cast<int>(new_profile) << " coded_size: ("
        << new_coded_size.width() << ", " << new_coded_size.height() << ")";
    profile_ = new_profile;
    config_.set_profile(profile_);
    config_.set_coded_size(new_coded_size);

    // 如果发生变化，则重新创建D3D11Decoder
    auto video_decoder_or_error = CreateD3D11Decoder();
    if (video_decoder_or_error.has_error()) {
        return NotifyError(std::move(video_decoder_or_error).error());
    }
    DCHECK(set_accelerator_decoder_cb_);
    set_accelerator_decoder_cb_.Run(
        std::move(video_decoder_or_error).value());
    picture_buffers_.clear();
} else if (result == media::AcceleratedVideoDecoder::kTryAgain) {
...
```

### **处理非 HEVC Main / Main10 的其他 Profile**

根据 HEVC Spec 2021，HEVC 一共存在 11 种 Profile，具体视频使用哪种 Profile 可由 SPS 中的`general_profile_idc`的值来判断，由于之前 Chromium 没有定义其他 8 种 Profile，导致其他 Profile 会被当作 Main Profile，并使 FFMpegVideoDecoder 的兜底逻辑失败，因此在这个 CL （[https://chromium-review.googlesource.com/c/chromium/src/+/3552293](https://link.zhihu.com/?target=https%3A//chromium-review.googlesource.com/c/chromium/src/%2B/3552293)）中将其他几种 Profile 添加解决了这个问题。

HEVC 的 11 种 Profile：

```
// media/mojo/mojom/stable/stable_video_decoder_types.mojom

// Maps to |media.mojom.VideoCodecProfile|.
[Stable, Extensible]
enum VideoCodecProfile {
  // Keep the values in this enum unique, as they imply format (h.264 vs. VP8,
  // for example), and keep the values for a particular format grouped
  // together for clarity.
  // Next version: 2
  // Next value: 37
  // 跳过
  ...,
  kHEVCProfileMin = 16,
  // 下面的三种Profile是HEVC Version1定义的三种基础Profile
  // HEVC Main Profile，最高支持8Bit，YUV420
  // 苹果老款不支持杜比世界的iPhone拍的都是这种
  kHEVCProfileMain = kHEVCProfileMin,
  // HEVC Main10 Profile, 支持最高10bit，YUV420
  // 苹果新款支持杜比视界（HLG8.4）的iPhone拍的HDR视频都是这种
  kHEVCProfileMain10 = 17,
  // 一个传说中的Profile，并没有见过一个视频是这个Profile
  // 使用`ffmpeg -i bear-1280x720.mp4 -vcodec hevc -profile:v mainstillpicture bear-1280x720-hevc-msp.mp4`
  // 转码后也无法获得该类型profile
  kHEVCProfileMainStillPicture = 18,
  kHEVCProfileMax = kHEVCProfileMainStillPicture,
  // 跳过
  ...,
  // 这里是新增的8种Profile
  [MinVersion=1] kHEVCProfileExtMin = 29,
  // Format range extension（HEVC扩展格式，HEVC Version2新增）
  // 佳能，索尼，尼康等新机型拍出来的422 10bit HEVC都是这种 最高支持16bit，YUV444
  // 在macOS M1 Mac机型10bit及以下可硬解
  // 在Windows Intel机型可硬解，因为Intel自己扩展了DXVA规范实现了这部分能力
  //（VLC支持，但目前Chromium还没支持）
  [MinVersion=1] kHEVCProfileRext = kHEVCProfileExtMin,
  // 后面的这7种都是存在于Spec上的Profile，俺也没见找到过样片，只知道他们都不能硬解
  [MinVersion=1] kHEVCProfileHighThroughput = 30,
  [MinVersion=1] kHEVCProfileMultiviewMain = 31,
  [MinVersion=1] kHEVCProfileScalableMain = 32,
  [MinVersion=1] kHEVCProfile3dMain = 33,
  [MinVersion=1] kHEVCProfileScreenExtended = 34,
  [MinVersion=1] kHEVCProfileScalableRext = 35,
  [MinVersion=1] kHEVCProfileHighThroughputScreenExtended = 36,
  [MinVersion=1] kHEVCProfileExtMax = kHEVCProfileHighThroughputScreenExtended,
};
```

Profile 的赋值逻辑：

```
// media/ffmpeg/ffmpeg_common.cc

int hevc_profile = -1;
// 这里由于chrome并没有引入ffmpeg hevcps相关代码，因此需要自己解析一遍
// 拿到HEVCDecoderConfigurationRecord，并获取general_profile_idc
if (codec_context->extradata && codec_context->extradata_size) {
  mp4::HEVCDecoderConfigurationRecord hevc_config;
  if (hevc_config.Parse(codec_context->extradata,
                        codec_context->extradata_size)) {
    hevc_profile = hevc_config.general_profile_idc;
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
    if (!color_space.IsSpecified()) {
      // 由于没有引入hevc_ps相关代码，在无法从容器获取色彩空间的情况
      // 手动从SPS提取色彩空间
      color_space = hevc_config.GetColorSpace();
    }
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  }
}
// The values of general_profile_idc are taken from the HEVC standard, see
// the latest https://www.itu.int/rec/T-REC-H.265/en
switch (hevc_profile) {
    case 1:
        profile = HEVCPROFILE_MAIN;
        break;
    case 2:
        profile = HEVCPROFILE_MAIN10;
        break;
    case 3:
        profile = HEVCPROFILE_MAIN_STILL_PICTURE;
        break;
    case 4:
        profile = HEVCPROFILE_REXT;
        break;
    case 5:
        profile = HEVCPROFILE_HIGH_THROUGHPUT;
        break;
    case 6:
        profile = HEVCPROFILE_MULTIVIEW_MAIN;
        break;
    case 7:
        profile = HEVCPROFILE_SCALABLE_MAIN;
        break;
    case 8:
        profile = HEVCPROFILE_3D_MAIN;
        break;
    case 9:
        profile = HEVCPROFILE_SCREEN_EXTENDED;
        break;
    case 10:
        profile = HEVCPROFILE_SCALABLE_REXT;
        break;
    case 11:
        profile = HEVCPROFILE_HIGH_THROUGHPUT_SCREEN_EXTENDED;
        break;
    default:
        // Always assign a default if all heuristics fail.
        profile = HEVCPROFILE_MAIN;
        break;
}
```

当 Profile 能力补齐后，就可以支持将硬解不支持的 Profile 自动 fallback 到 FFMpegVideoDecoder 软解的能力了，这样可以确保我们目前可见的所有 HEVC Profile 都可以正常播放（能走硬解走硬解，否则走软解）。

### **处理色彩空间提取逻辑**

之前版本的 Chromium 提取色彩空间的逻辑要么是利用 ffmpeg 的 avcodec\_parameters\_to\_context 获取，最终利用 ffmpeg 解析 mov 或者 mp4 container 的逻辑获取，要么在 demux 阶段提取 FOURCC\_COLR Box 获取，这样做对于标准的 mov, mp4 视频并没有什么问题，然而很多编码器在实现时并没有将色彩空间信息写入容器，导致 Chromium 的之前的逻辑无法正确提取到 HEVC 视频的色彩空间。

因此我们需要利用解析好的 `HEVCDecoderConfigurationRecord`，在 demux 阶段对 SPS 进行解析，并提取其 `sps->vui_parameters->colour_primaries` , `sps->vui_parameters->transfer_characteristics` , `sps->vui_parameters->matrix_coeffs` , 以及 `sps->vui_parameters->video_full_range_flag`以生成 `VideoColorSpace`。

```
// media/formats/mp4/hevc.cc

#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
VideoColorSpace HEVCDecoderConfigurationRecord::GetColorSpace() {
  // 利用HEVCDecoderConfigurationRecord的HVCCNALArray，解析SPS
  if (!arrays.size()) {
    DVLOG(1) << "HVCCNALArray not found, fallback to default colorspace";
    return VideoColorSpace();
  }

  std::vector<uint8_t> buffer;

  for (size_t j = 0; j < arrays.size(); j++) {
    for (size_t i = 0; i < arrays[j].units.size(); ++i) {
      buffer.insert(buffer.end(), kAnnexBStartCode,
                    kAnnexBStartCode + kAnnexBStartCodeSize);
      buffer.insert(buffer.end(), arrays[j].units[i].begin(),
                    arrays[j].units[i].end());
    }
  }

  H265Parser parser;
  H265NALU nalu;
  parser.SetStream(buffer.data(), buffer.size());
  while (true) {
    H265Parser::Result result = parser.AdvanceToNextNALU(&nalu);

    if (result != H265Parser::kOk)
      return VideoColorSpace();

    switch (nalu.nal_unit_type) {
      case H265NALU::SPS_NUT: {
        int sps_id = -1;
        result = parser.ParseSPS(&sps_id);
        if (result != H265Parser::kOk) {
          DVLOG(1) << "Could not parse SPS, fallback to default colorspace";
          return VideoColorSpace();
        }

        const H265SPS* sps = parser.GetSPS(sps_id);
        DCHECK(sps);
        return sps->GetColorSpace();
      }
      default:
        break;
    }
  }
}
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)

// media/formats/mp4/box_definitions.cc

case FOURCC_HEV1:
case FOURCC_HVC1: {
  DVLOG(2) << __func__ << " parsing HEVCDecoderConfigurationRecord (hvcC)";
  std::unique_ptr<HEVCDecoderConfigurationRecord> hevcConfig(
      new HEVCDecoderConfigurationRecord());
  RCHECK(reader->ReadChild(hevcConfig.get()));
  video_codec = VideoCodec::kHEVC;
  // 这里调用，获取一下色彩空间
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  video_color_space = hevcConfig->GetColorSpace();
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  video_codec_profile = hevcConfig->GetVideoProfile();
  ...
case FOURCC_DVH1:
case FOURCC_DVHE: {
  DVLOG(2) << __func__ << " reading HEVCDecoderConfigurationRecord (hvcC)";
  std::unique_ptr<HEVCDecoderConfigurationRecord> hevcConfig(
      new HEVCDecoderConfigurationRecord());
  RCHECK(reader->ReadChild(hevcConfig.get()));
  // 这里调用，获取一下色彩空间
#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  video_color_space = hevcConfig->GetColorSpace();
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
  ...
```

在与 Edge 进行对比后可以发现，Edge 只通过容器读取色彩空间，而没有 SPS 读取的逻辑，这会导致 HDR 视频无法正确 Tone Mapping，最终渲染视频异常，而在 Chromium 内则一切正常。

![](https://pic2.zhimg.com/v2-c332c8e35b6f538ecd97c039e1518e25_b.jpg)

左图为 Edge 在处理 HLG 视频时 Tone Mapping 异常的问题

### **总结**

在上述步骤后，硬解步骤已完成的差不多，目前所有的 CL 和 Fix 已合入 Chromium 104（main 分支），Windows 平台具体实现过程和代码 Diff 也可以追溯这个 Crbug（[https://bugs.chromium.org/p/chromium/issues/detail?id=1286132](https://link.zhihu.com/?target=https%3A//bugs.chromium.org/p/chromium/issues/detail%3Fid%3D1286132)）。

## **与 Edge / Safari 的对比与测试**

说了一堆技术实现可能会很枯燥，下面来到最有趣的环节：“与竞品对比”。为了公平起见，使用原生 HTML + 原生 Video 标签方式，排除一切外界干扰完成一个基础的测试页面，并收集了 28 个不同 Profile、HDR / 非 HDR、不同位深的测试 Case（测试素材来自网络：[https://lf3-cdn-tos.bytegoofy.com/obj/tcs-client/resources/video\_demo\_hevc.html](https://link.zhihu.com/?target=https%3A//lf3-cdn-tos.bytegoofy.com/obj/tcs-client/resources/video_demo_hevc.html)），下面开始测试：

### **HDR 测试**

我们首先进行 HDR 能力测试，测试选择了多个 PQ、HLG Transfer 的 HEVC 视频。

### **PQ SDR 显示器测试**

毕竟不是所有人都使用 HDR 显示器，甚至可以说 99.99% 的用户仍在使用 SDR 显示器，因此 HDR 视频是否能在普通 SDR 显示器正确显示，是非常重要的，将 HDR 视频转换为 SDR 视频的过程一般被称作做 `Tone Mapping`，因此下述测试主要测试浏览器是否支持 `Tone Mapping`。

![](https://pic1.zhimg.com/v2-17a12f08b4469b6e61c72fc0c0645d7c_b.jpg)

![](https://pic3.zhimg.com/v2-d1d954efb11544c4a0bee65fb359efea_b.jpg)

左图：Edge 102 Windows，右图: Chromium 104 Windows

可以看到在 Windows 平台，Edge 在处理 PQ 曲线的 HDR 视频时存在 `Tone Mapping` 异常的问题，而 Chromium 可以正常 `Tone Mapping`，这一轮 Chromium 胜。

![](https://pic3.zhimg.com/v2-2475c9535f0eba810d7a5ad040ac13be_b.jpg)

左图：Safari 15.3 macOS，右图: Chromium 104 macOS

在 macOS 平台，Safari 的对 PQ HDR 视频的 `Tone Mapping` 处理的很棒，Chromium 104 也同样不错，二者效果完全相同，而且由于 macOS 支持 EDR（简介：[https://developer.apple.com/videos/play/wwdc2021/10161/](https://link.zhihu.com/?target=https%3A//developer.apple.com/videos/play/wwdc2021/10161/)），即使使用 SDR 显示器，其显示效果相比 Windows 平台更佳（Mac 会适当扩展高光），两款浏览器这一轮打平。

![](https://pic4.zhimg.com/v2-036d84d9b740591604cc61dbb189d133_b.jpg)

EDR 简介，图片来自 Apple

### **PQ HDR 显示器测试**

接着我们将显示器调为 HDR 模式，并开启操作系统的 HDR 输出。

![](https://pic1.zhimg.com/v2-a8e64fb0444be3a8be86389797ef3228_b.jpg)

![](https://pic1.zhimg.com/v2-fd10f3b844c4f17f2bd8be26ab57a198_b.jpg)

左图：Edge 102 Windows，右图: Chromium 104 Windows

可以看到，如果视频是 mov，一般都会在封装容器会写入色彩空间，此时二者区别不大，都可以较好在 HDR 显示器以 HDR 效果显示 PQ HDR 视频内容，但如果视频是 mp4 封装，则由于 mp4 一般不写入色彩空间到封装容器，Edge 存在 PQ 视频显示异常的问题，这一轮 Chromium 胜。

接着我们测试 macOS，在 macOS 播放 HDR 视频，无需任何设置，因为其支持 EDR 功能，我们选择支持 HDR 的 XDR 显示器 Mac (新款 M1 Pro / Max Macbook Pro）进行测试，正确显示 HDR 视频无需任何设置。（注：如果需要为外置显示器强制启用 HDR，需要使用支持的显示器并在 设置-显示器 面板开启“高动态范围”选项）

![](https://pic4.zhimg.com/v2-0f4419b2f56bc4fd0f524dd93a7f7c8b_b.jpg)

![](https://pic4.zhimg.com/v2-e3a145002a06083661ef6b872ea3e8a3_b.jpg)

左图：Safari 15.3 macOS，右图: Chromium 104 macOS

可以看到在 macOS 平台，Safari 对 PQ HDR 视频的 Tone Mapping 处理的很棒，Chromium 104 也同样不错，因此二者显示效果完全相同（由于 macOS 是默认 EDR，无需额外设置），这一轮打平。

### **HLG SDR 显示器测试**

![](https://pic2.zhimg.com/v2-c332c8e35b6f538ecd97c039e1518e25_b.jpg)

左图：Edge 102 Windows，右图: Chromium 104 Windows

可以看到 Edge 在处理 HLG 视频时，如果色彩空间没有写入封装容器，则无法正确读取色彩空间，导致存在偏色问题，而 Chromium 目前会通过容器读取色彩空间，如果不存在，则继续从 SPS 读取色彩空间，这可以保证所有 HLG 视频均可正确 Tone Mapping，这一轮 Chromium 胜。

![](https://pic2.zhimg.com/v2-8eac9f714268b1b44faeffed276550e9_b.jpg)

左图：Safari 15.3 macOS，右图: Chromium 104 macOS

在 macOS 平台，Safari 的对 PQ HDR 视频的 Tone Mapping 处理的很棒，Chromium 104 也同样不错，二者效果完全相同，而且由于 macOS 支持 EDR（简介），即使使用 SDR 显示器，其处理效果相比 Windows 平台更佳（Mac 不会强制压高光），两款浏览器这一轮打平。

### **HLG HDR 显示器测试**

![](https://pic1.zhimg.com/v2-01d93ab7f75f856c6698df39d6eaeafc_b.jpg)

![](https://pic4.zhimg.com/v2-5c0b0f5f5b8346a465b41a9ddbd476af_b.jpg)

容器未写入色彩空间情况，左图：Edge 102 Windows，右图: Chromium 104 Windows

![](https://pic3.zhimg.com/v2-8abb310f81e73a3e89ae7a5be3148cda_b.jpg)

容器写入色彩空间情况，左图：Edge 102 Windows，右图: Chromium 104 Windows

可以看到 Edge 在显示 HLG 视频时并未激活 HDR 输出，而 Chromium 可完美 HDR 输出（肉眼效果和截图不一致，截图比较亮，肉眼显示是正常的），同时，即使视频容器有写入色彩空间，Edge 处理后的视频存在过曝问题，这一轮 Chromium 完胜。

![](https://pic1.zhimg.com/v2-5dcc9524cf77053eec3e40784552e298_b.jpg)

左图：Safari 15.3 macOS，右图: Chromium 104 macOS

在 macOS 平台，Safari 完全支持 HLG HDR 视频，Chromium 104 也不错，二者效果完全相同，均可良好支持 HLG HDR 在 HDR 显示器以 HDR 格式完美显示，这一轮打平。

### **小结**

根据上述测试结果，总结如下：

![](https://pic3.zhimg.com/v2-c2ce0378400fff47e2c71df903421292_b.jpg)

在 macOS 平台，Safari 和 Chromium 的 HDR 表现均良好，且支持 EDR。

在 Windows 平台，如果你想观看 HDR 内容，没有别的选择，Chromium 是唯一完全支持 HDR 的浏览器。

### **Rext Profile 测试**

在 Windows 平台，Edge 与 Windows 均不支持 HEVC Rext，这是因为 DXVA 规范并没有制定除 Main / Main10 以外的 Profile（尽管 Intel 和 NVIDIA 后期自己实现了 422 444 Rext 的硬解，但这不在规范里）。

![](https://pic3.zhimg.com/v2-0b5a375164fb91971e473f7e8714a676_b.jpg)

左图：Safari 15.3 macOS + Intel Mac，右图: Chromium 104 macOS

![](https://pic2.zhimg.com/v2-b5fc53a8c9f0ca28f29dd0138bf12dfd_b.jpg)

左图：Safari 15.3 macOS + M1 Mac，右图: Chromium 104 macOS

如图所示，在 macOS 平台，Safari 不完全支持 HEVC Rext（比如 Intel 就不支持，M1 芯片的 Mac 支持一部分），而 Chromium 104 支持硬 / 软解 HEVC Rext (Apple Silicon 芯片支持 10bit Rext 硬解，Intel 芯片 Mac 支持 Rext 软解)，Chromium 胜在兼容性。

### **8K 支持测试**

在 Windows 平台，结论是：这一轮 Chromium 和 Edge 打平，目前能找到的 8K 视频二者都可正常播放，因此这里暂时不放截图了。

![](https://pic1.zhimg.com/v2-7b3f7b078cd44544031ed3f6e88b0000_b.jpg)

左图：Safari 15.3 macOS，右图: Chromium 104 macOS

在 macOS 平台，诚然，Safari 确实支持 8K，这一点可以在 B 站的 8K HEVC 视频上验证，但是由于其“挑格式”的小毛病，测试页面“为数不多”的几个 8K 测试的视频团灭，因此这一轮 Chromium 胜。

### **格式兼容性测试**

在 Windows 平台，结论是：这一轮 Chromium 和 Edge 打平，目前我能找到的所有 Main / Main10 Profile 的 HEVC 视频均可在二者正常播放，因此这里暂时不放截图了。

![](https://pic4.zhimg.com/v2-143f31d2f6eee86007028b856c73936b_b.jpg)

![](https://pic4.zhimg.com/v2-349bd520977cc295be387e41351ffcbb_b.jpg)

左图：Safari 15.3 macOS，右图: Chromium 104 macOS

在 macOS 平台，Safari 彻底输了，40 个测试视频约一半无法正常播放，这一波 Chromium 完胜。

### **实际场景测试**

BiliBili 是目前官方支持 HEVC 的网站，通过使用 User Agent 修改插件，并模拟成 macOS 的 Safari，我们可以使 B 站优先使用 HEVC 播放视频，且最大支持 8K。

![](https://pic1.zhimg.com/v2-b7efae115a8bc87f1e4b15b467ec6654_b.jpg)

![](https://pic4.zhimg.com/v2-e66efb5a2ac1bf7334525935ccf1d047_b.jpg)

如下图所示，开启硬解后的 Chromium 可以流畅播放 8K 60P 的 HEVC 视频：

![](https://pic2.zhimg.com/v2-4241742a7f07179a4ff0213fe7f29a65_b.jpg)

打开`chrome://media-internals`可以验证，视频是分辨率确实是`7680*4320`的 HEVC。

![](https://pic3.zhimg.com/v2-76d217926f98edce698107e9af586142_b.jpg)

**性能如何？**

为此我找了一台使用 HD620 GPU 的 Lenovo Thinkpad T14，尝试在 B 站播放 8K 60P 的视频：

![](https://pic3.zhimg.com/v2-82c30da3bcbc80cda056936777bb3bb2_b.jpg)

Chromium 104，HD620 在播放 8K 60P 视频时的 GPU Decode 占用率达到了 100%

可以看到 HD620 核显拼一拼还是可以播 8K 60P 视频的（虽然偶尔有一点掉帧），同时也可以看到在播放 8K 视频时的系统 CPU 占用率只有 16%，硬解带来的性能收益非常显著。

通过使用 User Agent 修改插件，并模拟 Edge 18.19041，我们可以解锁 B 站 的 HDR 模式，最大分辨率为 4K。我们选择了一个支持 HDR 的视频，并对比 Edge 和 Chromium。

![](https://pic4.zhimg.com/v2-e77661b5da4938ea214958d4b035d877_b.jpg)

![](https://pic4.zhimg.com/v2-c1acb8cadd342bcb8a88f757b846fe33_b.jpg)

![](https://pic3.zhimg.com/v2-8ec13f48df35d58b310f30e27d5bac82_b.jpg)

左图：Edge 102 Windows，右图: Chromium 104 Windows

可以发现在 Windows 平台，Chromium 是唯一良好支持 B 站 HDR10 的浏览器，而 Edge 无法正常显示。

![](https://pic1.zhimg.com/v2-71c2a444b11d348dc376ffb06ee48814_b.jpg)

左图：Safari 15.3 macOS，右图: Chromium 104 macOS

可以发现在 macOS 平台，Safari 和 Chromium 均可良好支持 B 站的 HDR10，在 HDR 支持上，二者无区别。

### **总结**

经过上述测评想必大家应该可以看到，在 2022 年的今天，终于，Chromium 也可以完整支持硬解 HEVC 了，且相比 Safari 和 Edge，在 HDR 支持，格式兼容性，性能，平台支持四个方面均表现良好，甚至小幅超越。

## **运行并验证**

### **Chrome Canary**

访问这里，可直接下载 Chrome Canary （[https://www.google.com/chrome/canary/](https://link.zhihu.com/?target=https%3A//www.google.com/chrome/canary/)）进行测试（启动参数 --enable-features=PlatformHEVCDecoderSupport ）。

### **Mac**

在 macOS，如果需要开启 HEVC 硬解功能，一种方式是以命令行的方式打开：

```
// 通过--args 传入Switch参数
/Application/Google\ Chrome\ Canary/MacOS/Google\ Chrome\ Canary --args --enable-features=PlatformHEVCDecoderSupport
```

如果不喜欢命令行，也可以通过 `Automator` 建立启动自动操作的方式打开。

![](https://pic4.zhimg.com/v2-c8828a9c100329cfc8418fc8de8ac2ef_b.jpg)

如何验证是否生效？打开`chrome://gpu`, 如出现下图红圈所示的字样表示成功。

![](https://pic3.zhimg.com/v2-42a0dffc0850e51b9b2fe789fda20d56_b.jpg)

找到一个 HEVC 的视频并播放，打开 `chrome://media-internals` 页面，如视频解码出现`VDAVideoDecoder` 字样，表示硬解成功：

![](https://pic1.zhimg.com/v2-916afd738480ff0f3cef32fab604fc44_b.jpg)

打开活动监视器，搜索 `VTDecoderXPCService` , 播放视频时，观察到进程 CPU 占用率上涨也可说明硬解成功：

![](https://pic3.zhimg.com/v2-751ea0607c9c549aef011410ce2210d6_b.jpg)

### **Windows**

Windows 的 Chrome，在桌面快捷方式传入启动参数即可。

```
"C:\Users\Admin\AppData\Local\Google\Chrome SxS\Application\chrome.exe" --enable-features="PlatformHEVCDecoderSupport"
```

![](https://pic1.zhimg.com/v2-ee000362765de287cf69be510dc69968_b.jpg)

如何验证是否生效？打开 `chrome://gpu`, 如出现下图红圈所示的字样表示成功。

![](https://pic1.zhimg.com/v2-5bd0d83ded8a63dc32aca0088fafca84_b.jpg)

找到一个 HEVC 视频并播放，打开 `chrome://media-internals` 页面，如视频解码出现 `D3D11VideoDecoder` 字样，表示硬解成功：

![](https://pic4.zhimg.com/v2-55c666ae25e9da6d89c58c98dd8d2cb7_b.jpg)

打开 Windows 任务管理器 - 性能 - GPU - Video Decode 区域，观察播放时的使用率是否上涨，如果上涨亦可说明硬解成功，如果占比为 0% 说明硬解失败。

![](https://pic4.zhimg.com/v2-eb3e4f676c25d6a95edaacf25991390b_b.jpg)

在 AMD GPU 上，显示为 “Video Codec”。

![](https://pic3.zhimg.com/v2-2ca16d63c8f1fe6149291d3ea5c14172_b.jpg)

### **Android**

方式类似，传启动参数即可支持，这里由于本人不使用 Android 设备，暂未贴出具体步骤。

### **ChromeOS**

最新测试版本已原生集成于 OS，无需传启动参数。

### **预编译版本**

如果你觉得传参很麻烦，也可访问这里，下载无需启动参数的预编译版本（[https://github.com/StaZhu/enable-chromium-hevc-hardware-decoding/releases](https://link.zhihu.com/?target=https%3A//github.com/StaZhu/enable-chromium-hevc-hardware-decoding/releases)） 进行测试（需要 Google 服务只能用 Chrome Canary）。

### **稳定版本**

HEVC 硬解功能目前正在实验中，最早可能在 Chrome 105 稳定版发布。

### **集成到 Electron**

你可能希望把这个 Feature 编译到 CEF、Electron 等 Framework 内。如果是 Electron 20 正式版 (基于 Chromium 104，目前还是 beta 版本)，则已集成好 Mac, Windows 平台的 HEVC 硬解功能，在启动时执行 `app.commandLine.appendSwitch('enable-features', 'PlatformHEVCDecoderSupport'`) 即可启用硬解。如果是 Electron 20 以下版本，需要自己手动 CV 大法集成。

## **最后**

可分别 Trace 这两个 Issue（**Windows：**[https://bugs.chromium.org/p/chromium/issues/detail?id=1286132](https://link.zhihu.com/?target=https%3A//bugs.chromium.org/p/chromium/issues/detail%3Fid%3D1286132)，**macOS：**[https://bugs.chromium.org/p/chromium/issues/detail?id=1300444](https://link.zhihu.com/?target=https%3A//bugs.chromium.org/p/chromium/issues/detail%3Fid%3D1300444)）追踪后续进度。

如果有 HEVC 视频播放需求，不妨可以尝试一下 Chrome Canary 版本或预编译版本，如遇到 Bug，请在 [http://crbug.com](https://link.zhihu.com/?target=http%3A//crbug.com) 提交反馈。