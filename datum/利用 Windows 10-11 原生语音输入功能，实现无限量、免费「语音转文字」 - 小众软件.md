Windows 10、11 系统自带了一个语音输入功能，可以实现将麦克风的语音输入，实时转换为文字。那么利用系统声音设备的输入输出切换，就可以实现一个**系统原生**的、**免费**、**无限量**的语音转文字功能。@[Appinn](https://www.appinn.com/speech-to-text-windows10-and-11/)

![利用 Windows 10/11 原生语音输入功能，实现无限量、免费「语音转文字」](https://img3.appinn.net/static/wp-content/uploads/2022/10/Speech-to-Text.jpg "利用 Windows 10/11 原生语音输入功能，实现无限量、免费「语音转文字」 1")

感谢 @张同学 推荐的 [Kevin Stratvert](https://www.youtube.com/watch?v=1DsrniDGOJQ) 视频。

语音转文字是很多同学的刚需，可是青小蛙却从来不知道 Windows 里自带了这个功能。

## 使用语音键入来说话，而不是在电脑上键入

这是 Windows 10 与 Windows 11 自带的功能，原本是用来在电脑上进行语音输入的：

![利用 Windows 10/11 原生语音输入功能，实现无限量、免费「语音转文字」 1](https://img3.appinn.net/static/wp-content/uploads/2022/10/77cd8bd7-9620-5b88-fd89-ba68f373f943.jpg "利用 Windows 10/11 原生语音输入功能，实现无限量、免费「语音转文字」 2")

包括执行一些命令，比如「逐字逐句 ：逐字 移动：转到最后」；「移到段落末尾、删除 字词」等，有详细的[介绍](https://support.microsoft.com/zh-cn/windows/%E4%BD%BF%E7%94%A8%E8%AF%AD%E9%9F%B3%E9%94%AE%E5%85%A5%E6%9D%A5%E8%AF%B4%E8%AF%9D-%E8%80%8C%E4%B8%8D%E6%98%AF%E5%9C%A8%E7%94%B5%E8%84%91%E4%B8%8A%E9%94%AE%E5%85%A5-fec94565-c4bd-329d-e59a-af033fa5689f#WindowsVersion=Windows_10)。

## VB-CABLE 虚拟音频设备

实现[语音转文字](https://www.appinn.com/tag/%E8%AF%AD%E9%9F%B3%E8%BD%AC%E6%96%87%E5%AD%97/)的原理很简单，将电脑的音频输出提供给音频输入，就可以让 Windows 的**语音键入**功能转文字了。

[VB-CABLE](https://vb-audio.com/Cable/) 就是这样一款虚拟驱动程序，所有进入 CABLE 输入的音频都被简单地转发到 CABLE 输出。

在使用**管理员权限**安装 VB-CABLE 之后，进入系统声音设置，将输出输入设备都选择为 CABLE 即可：

![利用 Windows 10/11 原生语音输入功能，实现无限量、免费「语音转文字」 2](https://img3.appinn.net/static/wp-content/uploads/2022/10/Screen-Appinn2022-10-16-15.46.44.jpg "利用 Windows 10/11 原生语音输入功能，实现无限量、免费「语音转文字」 3")

## 语音转文字

最后，就可以在播放音频的同时，实现语音转文字了。

1.  开始播放音频文件、视频网页等
2.  打开记事本
3.  按下 Win + H 激活语音转文字
4.  自动语音转文字

青小蛙在 Windows 10 下测试时不是很准确，而换到了 Windows 11 下，堪称完美。

原文：https://www.appinn.com/speech-to-text-windows10-and-11/