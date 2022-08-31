## 文章目录

-   Attention Is All You Need
    
-   -   摘要
        
    -   [1 Introduction（简介）](https://blog.csdn.net/nocml/article/details/103082600#1_Introduction_8)
    -   [2 Background（背景）](https://blog.csdn.net/nocml/article/details/103082600#2_Background_17)
    -   [3 Model Architecture（模型结构）](https://blog.csdn.net/nocml/article/details/103082600#3_Model_Architecture_27)
    -   -   [3.1 Encoder and Decoder Stacks（编码器栈和解码器栈）](https://blog.csdn.net/nocml/article/details/103082600#31_Encoder_and_Decoder_Stacks_36)
        -   [3.2 Attention（注意力机制）](https://blog.csdn.net/nocml/article/details/103082600#32_Attention_41)
        -   [3.2.1 Scaled Dot-Product Attention（缩放的点积注意力机制）](https://blog.csdn.net/nocml/article/details/103082600#321_Scaled_DotProduct_Attention_47)
        -   [3.2.2 Multi-Head Attention（多头注意力机制）](https://blog.csdn.net/nocml/article/details/103082600#322_MultiHead_Attention_57)
        -   [3.2.3 Applications of Attention in our Model（注意力机制在我们模型中的应用）](https://blog.csdn.net/nocml/article/details/103082600#323_Applications_of_Attention_in_our_Model_70)
        -   [3.3 Position-wise Feed-Forward Networks（基于位置的前馈神经网络）](https://blog.csdn.net/nocml/article/details/103082600#33_Positionwise_FeedForward_Networks_78)
        -   [3.4 Embeddings and Softmax （词嵌入和 softmax）](https://blog.csdn.net/nocml/article/details/103082600#34_Embeddings_and_Softmax__softmax_85)
        -   [3.5 Positional Encoding（位置编码）](https://blog.csdn.net/nocml/article/details/103082600#35_Positional_Encoding_89)
        -   [4 Why Self-Attention（为什么选择selt-attention）](https://blog.csdn.net/nocml/article/details/103082600#4_Why_SelfAttentionseltattention_107)
    -   [5 Training(训练)](https://blog.csdn.net/nocml/article/details/103082600#5_Training_121)
    -   -   [5.1 Training Data and Batching（训练数据和batch）](https://blog.csdn.net/nocml/article/details/103082600#51_Training_Data_and_Batchingbatch_124)
        -   [5.2 Hardware and Schedule(硬件和时间)](https://blog.csdn.net/nocml/article/details/103082600#52_Hardware_and_Schedule_128)
        -   [5.3 Optimizer（优化器）](https://blog.csdn.net/nocml/article/details/103082600#53_Optimizer_132)
        -   [5.4 Regularization（正则化）](https://blog.csdn.net/nocml/article/details/103082600#54_Regularization_139)
    -   [6 Results（结果）](https://blog.csdn.net/nocml/article/details/103082600#6_Results_148)
    -   -   [6.1 Machine Translation（机器翻译）](https://blog.csdn.net/nocml/article/details/103082600#61_Machine_Translation_149)
        -   [6.2 Model Variations（模型变体）](https://blog.csdn.net/nocml/article/details/103082600#62_Model_Variations_161)
        -   [6.3 English Constituency Parsing（英文选区分析）](https://blog.csdn.net/nocml/article/details/103082600#63_English_Constituency_Parsing_172)
    -   [7 Conclusion（结论）](https://blog.csdn.net/nocml/article/details/103082600#7_Conclusion_186)

## 

Attention Is All You Need

## 

摘要

  主流的序列转换模型都是基于复杂的循环神经网络或卷积神经网络，且都包含一个encoder和一个decoder。表现最好的模型还通过attention机制把encoder和decoder联接起来。我们提出了一个新的、简单的网络架构，Transformer. 它只基于单独的attention机制，完全避免使用循环和卷积。在两个翻译任务上表明，我们的模型在质量上更好，同时具有更高的并行性，且训练所需要的时间更少。我们的模型在 WMT2014 英语-德语的翻译任务上取得了28.4的BLEU评分。在现有的表现最好模型的基础上，包括整合模型，提高了2个BLUE评分。在WMT2014英语-德语的翻译任务上,我们的模型在8个GPU上训练了3.5天（这个时间只是目前文献中记载的最好的模型训练成本的一小部分），创造了单模型的SOTA结果，BLEU分数为41.8，通过在大量和少量训练数据上所做的英语选区分析工作的成功，表明Transformer能很好的适应于其它任务。

## 1 Introduction（简介）

  RNN,LSTM,GRU,Gated Recurrent Neural Networks 在序列建模和转换任务上，比如语言模型和机器翻译，已经是大家公认的取得SOTA结果的方法。自此，无数的努力继续推动递归语言模型和encoder-decoder体系结构的界限。

  递归模型通常沿输入和输出序列的符号位置进行因子计算。在计算时将位置与步骤对齐，它们生成一系列隐藏状态h t h\_tht，t tt位置的h t h\_tht使用它的前驱h t − 1 h\_{t-1}ht−1和当前的输入生成。这种内部的固有顺阻碍了训练样本的并行化，在序列较长时，这个问题变得更加严重，因为内存的限制限制了样本之间的批处理。最近的工作通过因子分解技巧\[21\]和条件计算\[32\]在计算效率方面取得了显著的提高，同时也提高了后者的模型性能。然而，顺序计算的基本约束仍然存在。

  在各种各样的任务中，注意力机制已经成为各种引人注目的序列模型和转换模型中的不可或缺的组成部分，它允许对依赖关系建模，而不需要考虑它们在输入或输出序列中的距离。然而，在除少数情况外的所有情况下\[27\]，这种注意机制都与一个递归网络结合使用。

  在这项工作中，我们提出了Transformer，这是一种避免使用循环的模型架构，完全依赖于注意机制来绘制输入和输出之间的全局依赖关系。Transformer允许更显著的并行化，使用8个P100 gpu只训练了12小时，在翻译质量上就可以达到一个新的SOTA。

## 2 Background（背景）

  减少序列计算的目标也成就了 Extended Neural GPU \[16\],ByteNet\[18\],和ConvS2S\[9\]的基础,它们都使用了卷积神经网络作为基础模块，并行计算所有输入和输出位置的隐藏表示。在这些模型中，将来自两个任意输入或输出位置的信号关联起来所需的操作数，随位置间的距离而增长，ConvS2S为线性增长，ByteNet为对数增长。这使得学习远距离位置之间的依赖性变得更加困难\[12\]. 在Transformer中，这种情况被减少到了常数次操作，虽然代价是由于平均 注意力加权位置信息降低了有效分辨率，如第3.2节所述，我们用多头注意力抵消这种影响。

  self-attention,有时也叫做内部注意力，是一种注意力机制，它将一个序列的不同位置联系起来，以计算序列的表示。self-attention 已经成功的运用到了很多任务上，包括阅读理解、抽象摘要、语篇蕴涵和学习任务无关的句子表征等。

  已经被证明，端到端的记忆网络使用循环attention机制替代序列对齐的循环，在简单的语言问答和语言建模任务中表现良好。

  然而，据我们所知，Transformer是第一个完全依赖于self-attetion来计算其输入和输出表示而不使用序列对齐的RNN或卷积的转换模型，在下面的章节中，我们将描述Transformer,motivate ,self-attention，并讨论它相对于\[17,18\]和\[9\]等模型的优势。

## 3 Model Architecture（模型结构）

  大多数有竞争力的序列转换模型都有encoder-decoder结构构。这里，encoder将符号表示的输入序列( x 1 , . . . , x n ) (x\_1,...,x\_n)(x1,...,xn)映射成一个连续表示的序列z = ( z 1 , . . . , z n ) z = (z\_1,...,z\_n)z=(z1,...,zn)。给定z zz，解码器以一次生成一个字符的方式生成输出序列( y 1 , . . . , y m ) (y\_1,...,y\_m)(y1,...,ym) 。在每一步，模型都是自回归的\[10\]，在生成下一个字符时，将先前生成的符号作为附加输入。

  Transformer遵循这个总体架构，使用堆叠的self-attention层、point-wise和全连接层，分别用于encoder和decoder，如图1的左半部分和右半部分所示。

![Alt](https://img-blog.csdnimg.cn/20191108232051247.png#)

### 3.1 Encoder and Decoder Stacks（编码器栈和解码器栈）

**Encoder**:encoder由N(N=6)个完全相同的layer堆叠而成.每层有两个子层。第一层是multi-head self-attention机制，第二层是一个简单的、位置全连接的前馈神经网络。我们在两个子层的每一层后采用残差连接\[11\]，接着进行layer normalization\[1\]。也就是说，每个子层的输出是L a y e r N o r m ( x + S u b l a y e r ( x ) ) LayerNorm(x + Sublayer(x))LayerNorm(x+Sublayer(x))，其中S u b l a y e r ( x ) Sublayer(x)Sublayer(x) 是由子层本身实现的函数。为了方便这些残差连接，模型中的所有子层以及embedding层产生的输出维度都为d m o d e l = 512 d\_{model} = 512dmodel=512。

**Decoder**: decoder也由N(N=6)个完全相同的layer堆叠而成.除了每个编码器层中的两个子层之外，解码器还插入第三个子层，该子层对编码器堆栈的输出执行multi-head attention操作，与encoder相似，我们在每个子层的后面使用了残差连接，之后采用了layer normalization。我们也修改了decoder stack中的 self-attention 子层，以防止当前位置信息中被添加进后续的位置信息。这种掩码与偏移一个位置的输出embedding相结合， 确保对第i ii个位置的预测 只能依赖小于i ii 的已知输出。

### 3.2 Attention（注意力机制）

  Attention机制可以描述为将一个query和一组key-value对映射到一个输出，其中query，keys，values和输出均是向量。输出是values的加权求和，其中每个value的权重 通过query与相应key的兼容函数来计算。  
![Alt](https://img-blog.csdnimg.cn/20191110165037762.png#)  
Figure 2: (left) Scaled Dot-Product Attention. (right) Multi-Head Attention consists of several attention layers running in parallel.

### 3.2.1 Scaled Dot-Product Attention（缩放的点积注意力机制）

  我们称我们的特殊attention为Scaled Dot-Product Attention(Figure 2)。输入由query、d k d\_kdk的key和d v d\_vdv的value组成。我们计算query和所有key的点积，再除以d k \\sqrt{d\_k}dk,然后再通过softmax函数来获取values的权重。

  在实际应用中，我们把一组query转换成一个矩阵Q，同时应用attention函数。key和valuue也同样被转换成矩阵K和矩阵V。我们按照如下方式计算输出矩阵：  
A t t e n t i o n ( Q , K , V ) = s o f t m a x ( Q K t d k ) V Attention(Q,K,V)=softmax(\\frac{QK^t}{\\sqrt{d\_k}})VAttention(Q,K,V)=softmax(dkQKt)V

  additive attention和dot-product(multi-plicative) attention是最常用的两个attention 函数。dot-product attention除了没有使用缩放因子1 d k \\frac{1}{\\sqrt{d\_k}}dk1外，与我们的算法相同。Additive attention使用一个具有单隐层的前馈神经网络来计算兼容性函数。尽管在理论上两者的复杂度相似，但是在实践中dot-product attention要快得多，而且空间效率更高，这是因为它可以使用高度优化的矩阵乘法代码来实现。

  当d k d\_kdk的值较小时，这两种方法性能表现的相近，当d k d\_kdk比较大时，addtitive attention表现优于 dot-product attention。我们认为对于大的d k d\_kdk，点积在数量级上增长的幅度大，将softmax函数推向具有极小梯度的区域4 ^44。为了抵消这种影响，我们对点积扩展1 d k \\frac{1}{\\sqrt{d\_k}}dk1倍。

### 3.2.2 Multi-Head Attention（多头注意力机制）

  相比于使d m o d e l d\_{model}dmodel维度的queries,keys,values执行一个attention函数，我们发现使用不同的学习到的线性映射把queries, keys 和 values线性映射到d k d\_kdk, d k d\_kdk 和 d v d\_vdv维度h次是有益的。在queries,keys和values的每个映射版本上，我们并行的执行attention函数，生成d v d\_vdv维输出值。它们被拼接起来再次映射，生成一个最终值，如 Figure 2 中所示。

  Multi-head attention允许模型把不同位置子序列的表示都整合到一个信息中。如果只有一个attention head，它的平均值会削弱这个信息。

M u l t i H e a d ( Q , K , V ) = C o n c a t ( h e a d 1 , . . . , h e a d h ) W o MultiHead(Q,K,V) = Concat(head\_1, ..., head\_h)W^oMultiHead(Q,K,V)=Concat(head1,...,headh)Wo  
w h e r e 　 h e a d i = A t t e n t i o n ( Q W i Q , K W i K , V W i V ) where　head\_i=Attention(QW^Q\_i,KW^K\_i,VW^V\_i)where　headi=Attention(QWiQ,KWiK,VWiV)

  其中，映射为参数矩阵W i Q ∈ R d m o d e l × d k W\_i^Q ∈ ℝ^{d\_{model}×d\_k}WiQ∈Rdmodel×dk , W i K ∈ R d m o d e l × d k W\_i^K ∈ ℝ^{d\_{model}×d\_k}WiK∈Rdmodel×dk , W i V ∈ R d m o d e l × d v W\_i^V ∈ ℝ^{d\_{model}×d\_v}WiV∈Rdmodel×dv 及W O ∈ R h d v × d m o d e l W^O ∈ ℝ^{hd\_v×d\_{model}}WO∈Rhdv×dmodel。

在这项工作中，我们采用h = 8 h = 8h=8 个并行attention层或head。 对每个head，我们使用 d k = d v = d m o d e l / h = 64 d\_k = d\_v = d\_{model}/h = 64dk=dv=dmodel/h=64。 由于每个head尺寸上的减小，总的计算成本与具有全部维度的单个head attention相似。

### 3.2.3 Applications of Attention in our Model（注意力机制在我们模型中的应用）

multi-head attention在Transformer中有三种不同的使用方式：

-   在encoder-decoder attention层中，queries来自前面的decoder层，而keys和values来自encoder的输出。这使得decoder中的每个位置都能关注到输入序列中的所有位置。 这是模仿序列到序列模型中典型的编码器—解码器的attention机制，例如\[38, 2, 9\]。
-   encoder包含self-attention层。 在self-attention层中，所有的key、value和query来自同一个地方，在这里是encoder中前一层的输出。 encoder中的每个位置都可以关注到encoder上一层的所有位置。
-   类似地，decoder中的self-attention层允许decoder中的每个位置都关注decoder层中当前位置之前的所有位置（包括当前位置）。 为了保持解码器的自回归特性，需要防止解码器中的信息向左流动。我们在scaled dot-product attention的内部 ，通过屏蔽softmax输入中所有的非法连接值（设置为 −∞）实现了这一点。

### 3.3 Position-wise Feed-Forward Networks（基于位置的前馈神经网络）

  除了encoder子层之外，我们的encder和decoder中的每个层还包含一个全连接的前馈网络，该网络分别单独应用于每一个位置。这包括两个线性转换，中间有一个ReLU激活。  
KaTeX parse error: Expected 'EOF', got '&' at position 41: …1 )W\_2 + b\_2 &̲emsp;&emsp;&ems…

  尽管线性变换在不同位置上是相同的，但它们在层与层之间使用不同的参数。 它的另一种描述方式是两个内核大小为1的卷积。 输入和输出的维度为d m o d e l d\_{model}dmodel = 512，内部层的维度为d f f d\_{ff}dff = 2048。

### 3.4 Embeddings and Softmax （词嵌入和 softmax）

  与其他序列转换模型类似，我们使用学习到的嵌入词向量 将输入字符和输出字符转换为维度为d m o d e l d\_{model}dmodel的向量。我们还使用普通的线性变换和softmax函数将decoder输出转换为预测的下一个词符的概率。在我们的模型中，两个嵌入层之间和pre-softmax线性变换共享相同的权重矩阵，类似于\[30\]。 在嵌入层中，我们将这些权重乘以d m o d e l \\sqrt{d\_{model}}dmodel

### 3.5 Positional Encoding（位置编码）

  由于我们的模型不包含循环或卷积，为了让模型利用序列的顺序信息，我们必须加入序列中关于字符相对或者绝对位置的一些信息。 为此，我们在encoder和decoder堆栈底部的输入嵌入中添加“位置编码”。 位置编码和嵌入的维度d m o d e l d\_{model}dmodel相同，所以它们两个可以相加。有多种位置编码可以选择，例如通过学习得到的位置编码和固定的位置编码\[9\]

Table1:Maximum path lengths, per-layer complexity and minimum number of sequential operations for different layer types. n is the sequence length, d is the representation dimension, k is the kernel size of convolutions and r the size of the neighborhood in restricted self-attention.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191113165200963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vY21s,size_16,color_FFFFFF,t_70)

  在这项工作中，我们使用不同频率的正弦和余弦函数:

P E ( p o s , 2 i ) = s i n ( p o s / 1000 0 2 i / d m o d e l ) P E ( p o s , 2 i + 1 ) = c o s ( p o s / 1000 0 2 i / d m o d e l ) PE\_{(pos,2i)} = sin(pos/10000^{2i/d\_{model}})\\\\ PE\_{(pos,2i+1)} = cos(pos/10000^{2i/d\_{model}})PE(pos,2i)=sin(pos/100002i/dmodel)PE(pos,2i+1)=cos(pos/100002i/dmodel)

  其中pos 是位置，i 是维度。也就是说，位置编码的每个维度对应于一个正弦曲线。波长形成了从2π到10000·2π的几何数列。我们之所以选择这个函数，是因为我们假设它可以让模型很容易地通过相对位置来学习,因为对任意确定的偏移k, P E p o s + k PE\_{pos+k}PEpos+k可以表示为P E p o s PE\_{pos}PEpos的线性函数。

  我们还尝试使用预先学习的positional embeddings\[9\]来代替正弦波，发现这两个版本产生了几乎相同的结果 (see Table 3 row (E))。我们之所以选择正弦曲线，是因为它允许模型扩展到比训练中遇到的序列长度更长的序列。

### 4 Why Self-Attention（为什么选择selt-attention）

  在这一节中，我们将self-attention layers与常用的recurrent layers和convolutional layers进行各方面的比较，比较的方式是 将一个可变长度的符号表示序列 ( x 1 , . . . , x n ) (x\_1, ..., x\_n)(x1,...,xn)映射到另一个等长序列( z 1 , . . . , z n ) (z\_1, ..., z\_n)(z1,...,zn)，用x i , z i ∈ R d x\_i,z\_i ∈ ℝ^dxi,zi∈Rd，比如在典型的序列转换的encoder或decoder中的隐藏层。我们考虑三个方面，最后促使我们使用self-attention。

  一是每层的总计算复杂度。另一个是可以并行化的计算量，以所需的最小序列操作数衡量。

  第三个是网络中长距离依赖关系之间的路径长度。在许多序列转换任务中，学习长距离依赖性是一个关键的挑战。影响学习这种依赖关系能力的一个关键因素是网络中向前和向后信号必须经过的路径的长度。输入和输出序列中任意位置组合之间的这些路径越短，越容易学习长距离依赖。因此，我们还比较了在由different layer types组成的网络 中的任意两个输入和输出位置之间的最大的路径长度。

  如表1所示，self-attention layer用常数次(O ( 1 ) O(1)O(1))的操作连接所有位置，而recurrent layer需要O ( n ) O(n)O(n)顺序操作。在计算复杂度方面，当序列长度N小于表示维度D时，self-attention layers比recurrent layers更快，这是使用最先进的机器翻译模型表示句子时的常见情况，例如word-piece \[38\] 和byte-pair \[31\] 表示。为了提高包含很长序列的任务的计算性能，可以仅在以输出位置为中心，半径为r的的领域内使用self-attention。这将使最大路径长度增长到O ( n / r ) O(n/r)O(n/r)。我们计划在今后的工作中进一步研究这种方法。

  核宽度为k < n k < nk<n的单层卷积不会连接每一对输入和输出的位置。要这么做，在相邻的内核情况下，需要一个n个卷积层的堆栈， 在扩展卷积的情况下需要O ( l o g k ( n ) ) O(logk(n))O(logk(n)) 层\[18\]，它们增加了网络中任意两个位置之间的最长路径的长度。 卷积层通常比循环层代价更昂贵，这与因子k有关。然而，可分卷积\[6\]大幅减少复杂度到O ( k ⋅ n ⋅ d + n ⋅ d 2 ) O(k ⋅n⋅d + n⋅d^2)O(k⋅n⋅d+n⋅d2)。然而，即使k = n k= nk=n，可分离卷积的复杂度等于self-attention layer和point-wise feed-forward layer的组合，这是我们在模型中采用的方法。

  一个随之而来的好处是，self-attention可以产生更多可解释的模型。我们从我们的模型中研究attention的分布，并在附录中展示和讨论示例。每个attention head不仅清楚地学习到执行不同的任务，还表现出了许多和句子的句法和语义结构相关的行为。

## 5 Training(训练)

  本节介绍我们的模型训练方法。

### 5.1 Training Data and Batching（训练数据和batch）

  我们在标准的WMT 2014英语-德语数据集上进行了训练，其中包含约450万个句子对。 这些句子使用byte-pair编码\[3\]进行编码，源语句和目标语句共享大约37000个词符的词汇表。 对于英语-法语翻译，我们使用大得多的WMT 2014英法数据集，它包含3600万个句子，并将词符分成32000个word-piece词汇表\[38\]。 序列长度相近的句子一起进行批处理。 每个训练批次的句子对包含大约25000个源词符和25000个目标词符。

### 5.2 Hardware and Schedule(硬件和时间)

  我们在一台具有8个 NVIDIA P100 gpu的机器上训练我们的模型。对于paper中描述的使用超参数的基础模型，每个训练步骤大约需要0.4秒。我们对基础模型进行了总共100000步或12小时的训练。对于我们的大型模型（见表3的底线），步进时间为1.0秒。大模型 使用了30万步（3.5天）的训练。

### 5.3 Optimizer（优化器）

  我们使用Adam优化器\[20\]，其中β1 = 0.9, β2 = 0.98及ϵ= 10-9。 我们根据以下公式在训练过程中改变学习率：

KaTeX parse error: Expected 'EOF', got '&' at position 77: …\\\_steps^{−1.5})&̲emsp;&emsp;&ems…

  这对应于在第一次warmup\_steps 步骤中线性地增加学习速率，并且随后将其与步骤数的平方根成比例地减小。 我们使用w a r m u p \_ s t e p s = 4000 warmup\\\_steps = 4000warmup\_steps=4000。

### 5.4 Regularization（正则化）

  训练中我们采用三种正则化：

**Residual Dropout** 我们在对每个子层的输出上执行dropout操作,这个操作在additive操作（子层的输出加上子层的输入）和 normalized操作之前。 此外，在编码器和解码器堆栈中，我们将丢弃应用到嵌入和位置编码的和。 对于基础模型，我们使用P d r o p P\_{drop}Pdrop = 0.1丢弃率。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191113145936410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vY21s,size_16,color_FFFFFF,t_70)

**Label Smoothing** 在训练过程中，我们采用了值ε l s = 0.1 ε\_{ls} = 0.1εls=0.1\[36\]的标签平滑。这会影响ppl,因为模型学习到了更多的不确定性，但提高了准确率和BLEU评分。

## 6 Results（结果）

### 6.1 Machine Translation（机器翻译）

  在WMT 2014英语-德语翻译任务中，大型Transformer模型（表2中的Transformer (big)）比以前报道的最佳模型（包括整合模型）高出2个以上的BLEU评分，以28.4分建立了一个全新的SOTA BLEU分数。 该模型的配置列在表3的底部。 在8 个P100 GPU上花费3.5 天进行训练。 即使我们的基础模型也超过了以前发布的所有模型和整合模型，且训练成本只是这些模型的一小部分。

  我们的模型在 WMT2014 英语-德语的翻译任务上取得了28.4的BLEU评分。在现有的表现最好模型的基础上，包括整合模型，提高了2个BLUE评分。

     在WMT 2014英语-法语翻译任务中，我们的大型模型的BLEU得分为41.0，超过了之前发布的所有单一模型，训练成本低于先前最先进模型的1 ∕ 4 。 英语-法语的Transformer (big) 模型使用P d r o p = 0.1 P\_{drop} = 0.1Pdrop=0.1，而不是0.3。

  对于基础模型，我们使用的单个模型来自最后5个checkpoints的平均值，这些checkpoints每10分钟保存一次。 对于大型模型，我们对最后20个checkpoints进行了平均。 我们使用beam search，beam大小为4 ，长度惩罚α = 0.6 \[38\]。 这些超参数是在开发集上进行实验后选定的。 在推断时，我们设置最大输出长度为输入长度+50，但在条件允许时会尽早终止\[38\]。

  表2总结了我们的结果，并将我们的翻译质量和训练成本与文献中的其他模型体系结构进行了比较。 我们通过将训练时间、所使用的GPU的数量以及每个GPU的持续单精度浮点能力的估计相乘来估计用于训练模型的浮点运算的数量5 ^55。

### 6.2 Model Variations（模型变体）

  为了评估Transformer不同组件的重要性，我们以不同的方式改变我们的基础模型，观测在开发集newstest2013上英文-德文翻译的性能变化。 我们使用前一节所述的beam search，但没有平均checkpoint。 我们在表中列出这些结果 3.

  在表3的行（A）中，我们改变attention head的数量和attention key和value的维度，保持计算量不变，如3.2.2节所述。 虽然只有一个head attention比最佳设置差0.9 BLEU，但质量也随着head太多而下降。

  在表3行（B）中，我们观察到减小key的大小d k d\_kdk会有损模型质量。 这表明确定兼容性并不容易，并且比点积更复杂的兼容性函数可能更有用。 我们在行（C）和（D）中进一步观察到，如预期的那样，更大的模型更好，并且dropout对避免过度拟合非常有帮助。 在行（E）中，我们用学习到的positional encoding\[9\]来替换我们的正弦位置编码，并观察到与基本模型几乎相同的结果。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191113154248431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vY21s,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191113154304229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vY21s,size_16,color_FFFFFF,t_70)

### 6.3 English Constituency Parsing（英文选区分析）

  为了评估Transformer是否可以扩展到其他任务，我们进行了英语选区解析的实验。这项任务提出特别的挑战：输出受到很强的结构性约束，并且比输入要长很多。 此外，RNN序列到序列模型还没有能够在小数据\[37\]中获得最好的结果。

  我们用d m o d e l d\_{model}dmodel = 1024 在Penn Treebank\[25\]的Wall Street Journal（WSJ）部分训练了一个4层的transformer，约40K个训练句子。 我们还使用更大的高置信度和BerkleyParser语料库，在半监督环境中对其进行了训练，大约17M个句子\[37\]。 我们使用了一个16K词符的词汇表作为WSJ唯一设置，和一个32K词符的词汇表用于半监督设置。

  我们只在开发集的Section 22 上进行了少量的实验来选择dropout、attention 和residual（第5.4节）、learning rates和beam size，所有其他参数从英语到德语的基础翻译模型保持不变。在推断过程中，我们将最大输出长度增加到输入长度+300。 对于WSJ和半监督设置，我们都使用beam size = 21 和α = 0.3 。

  表4中我们的结果表明，尽管缺少特定任务的调优，我们的模型表现得非常好，得到的结果比之前报告的Recurrent Neural Network Grammar \[8\]之外的所有模型都好。

  与RNN序列到序列模型\[37\]相比，即使仅在WSJ训练40K句子组训练时，Transformer也胜过BerkeleyParser \[29\]。

## 7 Conclusion（结论）

  在这项工作中，我们提出了Transformer，第一个完全基于attention的序列转换模型，用multi-headed self-attention取代了encoder-decoder架构中最常用的recurrent layers。

  对于翻译任务，Transformer比基于循环或卷积层的体系结构训练更快。 在WMT 2014英语-德语和WMT 2014英语-法语翻译任务中，我们取得了最好的结果。 在前面的任务中，我们最好的模型甚至胜过以前报道过的所有整合模型。

  我们对基于attention的模型的未来感到兴奋，并计划将它们应用于其他任务。 我们计划将Transformer扩展到除文本之外的涉及输入和输出模式的问题，并研究局部的、受限的attention机制，以有效地处理图像、音频和视频等大型输入和输出。 让生成具有更少的顺序性是我们的另一个研究目标。

  我们用于训练和评估模型的代码可以在[https://github.com/tensorflow/tensor2tensor](https://github.com/tensorflow/tensor2tensor)找到。

  我们十分感谢Nal Kalchbrenner和Stephan Gouws卓有成效的评论、修正和灵感。

  受人本英文能力影响，有些地地方翻译的不是很友好，欢迎同学们及时指正，同时欢迎讨论论文中的相关内容。有需要pdf版的可以留下邮箱，我有时间时会发给你们。