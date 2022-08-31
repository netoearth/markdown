## 缘起

这个故事要从上海的疫情说起。

三月份，上海爆发了疫情。全市开始封城，停工停课，所有人都不得不居家防疫。离不了家门，意味着空闲时间增多了，也正好可以利用一下，做个小项目之类的。

也正巧，前段时间研究了[eBPF攻防的技术](https://mp.weixin.qq.com/s/GAftwUGqFkIbORL9909OKA)，而技术的应用场景最广泛的就是跟踪调试了。广大研发、运维同学们面临的最大场景就是网络抓包，恰恰大量数据包都是HTTPS加密的，这就称为多数人的痛点，那么结合eBPF技术，可以很好的解决这一痛点。

说干就干，三月中旬就发了初版[eCapture：无需CA证书抓https网络明文通讯](https://mp.weixin.qq.com/s/DvTClH3JmncpkaEfnTQsRg)，随着上海封城的持续，空闲时间也多，项目也逐步完善，支持4.X至5.X的内核版本。

最近呢，复工复产了，空闲时间就少了，不过，近期搞到凌晨一两点的情况也是常态。不能半途而废不是嘛。  
![](https://image.cnxct.com/2022/06/git-log.jpg)

## 进展

这几天，有几个好消息分享给大家，有的是付出得到的回报，有的是项目的新功能。

### ebpf.io 收录

eCapture项目被 [ebpf.io](https://ebpf.io/projects/) 官网收录了 ，在「Emerging新兴项目」中，暂排第一。是按照Star数量来的，项目复杂度、技术深度不如其他项目，但实用性比较大，深受大家喜欢。

![](https://image.cnxct.com/2022/06/screenshot-ebpf.io-2022.06.22-00_23_51.png)

### eCapture for Android

![](https://image.cnxct.com/2022/06/Android_logo_2019.png)

eCapture也支持了[Android的GKI](https://source.android.com/devices/architecture/kernel/generic-kernel-image)发型版本，你在[eCapture v0.1.10 release (Linux x86\_64/aarch64, Android GKI)](https://github.com/ehids/ecapture/releases/tag/v0.1.10)中下载试用。

![](https://image.cnxct.com/2022/06/github.com_ehids_ecapture_releases.png)

也就是说，Android 11，Android 12的版本都支持。 4.X内核的Android支持，还会远吗？

有位同学告诉我说，这会让反爬的同学受到很大挑战，eBPF的抓包，APP层很难发现，APP自身没有特征，也很难防御。

我不这么看，我觉得风控对抗上，核心点不该只在端上对抗，而应该在数据分析、好人坏人识别等方向上。地理位置、网络环境、设备型号等问题可以更加综合的考虑。

另外，可以再思考一个问题，这意味着什么？

意味着Linux上的eBPF功能都可以在Android上使用，各种网络管理、监控软件、安全保护等，甚至后门。你觉得，还有什么场景呢？

#### 演示视频

付上一个eCapture for Android的演示视频，是[TaiChi Xposed框架](https://github.com/taichi-framework/TaiChi)作者 weishu 大佬帮测试的，我没有Android手机，对Android生态也不了解，感谢weishu在这个事情上的支持。

### eCapture中文名旁观者

eCapture有了中文名，叫「旁观者」，即「当局者迷，旁观者清」，与其本身功能【旁路、观察】契合，且发音与英文有相似之处。 是不是有点信雅达呢？ 要不，您给起个名？

![](https://image.cnxct.com/2022/06/ecapture-chinese-name.png)

## 加入贡献

现在，还缺少一个官方网站，或者融合中文含义的新LOGO，如有对[eCapture](https://github.com/ehids/ecapture)开源项目感兴趣的设计师、UI前端工程师，愿意帮助项目设计新页面、新LOGO的同学，可以在微信公众号内留言，我会私聊你，谢谢。

### 红领巾

又为社会做了一点点贡献，胸前的红领巾更加鲜艳了。

![](https://image.cnxct.com/2022/06/honglingjin.jpeg)

[![知识共享许可协议](https://www.cnxct.com/attachments/88x31.png)](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)CFC4N的博客 由 [CFC4N](http://www.cnxct.com/) 创作，采用 [知识共享 署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)进行许可。基于[https://www.cnxct.com](http://www.cnxct.com/)上的作品创作。转载请注明转自：[eCapture的几个好消息，支持Android…](https://www.cnxct.com/ecapture-news-android/)