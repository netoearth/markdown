![](https://miro.medium.com/max/770/1*qFk6hlXpLn4uHySw3lqd3Q.png)

来自 [auditore\_k](https://twitter.com/auditore_k) 生成的照片

## 前言

近期 AI 领域被 ChatGPT 又带火起来了，ChatGPT 如此庞大的训练语料和参数数量是一般人难以进行实践的，我一直关注 Stable Diffusion 的发展，同时也知道无论是能力还是财力，我个人其实是不容易做出属于自己的很特别的模型，但是对这方面一直有兴趣，所以即使没有能力做训练（train），也想体验使用其他人模型创造一些内容（inference）。

这里先介绍两个网站，对于有能力探寻更深入的人会有所帮助：

> [https://huggingface.co/](https://huggingface.co/) ，Hugging Face 上有很多数据集可以用来做训练，也可以基于其他人共享的数据集进行训练，比自己找数据要方便很多，也有 auto train 功能，可以使用他们付费的服务进行训练。也可以创建 space 来快速给一个训练好的模型生成 API 接口，方便以服务的形式开放给他人使用或者让自己的其他服务调用。有了这些功能深度学习工程师便可以更专注于深度学习本身，而不用特别关注训练数据的来源或者如何部署机器进行训练、回测以及接口化。
> 
> [https://civitai.com/](https://civitai.com/) CivitAI 主要是模型分享以及社区，用户会在其他人的模型下面回复通过这个模型生成的样本，以及生成时的参数和 seed，方便我们调试其他人模型时可以参考已有的输出进行快速尝试，不然可能会一直觉得自己生成的不够好。

## ChilloutMix & LoRA

首先，我选用的基础模型是 [https://civitai.com/models/6424/chilloutmix](https://civitai.com/models/6424/chilloutmix) ，这个模型擅长于生成亚洲女性特征的图片。

其次，选择一个自己顺眼的 LORA 训练模型。LoRA（Low-Rank Adaptation of Large Language Models）粗略地讲就是利用少量的图像来对 AI 进行额外学习训练，并在一定程度上控制结果。

流程上也比较简单，就是基于 ChilloutMix 的训练结果和你自己准备的数据集来进行二次训练，目的是最终输出的内容既具有原始 ChilloutMix 的能力，又更倾向你提供的数据的特征。

这里我尝试了三个 LORA 模型：

他们本质上没有区别，但是因为二次训练集的差别，三个模型输出的内容特征分别倾向于：韩国女性、台湾女性、日本女性。当然你也可以生成中国女性、越南女性、二次元女性等等专属的模型，甚至去训练二次元图像或 NSFW 图像。每一种模型都有自己的特长，LoRA 的目的就是各具所长地进行输出而不是一个通用模型。

## 搭建环境

这里我以 Taiwan Doll Likeness 为例来叙述使用方法。

我们打开 Taiwan Doll Likeness 模型的页面，会看到页面右边有一个 ▶️ 按钮：

![](https://miro.medium.com/max/770/1*8qA5fcXt82El1ES07epDwg.png)

Taiwan Doll Likeness 页面右侧的 ▶️ 按钮

点开之后显示的是可以用哪些软件进行 inference（推断）：

![](https://miro.medium.com/max/770/1*a5ic-nlbGZA1x1Mb9y-UHg.png)

软件可用性

这里我们发现可用的只有一个，就是 Automatic 1111 Web UI (Local)。意思就是你要先配置好 Automatic 1111 Web UI 这个软件，然后它是运行在你本地的，而不是一个服务。

下面不可用的服务意思是这个模型没有提供支持下列服务的方式，可以简单认为互不兼容的意思。

> 如果你不具备基础的编程能力（例如安装软件环境或配置服务器等），你可以不用看这一章的内容了，上面图中最后一个软件是 Draw Things，这个软件在 macOS 和 iOS 上都可以下载到，你可以选择例如 [https://civitai.com/models/3950/art-and-eros-aeros-a-tribute-to-beauty](https://civitai.com/models/3950/art-and-eros-aeros-a-tribute-to-beauty) 支持 Draw Things 运行的模型，就可以不用写代码就可以通过界面交互的方式在软件里去尝试。当然 ChilloutMix 也是支持 Draw Things 的。

![](https://miro.medium.com/max/770/1*H2nZQujGCQiLRccca8YlGQ.png)

我使用 Draw Things 来生成的一张图片

## 租用服务器

如果你已经选择好要运行软件的机器了，可以跳过这一章。

如果你用的电脑没有额外的显卡，或者在用不带外接显卡的苹果设备，抑或网络条件较差，那么强烈建议你租用一台有显卡的服务器进行尝试，这样会在下载和生成的时候节约大量的时间。

> 我自用的 MacBook Pro 配有 M1 Pro 8 核心 CPU 和 16GB ram，生成图片的速度很慢，大概生成 steps 在 20 左右的低清晰度图片需要 5–10 分钟。而在一台共享 GPU 的服务器上 10s 之内可以完成。

如果你不知道如何选择服务器，这里我推荐 [Vultr](https://www.vultr.com/?ref=6981435) 的服务器，相对其他大厂商而言 Vultr 页面配置简单，价格也相对低廉，在小厂商中也是少有的提供 GPU 服务器的厂商。而且支持虚拟货币和支付宝支付，没有国际信用卡的朋友用起来也方便。

你可以使用我的邀请链接注册 Vultr，这样我们都可以获得 $10 的抵扣额度：

创建服务器时选择 Cloud GPU 并选择 NVIDIA A40 系列：

![](https://miro.medium.com/max/770/1*jrdM5rs4ssJUSTOVEgjtjg.png)

Cloud GPU + A40

选择 1/6 GPU 和 8GB 显存的版本，这种配置是我对比下来性价比最高的。记得关掉自动备份的开关，每个月可以省 $42。这样的花平均一天的成本就是 $7，我只开了几个小时，总共花费 $2 就可以进行非常完整的体验了。

![](https://miro.medium.com/max/770/1*NPtEN3g0bTpwWsyKUKdyTw.png)

记得关闭自动备份

## 配置环境

先打开 [https://github.com/AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 根据项目介绍 Automatic Installation on Linux 这一章节在服务器上进行安装，安装过程比较简单，不会出现什么意外的错误，但是记得不要用 root 用户进行安装，如果创建 sudo 权限用户请自行搜索。

安装完之后先不要运行，项目需要在启动阶段加载一个模型否则启动不了。

在 ChilloutMix 页面右侧找到下载按钮，取得下载地址后在服务器上进行下载，模型大小 3.97GB 大概几分钟时间可以下载好，下载好的模型请放置在stable-diffusion-webui 项目下的 models/Stable-diffusion/ 目录下，并确认下载好的文件名结尾为 .ckpt。

![](https://miro.medium.com/max/770/1*XF3-roYkr2b6cjLqWa8_NQ.png)

下载 ChilloutMix

同样的，前往 Taiwan Doll Likeness 页面下载模型，并放置在 models/Lora/ 目录下，也请确认扩展名为 .safetensors。

模型都下载好后，就可以运行 webui.sh 启动软件了。启动起来的服务监听在 127.0.0.1，我通过 cloudflared 进行 tunnel 转发以便在外部访问，当然你也可以通过 webui.sh 的 listen 参数让他监听在 0.0.0.0 以便可以通过 IP 地址直接访问，或者通过配置 nginx 进行转发也是可行的，具体过程此处不赘述。

当你看到如下提示时就代表启动成功了：

```
Running on local URL:  http://127.0.0.1:7860
```

## 安装插件

在浏览器中打开相应地址，切换到 Extensions 标签页面的 Install from URL，并输入 [https://github.com/civitai/sd\_civitai\_extension](https://github.com/civitai/sd_civitai_extension) 点击 Install 安装 CivitAI 提供的插件，安装成功后切换回 Installed 页面点击 Apply and restart UI 重新加载。

![](https://miro.medium.com/max/770/1*tCJYVHgpxD5eGZt0emyg2A.png)

安装 CivitAI 提供的插件

## 开始生成图片

第一步：确认左上角已经加载了 ChilloutMix 模型：

![](https://miro.medium.com/max/770/1*7xdyixzlCmQx1NZu2uxFsg.png)

第二步：在右侧找到这个按钮，点击之后左侧会展开一些新的元素

![](https://miro.medium.com/max/770/1*2HC3T8Iw3r8UeJiCHx-Qyg.png)

第三步：在左侧展开的标签中点击 Lora，并选择我们刚下载好的模型：

![](https://miro.medium.com/max/770/1*8OL-IixwXIqbq3AfDzYiUw.png)

第四步：点击之后你会看到左侧有了变化，prompt 中自动多了一行文字，代表使用 taiwanDollLikeness 这个模型：

![](https://miro.medium.com/max/770/1*iaw9rwo9fxKfHFxkhypqSA.png)

我直接输入 a girl，点击右侧的 Generate 尝试生成：

![](https://miro.medium.com/max/660/1*WtbYerfmpPoHL1cYbJVXjQ.png)

此时无论在页面上还是在命令行窗口中都可以看到生成进度和预期完成时间：

![](https://miro.medium.com/max/770/1*P1rWYKBwXm0b41zeSNUFTg.png)

![](https://miro.medium.com/max/770/1*y5S-ACrshZUxognlCFDFnQ.png)

生成完成后的图片是这样的，很容易看出来不太尽如人意，不只是不好看的问题，甚至有些不像一个人类。

![](https://miro.medium.com/max/770/1*3DRtZvsN4L7jaOKlAR6dCw.png)

## 参数调优

首先我们先调优生成的关键词，如果不知道如何选取关键词可以去官方页面的官方样例或回复中别人生成的样例中点击一张你喜欢的风格的图片，看看哪张图片是用什么关键词生成的，例如这张：

![](https://miro.medium.com/max/770/1*3lcp0XVO7dGTKsZclpQPrQ.png)

一张其他用户生成的图片和参数

此处 Prompt 代表你想要生成的元素，而 Negative prompt 代表你想要避免出现的元素，我们就先复制这张图片的参数进行生成。

> ⚠️ 注意：复制时候要注意不要全复制了，从模型文件后面开始复制，图中的生成使用的是 <lora:taiwanDollLikeness\_v10:0.66>，而我使用的是 <lora:taiwanDollLikeness\_v10:1>，注意灵活使用。

同时我们也发现该图片使用的 sampler 是 DPM++ SKE Karras，这也是比较推荐的一个效果不错的 sampler，我们也把 sampler 改成同样的。相应地，也将 CFG scale 和 Steps 参数和该图片使用的参数对齐。（Steps 越多，生成效果越好，生成速度越慢），改好的界面如下：

![](https://miro.medium.com/max/770/1*6aJe_Bf87NpBqEJK3yJD6Q.png)

> ⚠️ 注意：不建议改 Seed，seed 的意思是该次生成的「路线」，代表一定指向性的特征，我们不需要和那张样例图片的感觉一模一样，所以我选择不改 seed。

这次时间会比上次久一点，但效果好了不少：

![](https://miro.medium.com/max/563/1*KgnoSF_XGurtIFSZSrzEnw.png)

不过图片还是有点模糊，因为我们生成的图片是 512x512 大小的。生成更高清晰度的图片不要直接调节图片大小，这样会让生成的速度变慢许多，最好的方式是仍然使用 AI 让我们对模糊的图片进行自动放大：

![](https://miro.medium.com/max/770/1*yIkLOnHs8R45qdVAd4wCGg.png)

勾选 Hires. fix 选项，Hires steps 可以调节为 10–20 中的数值，并确定 Upsacle by 值为 2，这样就可以在生成出 512x512 的图片后将其放大为 1024x1024。我们再生成一次试试：

![](https://miro.medium.com/max/770/1*Mgu7-EgP1KypNGLhSueLnQ.png)

效果好多了。

关于参数方面该模型的作者已经在页面中详细介绍了如何调整参数才能获得较好的效果，大家要仔细读一下，比自己乱尝试要高效（但是自己乱尝试很好玩）：

```
Things to note if you've used my other LORAs: -1. It has more variation, and less consistent in making the same types of faces, it is easier to have more facial expressions due to this.2. Different LORA weight control would easily change the face variation.Explanation:1. This is a LORA trained for sd1.5, the trigger words are 'girl' and 'woman', or you could control the weight with LORA taiwanDollLikeness_v10 prompt.2. This LORA works extremely well with ChilloutMix and potentially any other photorealistic models, including nsfw models.3. Works okay when used together with PureErosFace but you must know how to control the weights properly.https://civitai.com/models/4514/pure-eros-face5. May also work together with 'ulzzang-6500' but weight control is a must.https://civitai.com/models/8109/ulzzang-6500-korean-doll-aesthetic6. Try to use hires fix with around 10-20 hires steps, latent upscaling (bicubic antialiased), denoising strength between 0,4-0.7, its given the best results by far.7. Higher resolution also helps when creating further subjects, hires fix helps greatly with this too.|8. Best samplers are DPM++ SDE Karras, Euler a, DPM2 a Karras and DPM++ SDECaution:1. Recommended weights for txt2img - 0.5-0.82. Recommended weights for img2img - 0.4-0.8
```

## Prompt

掌握了这些技巧大家就可以生成自己喜欢的图片了，但是可能会发现其中有一个点很困难，就是初学者如何选择 prompt。AI 从业者会形象地将 prompt 和 negative prompt 称为「咒语」并将这种生成过程称为「咒语吟唱」。你的吟唱越精确，咒语细节越多，最终生成的图片就越符合你的想象。

如果你不知道如何生成这些咒语，那么我有几种方法可能会对你有一些启发：

## Prompt Hero

访问 [https://prompthero.com/](https://prompthero.com/) 在顶部菜单选择 Stable Diffusion，在搜索框中输入 girl 关键词（或者任何你想要的关键词），在搜索结果中找到自己喜欢的风格，就可以复制这张图片的 prompt 进行尝试了。类似的网站有很多大家可以自行找一下。当然在 CivitAI 上去找 prompt 也是很好的方式。

![](https://miro.medium.com/max/770/1*LTTDOit9i8MNGj1islofIA.png)

## ChatGPT

使用 ChatGPT 进行关键词生成也是很好的方式，特别是对英文表示很好的朋友会很有帮助，请 ChatGPT 帮助你去生成你想要的 prompt：

![](https://miro.medium.com/max/770/1*a4uvTTrjueP-MYNfvnfNkQ.png)

![](https://miro.medium.com/max/770/1*5uZ6cY8fTYPtCJAMzlSLdQ.png)

## Clip Interrogator

如果你有想要模仿风格的照片，也可以把这张照片传进 clip interrogator 中，这个 AI 会帮助你提取关键词。很多人都自己实现了 clip interrogator，你也可以用这个网站直接使用：

![](https://miro.medium.com/max/770/1*DdGsq2crKRCGmtXYCpL2EA.png)

## 我生成的一些图片

![](https://miro.medium.com/max/770/1*Mgu7-EgP1KypNGLhSueLnQ.png)

![](https://miro.medium.com/max/770/1*lmAj9rHwppKc7ALW_nzEYw.png)

![](https://miro.medium.com/max/770/1*KSU4VOePJWI30kAjkEBaIg.png)

![](https://miro.medium.com/max/770/1*IUM4UmJ0KFg7PVFq-qhBag.png)

![](https://miro.medium.com/max/563/1*dZobuRi4kDz605CvbPLKnQ.png)

![](https://miro.medium.com/max/770/1*SDH-EMne1RpmIJjjGp3oJA.png)

## 手指好奇怪！

Stable Diffusion 一个非常常见的问题就是人类的关节、手指乱掉了，例如这样：

![](https://miro.medium.com/max/396/0*YWSGfQ7Gn4hqN_k1)

手指混乱

或者这样：

![](https://miro.medium.com/max/405/1*YiwiqbEQQ-DoWWWl7GP5Bw.png)

关节或人体结构混乱

目前暂时没有太好的解决方法，但是我们可以巧妙地绕过去，例如：

1.  要求图片生成时不要露出手来
2.  添加 negative 关键词，例如：extra fingers
3.  添加关键词，例如：5 fingers per hand
4.  多生成几次

在网上也看到过一些 AI 模特的手指和关节都非常正常，其中有几个原作者也明确表示了有人工进行后期图片优化。

相信这个问题不久也可以得到解决。

## NSFW

估计大家已经开始玩起来了，甚至开始生成 NSFW 图片来对 AI 进行更全面的能力测试（满足自己奇怪的性癖），但是却发现效果始终不好。原因很简单啦，上述这些模型是生成正常的照片的，所以训练集中色情元素本身就没有很多，自然发挥不佳。寻找专门的 porn 模型进行尝试会效果更好。

如果你喜欢 ChilloutMax 的人脸和身材效果，可以喂给它一些你喜欢的 porn 元素的图片来生成特定的（满足自己奇怪的性癖）色情照片。

但记得不要做违法的事情（例如在不合适的场合传播色情照片、使用特定的人类去生成他的色情照片、生成儿童色情等）。