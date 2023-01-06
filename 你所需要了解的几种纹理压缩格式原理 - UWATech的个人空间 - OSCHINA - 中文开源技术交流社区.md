本文基于资料收集，概括了几种纹理压缩格式的基本思想，希望对于学习有所帮助。

> ### **为什么我们需要纹理压缩格式？**

例如 R5G6B5、A4R4G4B4、A1R5G5B5、R8G8B8 或 A8R8G8B8 等未经压缩的图片格式，是能够被 GPU 直接读取的原生纹理格式。但在低端硬件设备或者说移动平台下，有两个问题需要解决。

一个是**内存**，例如 A8R8G8B8 格式中一个像素占 4 字节，如果是 512x512 分辨率内存就占用 512x512x4 B=1048576 B=1 MB，这种内存消耗在低端设备上根本无法接受。

另一个重要的是数据传输时的**带宽**，带宽是发热的元凶，在渲染 3D 场景时，会有大量的贴图被传输到 GPU，若不限制，总线带宽很快就会成为瓶颈，手机秒变暖手宝，严重的还会影响渲染性能。

因此我们需要一种内存占用既小又能被 GPU 读取的格式 —— 压缩纹理格式。纹理压缩对应的算法是以某种形式的固定速率有损向量量化（Lossy Vector Quantization）将固定大小的像素块编码进固定大小的字节块中。

**有损**表示对于渲染来说，有损压缩是可以接受的，一般选择压缩格式时需要在纹理质量和文件大小上寻求一个平衡。

**固定速率压缩**指的是什么呢？因为 GPU 需要能够高效的随机访问一个像素，这意味着对任意像素，解码速度不该有太大的变化。因此，我们常见的贴图压缩算法都是有损压缩。相反的例如 zip 则是一种可变速率压缩。

**向量量化（Vector Quantization，VQ）**是一种量化技术，将一组大量的点（向量）分成具有近似相同数量的最接近它们的点的组。每个组用它的质心点表示，因此存在数据误差，适用于有损压缩。放到纹理压缩中来理解，就例如将 4x4 块像素的颜色以 2 个基色来表示。

**编码和解码速度**：一般来说编码速度慢没关系，因为通常纹理压缩只需要在游戏打包时进行一次，对于用户运行时体验完全没有影响。但解码速度必须足够快，而且基本上不能影响到渲染性能。

**压缩比**：通常以比特率或每像素的平均比特数（bits per pixel，bpp）表示，常见的为 2~8bpp。一般 RGB 原生纹理的像素指 24 位，4bpp 表示每像素占 4 位，所以也可以认为 4bpp 表示压缩比为 6:1。

顺便一提，在 Unity 中，任何图片文件格式都存在一个导入过程，导入后的文件格式都是 Texture2D，在 Texture2D 的导入设置选项中需要针对不同平台设置纹理压缩格式。

> ### **为什么我们不使用 png、jpg 这类常见的压缩格式？**

尽管像 jpg、png 的压缩率很高，但并不适合纹理，主要问题是不支持像素的随机访问，这对 GPU 相当不友好，GPU 渲染时只使用需要的纹理部分，我们总不可能为了访问某个像素去解码整张纹理吧？不知道顺序，也不确定相邻的三角形是否在纹理上采样也相邻，很难有优化。这类格式更适合下载传输以及减少磁盘空间而使用。

**常见纹理压缩格式**  
**ETC**  
ETC（Ericsson Texture Compression）最初为移动设备开发，如今它是安卓的标准压缩方案，ETC1 在 OpenGL 和 OpenGL ES 中都有支持。

其原理简单来说，是将 4x4 的像素块编码为 2x4 或 4x2 像素的两个块的方法，每个块指定一个基色，每个像素的颜色通过一个编码为相对于这些基色偏移的灰度值确定。

具体来说，ETC1 每 4x4 像素块编码为 64 位的字节数据，每一个像素块又分为两个 2x4 子块（由一个 “flip” 位控制水平或竖直划分），每个子块包含一个 3 位的修饰表索引（modifier table index）和一个基本颜色值，这两个颜色值要么是 2\*R4G4B4 要么是 R5G5B5+R3G3B3（由一个 “ diff” 位控制是哪一种）。

3 位的修饰表索引对应于 8 种修饰值：

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/1.png)

这样一个子块由 1 个基本颜色值和 4 个修饰值可以确定出 4 种新的颜色值：

```
color0 = base_color + RGB(modifier0, modifier0, modifier0) 
```

```
color1 = base_color + RGB(modifier1, modifier1, modifier1) 
```

```
color2 = base_color + RGB(modifier2, modifier2, modifier2) 
```

```
color3 = base_color + RGB(modifier3, modifier3, modifier3) 
```

最终的颜色就是从这 4 个颜色值中选出。其原理就是另外 32 位数据中包含 16 个 2 位选择器数据，每个像素的颜色都根据 2 位选择器的值从这 4 种中选出。

ETC1 编码方式直观图如下：

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/2.png)

Unity 的几种 ETC 纹理压缩格式：

RGB ETC1 4 bit：4 bits/pixel，对 RGB 压缩比 6:1，不支持 Alpha，绝大部分安卓设备都支持。

RGB ETC2 4 bit：4 bits/pixel，对 RGB 压缩比 6:1。不支持 Alpha，ETC2 兼容 ETC1，压缩质量可能更高，但对于色度变化大的块误差也更大，需要在 OpenGL ES 3.0 和 OpenGL 4.3 以上版本。

RGBA ETC2 8bit：8 bits/pixel，对 RGBA 压缩比 4:1。支持完全的透明通道，版本要求同上。

RGB +1bit Alpha ETC2 4bit：4 bits/pixel。支持 1bit 的 Alpha 通道，也就是只支持镂空图，图片只有透明和不透明部分，没有中间的透明度。

EAC：核心原理与 ETC 相同，但它只用于单通道或双通道数据，OpenGL ES 3.0 和 OpenGL 4.3 后的设备大部分支持，但由于安卓平台五花八门的兼容性，一般不建议用单双通道贴图。

**DXT**  
原名 S3TC（S3 Texture Compression ），由 S3 Graphics 开发的一种与组块有关的有损压缩算法，也叫 DXTn 或 DXTC（DirectX Texture），是块截断编码（Block Truncation Coding）的一种改进。由于版权专利一般用于 Windows 平台。

其原理简单来说，是由一对低精度的 “基色” 来描述一个 4x4 的 RGB 像素块，并允许每个像素在这些基色之间指定一个插值。S3TC 有多种变体，每种都是为特定类型的图像数据设计，但它们都是将 4x4 的像素块转换为 64 位或 128 位的数据。

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/3.png)

BC1（Block Compression）是最小的一种变体，也是转换比最高的一种，在不需要高精度也不需要 a 值时可以使用。它将 4x4 个像素作为一个块（block）存储 64 位数据，其中包括两个 16 位 RGB 值（R5G6B5，人眼对绿色更敏感）$color\_0$ 和 $color\_1$，和 16 个 2 位选择器值，一个像素最终的颜色值由 $color\_0$、$color\_1$ 和对应的 2 位选择器值决定。混合公式如下：

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/4.png)

它的工作方式直观图如下：

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/5.png)

PS：BC1 的信息丢失主要集中在比较细的边界上，可提高分辨率来解决，高宽放大 41%（1.41\*1.41=1.9881），这样整个纹理差不多是以前的 2 倍，但总的压缩率还能保持为 3:1。另一个损失是 24 位 RGB 转为了 16 位颜色，看上去差别不大，但对于渐变色带来说就有一些细微差别了。

PPS：在 Unity 2017 中新增了基于 Crunch 库的改进版 DXT 算法，占用磁盘空间更小，解压速度更快，但是相应的图像质量不如 DXT 那么好。

BC2 在 BC1 的基础上支持了 a 值。它将 4x4 共 16 个像素存储为 128 位数据，其中包括 64 位 a 通道（每像素 4 位）和 64 位颜色通道，颜色和 DXT1 一致，即两个 16 位 RGB 值和 16 个 2 位索引表 x。它的透明度只有 4 位共 16 种数值，相比 DXT5 有些功能基本没人用。DXT2 和 DXT3 的差别在于是否是预乘过的颜色。

BC3 在 BC2 的基础上改进了 a 值算法。BC3 将 4x4 共 16 个像素存储为 128 位数据，其中包括 64 位 a 通道（两个 8 位 a 值和一个 4x4 的 3 位索引表）和 64 位颜色通道，颜色和 DXT1 一样，即两个 16 位 RGB 值和 16 个 2 位索引表 x。由于 a 值采用了和颜色一样类似的块压缩算法，所以数值范围更广，BC 是比较常见的。

BC4 和 BC5 在 D3D10 中可用，只能存储一个 / 两个颜色通道。

BC6H 和 BC7 在 D3D11 中可用。BC6H 是一个不带 a 值的 HDR 格式，它将 4x4 共 16 个像素存储为 128 位数据，其中包括两个 48 位的 RGB 值（16:16:16），每个颜色分量都是带符号浮点值（1 个符号位 + 5 个指数位 + 10 个尾数位），以及 16 个 2 位索引表。

BC7 相比其它有点特殊，虽然它也是将 4x4 共 16 个像素存储为 128 位数据，但它的最低有效位为 Mode 位，（最低有效位即最低的非 0 位）根据不同模式，颜色值的存储格式不同，是否有 a 值或 a 值的存储格式也不尽相同，是一种比较灵活的存储格式，但这也意味着解码所带来更多的消耗。

**PVRTC**  
PVRTC（PowerVR Texture Compression）由 Imagination 公司专为 PowerVR 显卡核心设计，由于专利原因一般它只被用于苹果的设备，仅 iPhone、iPad 和部分 PowerVR 的安卓机支持。这可能是这几种压缩格式中最不公开的技术。

PVRTC 不同于 DXT 和 ETC 这类基于块的算法，而将整张纹理分为了高频信号和低频信号，低频信号由两张低分辨率的图像 A 和 B 表示，这两张图在两个维度上都缩小了 4 倍，高频信号则是全分辨率但低精度的调制图像 M，M 记录了每个像素混合的权重。要解码时，A 和 B 图像经过双线性插值（Bilinearly）宽高放大 4 倍，然后与 M 图上的权重进行混合。

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/6.png)

PVRTC 4-bpp 模式下，每 4x4 像素占一个 64 位数据块，2-bpp 模式下每 8x4 像素会有一个 64 位数据块。两者大同小异，我们仅说 4-bpp 模式。

4-bpp 模式下，A 和 B 图缩小后都只保存一个颜色值，如下图所示，A 图比 B 图少 1 位，但两张图都可以选择以 RGB 或 ARGB 的方式存储（最高位决定为哪种），A 色可以用 RGB554 或 ARGB3443 格式编码，B 色可以用 RGB555 或 ARGB3444 格式编码。

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/7.png)

在解码时，为了解码任意像素，必须读取 4 个相邻的 PVRTC 块，使用这 4 个块来解码一个 5x5 块。

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/8.png)

使用双线性过滤来对 A 和 B 图进行扩大，然后 A 和 B 图根据 M 图与 “Mode” 位进行混合，这里的 "Mode" 位为 1 时，M 图中 10 值像素被看作是开启了 “punch-through alpha”，Alpha 通道会被强制清零，这种神奇的操作是为了兼容旧应用程序，具体就不说了。

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/9.png)

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/10.png)

单从 4-bpp 模式来看，PVRTC 和 BC、ETC 非常相似，都有两个颜色值，但基本思想却是不同的。

Unity 的几种 PVRTC 的纹理压缩格式：

苹果的所有移动设备都支持 PVRTC。

PVRTC2 质量比 PVRTC 更高，而且支持 NPOT（非 2 次幂纹理），是 PVRTC 的升级版。

PVRTC 和 PVRTC2 都支持 4-bpp 和 2-bpp 的 ARGB 格式。

RGB PVRTC 4 bit：4 bits/pixel，对 RGB 压缩比 6:1，安卓设备需要 PowerVR Series 5 以上。

RGBA PVRTC 4 bit：4 bits/pixel，对 RGBA 压缩比 8:1，3 位 Alpha 值，设备同上。

RGB PVRTC 2 bit：2 bits/pixel，对 RGB 压缩比 6:1，安卓和 iOS 设备需要 PowerVR Series 5X 以上。

RGBA PVRTC 2 bit：2 bits/pixel，对 RGBA 压缩比 8:1，3 位 Alpha 值，设备同上。

**ASTC**  
ASTC（Adaptive Scalable Texture Compression），由 ARM 和 AMD 联合开发，2012 年发布，是较新的一种压缩格式，唯一一个不受专利权影响的压缩格式。ASTC 在压缩率、图像质量、种类上都挺不错的，也正在逐步代替前三种，最大的缺点可能就是兼容性还不够完善和解码时间较长，但以现在移动端的发展趋势来看，GPU 计算能力越来越难成为瓶颈，因此非常有希望在以后能成为统一的压缩格式。

ASTC 也是一种基于块的有损压缩算法，它很像 BC7，不同的是块中像素数量可变，它的特点有很多：

-   较高的灵活性：支持 1-4 分量的贴图
-   压缩率 / 质量灵活可变：根据不同图片会选择不同压缩率级别的算法
-   支持 2D/3D 贴图
-   跨平台：iOS、安卓、PC
-   同时支持 LDR 和 HDR：BC6H 虽然支持 HDR 但不支持 Alpha 通道

ASTC 格式的块为固定大小的 128 位。2D 纹理编码中，它从 4x4 到 12x12 像素都有，对应的压缩比从 3:1 到 27:1。所有支持的块和比特率如下：

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/11.png)

Increment 列表示压缩比的增量，这也表示 ASTC 的比特率可以在小数级变化，这种技术称为 BISE（Bounded Integer Sequence Encoding）。

ASTC 的压缩非常复杂，分了很多可变的配置数据，也不打算再深入研究了。

但要注意的是尽管纹理可以被编码为 1-4 通道图像，但是解码后的值总是以 RGBA 格式输出。在 LDR sRGB 模式下，颜色值以 8 位整数的形式返回，而如果是 HDR 则将以 16 位浮点数的形式返回。

[《ASTC 纹理压缩格式》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fblog.uwa4d.com%2Farchives%2FUsparkle_ASTC.html)对 ASTC 做了详细的测试，非常建议去看看。

> ### **总结**

整理了下网上的资料，以 2020 年市场上的移动设备支持来看：

-   安卓：用 ETC2 没有什么问题；至于 ASTC 在 Android 5.0/OpenGL ES 3.1 后支持，市场大部分机型都支持（98.5%），可以考虑选择，但毕竟是安卓，要做好处理兼容性的心里准备。
-   iOS：在 iPhone6 以上（包含）都支持 ASTC，6 以下可以选择 PVRTC2。

最后，所有压缩格式的简要描述如下，灰色框表示已经过时了或基本不用了。

![](http://uwa-ducument-img.oss-cn-beijing.aliyuncs.com/Blog/USparkle_Texture/12.png)

___

参考来源：

[TEXTURE COMPRESSION TECHNIQUES](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fsv-journal.org%2F2014-1%2F06%2Fen%2Findex.php%3Flang%3Den)

[Recommended, default, and supported texture compression formats, by platform](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.unity3d.com%2FManual%2Fclass-TextureImporterOverride.html)

[Another Milestone for ASTC Texture Compression](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fcommunity.arm.com%2Farm-community-blogs%2Fb%2Fgraphics-gaming-and-vr-blog%2Fposts%2Fanother-milestone-for-astc-texture-compression)

[ASTC does it](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fcommunity.arm.com%2Farm-community-blogs%2Fb%2Fgraphics-gaming-and-vr-blog%2Fposts%2Fastc-does-it)

[《ASTC 纹理压缩格式》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fblog.uwa4d.com%2Farchives%2FUsparkle_ASTC.html)

[《移动平台纹理压缩格式选择》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fblog.uwa4d.com%2Farchives%2FTechSharing_186.html)

[Why use crunch compression?](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fforum.unity.com%2Fthreads%2Fwhy-use-crunch-compression.581986%2F)

[Crunch compression of ETC textures](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fblog.unity.com%2Ftechnology%2Fcrunch-compression-of-etc-textures)

[ETC1 Compressed Texture Image Formats](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fregistry.khronos.org%2FDataFormat%2Fspecs%2F1.1%2Fdataformat.1.1.html%23ETC1)

[Texture compression on mobile demystified](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ffrozenfractal.com%2Fblog%2F2017%2F2%2F20%2Ftexture-compression-on-mobile-demystified%2F)

[Texture compression using low-frequency signal modulation](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.researchgate.net%2Fpublication%2F221249059_Texture_compression_using_low-frequency_signal_modulation)

___

这是侑虎科技第 1187 篇文章，感谢作者蕾芙丽 Reverie 供稿。欢迎转发分享，未经作者授权请勿转载。如果您有任何独到的见解或者发现也欢迎联系我们，一起探讨。（QQ 群：793972859）

作者主页：[https://www.zhihu.com/people/hei-jing-79](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.zhihu.com%2Fpeople%2Fhei-jing-79)

再次感谢蕾芙丽 Reverie 的分享，如果您有任何独到的见解或者发现也欢迎联系我们，一起探讨。（QQ 群：793972859）