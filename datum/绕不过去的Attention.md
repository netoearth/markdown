[学习笔记](https://yinguobing.com/tag/xue-xi-bi-ji/)

Attention is all you need.

-   [![Yin Guobing](https://yinguobing.com/content/images/size/w100/2020/10/10267910.jpg)](https://yinguobing.com/author/yinguobing/)

May 12, 2021 • 8 min read

![绕不过去的Attention](https://yinguobing.com/content/images/size/w2000/2021/05/priscilla-du-preez-QTx_TbHmWDs-unsplash-2.jpg)

封面图片：[Priscilla Du Preez](https://unsplash.com/@priscilladupreez?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

_Attention Is All You Need_ 是由Google Brain的Ashish Vaswani等人在2017年发表的文章。文中提出了Transformer，一种仅仅基于注意力机制的网络架构，在传统的语言翻译中获得了优异的表现。你可以在这里找到原文：

[

Attention Is All You Need

The dominant sequence transduction models are based on complex recurrent orconvolutional neural networks in an encoder-decoder configuration. The bestperforming models also connect the encoder and decoder through an attentionmechanism. We propose a new simple network architecture, the Transformer…

![](https://static.arxiv.org/static/browse/0.3.2.6/images/icons/favicon.ico)arXiv.orgAshish Vaswani

![](https://static.arxiv.org/static/browse/0.3.2.6/images/icons/social/bibsonomy.png)

](https://arxiv.org/abs/1706.03762)

以下内容为对应的学习笔记。

___

自注意力（self-attention），有时也被称为内部注意力（intra-attention），是一种将单个序列中不同位置联系起来以获取新的序列表达的注意力机制。

常见的语言翻译架构多采用encoder-decoder模式。Encoder将输入序列(_x__1_ _,..., x__n_)转换为中间表达_z_ = (_z1, ..., zn_)，然后decoder根据_z_生成输出(_y1, ..., ym_)，一次处理一个符号。模型的每一步都是自回归的，下一个元素会将之前生成的元素作为额外输入。

Transformer大体上遵循了这种架构，在encoder与decoder中都使用堆叠的自注意力与逐点全连接层。

![](https://yinguobing.com/content/images/2021/05/transformer-01-2.jpg)

Transformer架构。作者：Ashish Vaswani，等。

### Encoder（编码器）

编码器由6个层结构叠加而成，每一层包括2个子层。第一层是多头自注意力机制，第二层是position- wise的全连接前向传播网络。两个子层都有自己的残差连接，并跟随layer normalization。所有子层的输出维度均为512。

![](https://yinguobing.com/content/images/2021/05/transformer-02-1.jpg)

编码器。作者：Ashish Vaswani，等。

### Decoder（解码器）

解码器同样是由6个层结构叠加而成。与编码器相比，解码器额外增加了一个子层，用于针对编码器输出的多头注意力机制。每一个子层同样拥有残差连接与layer normalization。作者还针对自注意力子层做了修订阻止了当前位置对后续位置的注意力。这种掩码机制保证了位置i的输出仅仅与位置i之前的已知输出相关。

![](https://yinguobing.com/content/images/2021/05/transformer-03-2.jpg)

解码器。作者：Ashish Vaswani，等。

### 注意力

一个注意力函数可以被描述为针对查询（query）与键值（key-value）数据集合的一种映射输出，其中query、key、value与输出均为向量。输出本质上是对value的加权和，而权重则取决于query与key。

### Scaled Dot-Product Attention

作者将文中的注意力称为Scaled Dot-Product Attention（缩放的点乘注意力）。其输入包括query，维度为 _d__k_ 的key以及维度为 _dv_ 的value。首先计算query与key的点乘，然后再除以sqrt(_d__k_)，再使用softmax即可获得value的权重。

![](https://yinguobing.com/content/images/2021/05/transformer-04.jpg)

Scaled Dot-Product Attention。作者：Ashish Vaswani，等。

实际中，作者将一个query集合打包成矩阵_Q_来实现批量计算，同样key与value也分别打包为矩阵K与V。注意力可以写为：

$$\\text{Attention}(Q,K,V)=\\text{softmax}(\\frac{QK^T}{\\sqrt{d\_k}})V$$

Additive attention与dot-product attention是两种常见的注意力函数。文中的函数与dot-product attention最大的差异在于缩放因子 $1/\\sqrt{d\_k}$ 。Additive attention使用具备单个隐含层的前向传播网络来计算兼容函数。两者在运算复杂度上相似，但是借助高度优化的矩阵相乘算法，点乘注意力的效率更高。作者发现当 _dk_ 的数值较大时，点乘后的数值过大导致softmax的梯度值变小，为此添加了缩放因子作为补偿。

### Multi-Head Attention（多头注意力）

相较于将一个注意力函数用在维度为d的query、key和value上，作者发现将query、key和value通过习得的线性透射参数变换_h_次到维度_d__k__、 dk_和_dv_ 上效果更好。这样可以并行计算注意力函数。之后再将结果拼接、投射生成最终结果。

![](https://yinguobing.com/content/images/2021/05/transformer-05.jpg)

Multi-Head Attention。作者：Ashish Vaswani，等。

多头注意力机制使得模型可以联合考虑不同位置不同表达子空间的信息。文中作者使用了_h_\=8，_dk_和_dv_ 为64，这样与单头的维度总体保持一致。

___

至此，模型采用了三种不同的注意力形式。

-   在编解码注意力层中，query来自前一个解码器层，key与value则来自编码器的输出。这使得解码器中的每一个位置都可以照顾到输入序列的全部位置。这与经典的编解码注意力机制类似。
-   编码器自注意力层中，query、key与value都来自编码器的前一层。编码器的每个位置可以照顾到前一层的全部位置。
-   类似的，解码器的自注意层中每个位置可以照顾到包含自己在内的所有前序位置。为保证解码器的自回归属性，需要阻断其左向的信息流动。作者通过mask的形式将其掩去。

### Position-wise Feed-Forward Networks

除了注意力层外，编解码器还包含了一个全连接前向传播网络，并使用ReLU激活函数。

$$\\text{FFN}(x) = \\text{max}(0, xW\_1+ b\_1 )W\_2 + b\_2$$

尽管不同位置处的线性变换是相同的，但是层与层采用了不同的参数。另一种等效的描述方式为两次1x1卷积，输入输出的维度为512，中间维度为2048。

### 嵌入与Softmax

与其它序列翻译模型类似，Transformer使用了习得的嵌入将输入与输出转换为维度为d的向量。同时还使用习得的线性变换与softmax将解码器输出转换为下一个token的预测概率。

### Positional Encoding

由于模型没有使用回归与卷积，为了使其充分利用序列的顺序信息，需要为其注入序列中token的相对与绝对位置信息。作者在编码器与解码器的层堆叠底端添加了包含位置编码的输入嵌入。位置编码的维度与模型相同。

![](https://yinguobing.com/content/images/2021/05/transformer-06.jpg)

Positional Encoding。作者：Ashish Vaswani，等。

具体的Position Encoding，作者采用了正余弦函数：

$$PE\_{(pos,2i)} =sin(pos/10000^{2i/d\_{model}}) \\\\ PE\_{(pos,2i+1)} =cos(pos/10000^{2i/d\_{model}})$$

其中pos为位置，i为维度。其理由是对于固定偏移_k_，$PE\_{pos+k}$可以看做是$PE\_{pos}$的线性变换，这样便于模型学习到相对位置信息。

### 为何选用自注意力

三个考虑：第一是每一层的运算复杂度；第二是并行计算的可能性；第三是网络中长距离依赖的路径长度。作者总结如下：

![](https://yinguobing.com/content/images/2021/05/transformer-07.jpg)

不同运算的复杂度。作者：Ashish Vaswani，等。

相比之下，Self-attention的耗费更小。

___

至此，Transformer的架构基本描述完整。原文的后续为大量的实验性质内容，不再单列。

如果你对Attention机制感兴趣，Distill还有一篇非常棒的文章提供了可以互动的示意图来辅助理解。