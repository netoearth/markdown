**作者：朱凯 - 机器学习 PAI 团队**

随着深度学习的不断发展，AI 模型结构在快速演化，底层计算硬件技术更是层出不穷，对于广大开发者来说不仅要考虑如何在复杂多变的场景下有效的将算力发挥出来，还要应对计算框架的持续迭代。深度编译器就成了应对以上问题广受关注的技术方向，让用户仅需专注于上层模型开发，降低手工优化性能的人力开发成本，进一步压榨硬件性能空间。阿里云机器学习 PAI 开源了业内较早投入实际业务应用的动态 shape 深度学习编译器 BladeDISC，本文将详解 BladeDISC 的设计原理和应用。

## **BladeDISC 是什么**

BladeDISC 是阿里最新开源的基于 MLIR 的动态 shape 深度学习编译器。

### **主要特性**

-   多款前端框架支持：TensorFlow，PyTorch
-   多后端硬件支持：CUDA，ROCM，x86
-   完备支持动态 shape 语义编译
-   支持推理及训练
-   轻量化 API，对用户通用透明
-   支持插件模式嵌入宿主框架运行，以及独立部署模式

### **开源地址**

[https://github.com/alibaba/BladeDISC](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Falibaba%2FBladeDISC)

### **深度学习编译器的背景**

近几年来，深度学习编译器作为一个较新的技术方向异常地活跃，包括老牌一些的 [TensorFlow XLA](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fwww.tensorflow.org%2Fxla%2F)、[TVM](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Ftvm.ai%2F)、[Tensor Comprehension](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fpytorch.org%2Fblog%2Ftensor-comprehensions%2F)、[Glow](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fengineering.fb.com%2Fml-applications%2Fglow-a-community-driven-approach-to-ai-infrastructure%2F)，到后来呼声很高的 [MLIR](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fmlir.llvm.org%2F) 以及其不同领域的延伸项目 [IREE](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fgoogle.github.io%2Firee%2F)、[mlir-hlo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fgithub.com%2Ftensorflow%2Fmlir-hlo) 等等。能够看到不同的公司、社区在这个领域进行着大量的探索和推进。

### **AI 浪潮与芯片浪潮共同催生 —— 从萌芽之初到蓬勃发展**

深度学习编译器近年来之所以能够受到持续的关注，主要来自于几个方面的原因：

**框架性能优化在模型泛化性方面的需求。**深度学习还在日新月异的发展之中，创新的应用领域不断涌现，复杂多变的场景下如何有效的将硬件的算力充分发挥出来成为了整个 AI 应用链路中非常重要的一环。在早期，神经网络部署的侧重点在于框架和算子库，这部分职责很大程度上由深度学习框架、硬件厂商提供的算子库、以及业务团队的手工优化工作来承担。

![](https://pic4.zhimg.com/80/v2-b8eed1e2d2701fcba108e0dfc1ee7f1b_720w.jpg)

上图将近年的深度学习框架粗略分为三代。一个趋势是在上层的用户 API 层面，这些框架在变得越来越灵活，灵活性变强的同时也为底层性能问题提出了更大的挑战。初代深度学习框架类似 Caffe 用 sequence of layer 的方式描述神经网络结构，第二代类似 TensorFlow 用更细粒度的 graph of operators 描述计算图，到第三代类似 PyTorch，TensorFlow Eager Mode 的动态图。我们可以看到框架越来越灵活，描述能力越来越强，带来的问题是优化底层性能变得越来越困难。业务团队也经常需要补充完成所需要的手工优化，这些工作依赖于特定业务和对底层硬件的理解，耗费人力且难以泛化。而深度学习编译器则是结合编译时图层的优化以及自动或者半自动的代码生成，将手工优化的原理做泛化性的沉淀，以替代纯手工优化带来的各种问题，去解决深度学习框架的灵活性和性能之间的矛盾。

**AI 框架在硬件泛化性方面的需求。**表面上看近些年 AI 发展的有目共睹、方兴未艾，而后台的硬件算力数十年的发展才是催化 AI 繁荣的核心动力。随着晶体管微缩面临的各种物理挑战越来越大，芯片算力的增加越来越难，摩尔定律面临失效，创新体系结构的各种 DSA 芯片迎来了一波热潮，传统的 x86、ARM 等都在不同的领域强化着自己的竞争力。硬件的百花齐放也为 AI 框架的发展带来了新的挑战。

而硬件的创新是一个问题，如何能把硬件的算力在真实的业务场景中发挥出来则是另外一个问题。新的 AI 硬件厂商不得不面临除了要在硬件上创新，还要在软件栈上做重度人力投入的问题。向下如何兼容硬件，成为了如今深度学习框架的核心难点之一，而兼容硬件这件事，则需要由编译器来解决。

**AI 系统平台对前端 AI 框架泛化性方面的需求。**当今主流的深度学习框架包括 Tensoflow、Pytorch、Keras、JAX 等等，几个框架都有各自的优缺点，在上层对用户的接口方面风格各异，却同样面临硬件适配以及充分发挥硬件算力的问题。不同的团队常根据自己的建模场景和使用习惯来选择不同的框架，而云厂商或者平台方的性能优化工具和硬件适配方案则需要同时考虑不同的前端框架，甚至未来框架演进的需求。Google 利用 XLA 同时支持 TensorFlow 和 JAX，同时其它开源社区还演进出了 Torch\_XLA，Torch-MLIR 这样的接入方案，这些接入方案目前虽然在易用性和成熟程度等方面还存在一些问题，却反应出 AI 系统层的工作对前端 AI 框架泛化性方面的需求和技术趋势。

### **什么是深度学习编译器**

![](https://pic1.zhimg.com/80/v2-c8bd08a5d757e0da65c6089cdfb3d640_720w.jpg)

传统编译器是以高层语言作为输入，避免用户直接去写机器码，而用相对灵活高效的语言来工作，并在编译的过程中引入优化来解决高层语言引入的性能问题，平衡开发效率与性能之间的矛盾。而深度学习编译器的作用相仿，其输入是比较灵活的，具备较高抽象度的计算图描述，输出包括 CPU、GPU 及其他异构硬件平台上的底层机器码及执行引擎。

传统编译器的使命之一是减轻编程者的压力。作为编译器的输入的高级语言往往更多地是描述一个逻辑，为了便利编程者，高级语言的描述会更加抽象和灵活，至于这个逻辑在机器上是否能够高效的执行，往往是考验编译器的一个重要指标。深度学习作为一个近几年发展异常快速的应用领域，它的性能优化至关重要，并且同样存在高层描述的灵活性和抽象性与底层计算性能之间的矛盾，因此专门针对深度学习的编译器出现了。而传统编译器的另外一个重要使命是，需要保证编程者输入的高层语言能够执行在不同体系结构和指令集的硬件计算单元上，这一点也同样反应在深度学习编译器上。面对一个新的硬件设备，人工的话不太可能有精力对那么多目标硬件重新手写一个框架所需要的全部算子实现，深度学习编译器提供中间层的 IR，将顶层框架的模型流图转化成中间层表示 IR，在中间层 IR 上进行通用的图层优化，而在后端将优化后的 IR 通用性的生成各个目标平台的机器码。

深度学习编译器的目标是针对 AI 计算任务，以通用编译器的方式完成性能优化和硬件适配。让用户可以专注于上层模型开发，降低用户手工优化性能的人力开发成本，进一步压榨硬件性能空间。

### **距离大规模应用面临的瓶颈问题**

深度学习编译器发展到今天，虽然在目标和技术架构方面与传统编译器有颇多相似之处，且在技术方向上表现出了良好的潜力，然而目前的实际应用范围却仍然距离传统编译器有一定的差距，主要难点包括：

**易用性：**深度学习编译器的初衷是希望简化手工优化性能和适配硬件的人力成本。然而在现阶段，大规模部署应用深度学习编译器的挑战仍然较大，能够将编译器用好的门槛较高，造成这个现象的主要原因包括：

-   与前端框架对接的问题。不同框架对深度学习任务的抽象描述和 API 接口各有不同，语义和机制上有各自的特点，且作为编译器输入的前端框架的算子类型数量呈开放性状态。如何在不保证所有算子被完整支持的情况下透明化的支持用户的计算图描述，是深度学习编译器能够易于为用户所广泛使用的重要因素之一。
-   动态 shape 问题和动态计算图问题。现阶段主流的深度学习编译器主要针对特定的静态 shape 输入完成编译，此外对包含 control flow 语义的动态计算图只能提供有限的支持或者完全不能够支持。而 AI 的应用场景却恰恰存在大量这一类的任务需求。这时只能人工将计算图改写为静态或者半静态的计算图，或者想办法将适合编译器的部分子图提取出来交给编译器。这无疑加重了应用深度学习编译器时的工程负担。更严重的问题是，很多任务类型并不能通过人工的改写来静态化，这导致这些情况下编译器完全无法实际应用。
-   编译开销问题。作为性能优化工具的深度学习编译器只有在其编译开销对比带来的性能收益有足够优势的情况下才真正具有实用价值。部分应用场景下对于编译开销的要求较高，例如普通规模的需要几天时间完成训练任务有可能无法接受几个小时的编译开销。对于应用工程师而言，使用编译器的情况下不能快速的完成模型的调试，也增加了开发和部署的难度和负担。
-   对用户透明性问题。部分 AI 编译器并非完全自动的编译工具，其性能表现比较依赖于用户提供的高层抽象的实现模版。主要是为算子开发工程师提供效率工具，降低用户人工调优各种算子实现的人力成本。但这也对使用者的算子开发经验和对硬件体系结构的熟悉程度提出了比较高的要求。此外，对于新硬件的软件开发者来说，现有的抽象却又常常无法足够描述创新的硬件体系结构上所需要的算子实现。需要对编译器架构足够熟悉的情况下对其进行二次开发甚至架构上的重构，门槛及开发负担仍然很高。

**鲁棒性：**目前主流的几个 AI 编译器项目大多数都还处于偏实验性质的产品，但产品的成熟度距离工业级应用有较大的差距。这里的鲁棒性包含是否能够顺利完成输入计算图的编译，计算结果的正确性，以及性能上避免 coner case 下的极端 badcase 等。

**性能问题：**编译器的优化本质上是将人工的优化方法，或者人力不易探究到的优化方法通过泛化性的沉淀和抽象，以有限的编译开销来替代手工优化的人力成本。然而如何沉淀和抽象的方法学是整个链路内最为本质也是最难的问题。深度学习编译器只有在性能上真正能够代替或者超过人工优化，或者真正能够起到大幅度降低人力成本作用的情况下才能真正发挥它的价值。

然而达到这一目标却并非易事，深度学习任务大都是 tensor 级别的计算，对于并行任务的拆分方式有很高的要求，但如何将手工的优化泛化性的沉淀在编译器技术内，避免编译开销爆炸以及分层之后不同层级之间优化的联动，仍然有更多的未知需要去探索和挖掘。这也成为以 MLIR 框架为代表的下一代深度学习编译器需要着重思考和解决的问题。

## **BladeDISC 主要技术特点**

项目最早启动的初衷是为了解决 XLA 和 TVM 当前版本的静态 shape 限制，内部命名为 DISC (DynamIc Shape Compiler)，希望打造一款能够在实际业务中使用的完备支持动态 shape 语义的深度学习编译器。

从团队四年前启动深度学习编译器方向的工作以来，动态 shape 问题一直是阻碍实际业务落地的严重问题之一。彼时，包括 XLA 在内的主流深度学习框架，都是基于静态 shape 语义的编译器框架。典型的方案是需要用户指定输入的 shape，或是由编译器在运行时捕捉待编译子图的实际输入 shape 组合，并且为每一个输入 shape 组合生成一份编译结果。

静态 shape 编译器的优势显而易见，编译期完全已知静态 shape 信息的情况下，Compiler 可以作出更好的优化决策并得到更好的 CodeGen 性能，同时也能够得到更好的显存 / 内存优化 plan 和调度执行 plan。然而，其缺点也十分明显，具体包括：

-   大幅增加编译开销。引入离线编译预热过程，大幅增加推理任务部署过程复杂性；训练迭代速度不稳定甚至整体训练时间负优化。
-   部分业务场景 shape 变化范围趋于无穷的，导致编译缓存永远无法收敛，方案不可用。
-   内存显存占用的增加。编译缓存额外占用的内存显存，经常导致实际部署环境下的内存 / 显存 OOM，直接阻碍业务的实际落地。
-   人工 padding 为静态 shape 等缓解性方案对用户不友好，大幅降低应用的通用性和透明性，影响迭代效率。

![](https://pic4.zhimg.com/80/v2-5254b934007c81b01d79fdde1261064f_720w.jpg)

在 2020 年夏天，DISC 完成了仅支持 TensorFlow 前端以及 Nvidia GPU 后端的初版，并且正式在阿里内部上线投入实际应用。最早在几个受困于动态 shape 问题已久的业务场景上投入使用，并且得到了预期中的效果。即在一次编译且不需要用户对计算图做特殊处理的情况下，完备支持动态 shape 语义，且性能几乎与静态 shape 编译器持平。对比 TensorRT 等基于手工算子库为主的优化框架，DISC 基于编译器自动 codegen 的技术架构在经常为非标准开源模型的实际业务上获得了明显的性能和易用性优势。

从 2020 年第二季度开始至今，DISC 持续投入研发力量，针对前文提到的从云端平台方视角看到的深度学习编译器距离大规模部署和应用的几个瓶颈问题，在性能、算子覆盖率和鲁棒性、CPU 及新硬件支持、前端框架支持等方面逐渐完善。目前在场景覆盖能力和性能等方面，已经逐渐替换掉团队过往基于 XLA 和 TVM 等静态 shape 框架上的工作，成为 PAI-Blade 支持阿里内部及阿里云外部业务的主要优化手段。2021 年后，DISC 在 CPU 及 GPGPU 体系结构的后端硬件上的性能有了显著的提升，同时在新硬件的支持上面投入了更多的技术力量。2021 年底，为了吸引更多的技术交流和合作共建需要，以及更大范围的用户反馈，正式更名为 BladeDISC 并完成了初版开源。

## BladeDISC 关键技术

BladeDISC 的整体架构，及其在阿里云相关产品中的上下文关系如下图所示：

![](https://pic2.zhimg.com/80/v2-f4bb1bb1892187f6eb920cca480b1b7d_720w.jpg)

### **MLIR 基础架构**

MLIR 是由 Google 在 2019 年发起的项目，MLIR 的核心是一套灵活的多层 IR 基础设施和编译器实用工具库，深受 LLVM 的影响，并重用其许多优秀理念。这里我们选择基于 MLIR 的主要原因包括其比较丰富的基础设施支持，方便扩展的模块化设计架构以及 MLIR 较强的胶水能力。

### **动态 shape 编译**

![](https://pic1.zhimg.com/80/v2-621513daf77c53334cad1256408ef314_720w.jpg)

上图为 BladeDISC 的主体 Pass Pipeline 设计。对比目前主流的深度学习编译器项目，主要技术特点如下：

**图层 IR 设计**

BladeDISC 选择基于 HLO 作为核心图层 IR 来接入不同的前端框架，但是 HLO 是原本为 XLA 设计的纯静态 shape 语义的 IR。静态场景下，HLO IR 中的 shape 表达会被静态化，所有的 shape 计算会被固化为编译时常量保留在编译结果中；而在动态 shape 场景下，IR 本身需要有足够的能力表达 shape 计算和动态 shape 信息的传递。BladeDISC 从项目建立开始一直与 MHLO 社区保持紧密的合作，在 XLA 的 HLO IR 基础上，扩展了一套具有完备动态 shape 表达能力的 IR，并增加了相应的基础设施以及前端框架的算子转换逻辑。这部分实现目前已经完整 upstream 至 MHLO 社区，确保后续其它 MHLO 相关项目中 IR 的一致性。

**运行时 Shape 计算、存储管理和 Kernel 调度**

动态 shape 编译的主要挑战来自于需要在静态的编译过程中能够处理动态的计算图语义。为完备支持动态 shape，编译结果需要能够在运行时做实时的 shape 推导计算，不仅要为数据计算，同时也需要为 shape 计算做代码生成。计算后的 shape 信息用于做内存 / 显存管理，以及 kernel 调度时的参数选择等等。BladeDISC 的 pass pipeline 的设计充分考虑了上述动态 shape 语义支持的需求，采用了 host-device 联合 codegen 的方案。以 GPU Backend 为例，包括 shape 计算、内存 / 显存申请释放、硬件管理、kernel launch 运行时流程全部为自动代码生成，以期得到完备的动态 shape 端到端支持方案和更为极致的整体性能。

**动态 shape 下的性能问题**

在 shape 未知或者部分未知的情况下，深度学习编译器在性能上面临的挑战被进一步放大。在大多数主流硬件 backend 上，BladeDISC 采用区分计算密集型部分和访存密集型部分的策略，以期在性能与复杂性和编译开销之间获取更好的平衡。

对于计算密集型部分，不同的 shape 要求更加精细的 schedule 实现来获得更好的性能，pass pipeline 在设计上的主要考虑是需要支持在运行时根据不同的具体 shape 选择合适的算子库实现，以及处理动态 shape 语义下的 layout 问题。

而访存密集型部分的自动算子融合作为深度学习编译器主要的性能收益来源之一，同样面临 shape 未知情况下在性能上的挑战。许多静态 shape 语义下比较确定性的问题，例如指令层的向量化，codegen 模版选择，是否需要 implicit broadcast 等等在动态 shape 场景下都会面临更大的复杂性。针对这些方面的问题，BladeDISC 选择将部分的优化决策从编译时下沉到运行时。即在编译期根据一定的规则生成多个版本的 kernel 实现，在运行时根据实际 shape 自动选择最优的实现。这一机制被称作 speculation，在 BladeDISC 内基于 host-device 的联合代码生成来实现。此外，在编译期没有具体 shape 数值的情况下，会很容易在各个层级丢失掉大量的优化机会，从图层的线性代数简化、fusion 决策到指令层级的 CSE、常数折叠等。BladeDISC 在 IR 及 pass pipeline 的设计过程中着重设计了 shape constraint 在 IR 中的抽象和在 pass pipeline 中的使用，例如编译期未知的不同 dimension size 之间的约束关系等。在优化整体性能方面起到了比较明显的作用，保证能够足够接近甚至超过静态 shape 编译器的性能结果。

**大颗粒度算子融合**

团队在开启 BladeDISC 项目之前，曾经基于静态 shape 编译器在大颗粒度算子融合及自动代码生成方面有过若干探索 \[3\]\[4\]，其基本思想可以概括为借助于 GPU 硬件中低访存开销的 shared memory 或 CPU 中低访存开销的 Memory Cache，将不同 schedule 的计算子图缝合进同一个 kernel 内，实现多个 parallel loop 复合，这种 codegen 方法称之为 fusion-stitching。这种访存密集型子图的自动代码生成打破了常规的 loop fusion，input/output fusion 对 fusion 颗粒度的限制。在保证代码生成质量的同时，大幅增加 fusion 颗粒度，同时避免复杂性及编译开销爆炸。且整个过程完全对用户透明，无需人工指定 schedule 描述。

![](https://pic4.zhimg.com/80/v2-17b6065303bbfb6de91abb2620cc5477_720w.jpg)

在动态 shape 语义下实现 fusion-stitching 对比静态 shape 语义下同样需要处理更大的复杂性，动态 shape 语义下的 shape constraint 抽象一定程度上简化了这一复杂性，使整体性能进一步接近甚至超过手工算子实现。

**多前端框架支持**

AICompiler 框架在设计时也包含了扩展支持不同前端框架的考虑。PyTorch 侧通过实现一个轻量的 Converter 将 TorchScript 转换为 DHLO IR 实现了对 PyTorch 推理作业的覆盖。MLIR 相对完备的 IR 基础设施也为 Converter 的实现提供了便利。BladeDISC 包含 Compiler 以及适配不同前端框架的 Bridge 侧两部分。其中 Bridge 进一步分为宿主框架内的图层 pass 以及运行时 Op 两部分，以插件的方式接入宿主框架。这种工作方式使 BladeDISC 可以透明化的支持前端计算图，可以适配用户各种版本的宿主框架。

**运行时环境适配**

为将编译的结果能够配合 TensorFlow/PyTorch 等宿主在各自的运行环境中执行起来，以及管理运行时 IR 层不易表达的状态信息等等，我们为不同的运行时环境实现了一套统一的 Compiler 架构，并引入了运行时抽象层，即 RAL (Runtime Abstraction Layer) 层。

RAL 实现了多种运行环境的适配支持，用户可以根据需要进行选择，具体包括：

-   全图编译，独立运行。当整个计算图都支持编译时，RAL 提供了一套简易的 runtime 以及在此之上 RAL Driver 的实现，使得 compiler 编译出来结果可以脱离框架直接运行，减少框架 overhead。
-   TF 中子图编译运行。
-   Pytorch 中子图编译运行。

以上环境中在诸如资源管理，API 语义等上存在差异，RAL 通过抽象出一套最小集合的 API ，并清晰的定义出它们的语义，将编译器与运行时隔离开来，来达到在不同的环境中都能够执行编译出来的结果的目的。此外 RAL 层实现了无状态编译，解决了计算图的编译之后，编译的结果可能被多次执行时的状态信息处理问题。一方面简化了代码生成的复杂度，另一方面也更容易支持多线程并发执行（比如推理）的场景，同时在错误处理，回滚方面也更加容易支持。

## **应用场景**

BladeDISC 的典型应用场景可以不太严格的分为两类：其一是在主流的硬件平台上（包括 Nvidia GPU，x86 CPU 等）上作为通用、透明的性能优化工具，降低用户部署 AI 作业的人力负担，提高模型迭代效率；另一个重要的应用场景是帮助新硬件做 AI 场景的适配和接入支持。

目前 BladeDISC 已经广泛应用在阿里内部和阿里云上外部用户的多个不同应用场景下，覆盖模型类型涉及 NLP、机器翻译、语音类 ASR、语音 TTS、图像检测、识别、AI for science 等等多种典型 AI 应用；覆盖行业包括互联网、电商、自动驾驶、安全行业、在线娱乐、医疗和生物等等。

在推理场景下，BladeDISC 与 TensorRT 等厂商提供的推理优化工具形成良好的技术互补，其主要差异性优势包括：

-   应对动态 shape 业务完备的动态 shape 语义支持
-   基于 compiler based 的技术路径的模型泛化性在非标准模型上的性能优势
-   更为灵活的部署模式选择，以插件形式支持前端框架的透明性优势

下图为 Nvidia T4 硬件上几个真实的业务例子的性能收益数字：

![](https://pic3.zhimg.com/80/v2-a8a7f97130a3e65b7cd1902114b7f9ee_720w.jpg)

在新硬件支持方面，目前普遍的情况是除了积累比较深厚的 Nvidia 等头部厂商之外，包括 ROCM 等其它 GPGPU 硬件普遍存在的情况是硬件的指标已经具备相当的竞争力，但厂商受制于 AI 软件栈上的积累相对较少，普遍存在硬件算力无法发挥出来导致硬件落地应用困难的问题。如前文所述，基于编译器的技术路径下天然对于硬件的后端具备一定的泛化能力，且与硬件厂商的技术储备形成比较强的互补。BladeDISC 目前在 GPGPU 和通用 CPU 体系结构上的储备相对比较成熟。以 GPGPU 为例，在 Nvidia GPU 上的绝大部分技术栈可以迁移至海光 DCU 和 AMD GPU 等体系结构相近的硬件上。BladeDISC 较强的硬件泛化能力配合硬件本身较强的通用性，很好的解决了新硬件适配的性能和可用性问题。

下图为海光 DCU 上几个真实业务例子上的性能数字：

<table><tbody><tr><td><span>某识别类模型</span></td><td>推理</td><td><span>不同 batchsize 下 2.21X ～ 2.31X</span></td></tr><tr><td><span>某检测类模型 A&nbsp; &nbsp;</span></td><td>推理</td><td><span>不同 batchsize 下 1.73X ～ 2.1X</span></td></tr><tr><td><span>某检测类模型 B</span></td><td>推理</td><td><span>不同 batchsize 下 1.04X ～ 1.59X</span></td></tr><tr><td><span>某分子动力学模型</span></td><td>训练</td><td><span>2.0X</span></td></tr></tbody></table>

## **开源生态：构想和未来**

我们决定建设开源生态主要有如下的考虑：

BladeDISC 发源于阿里云计算平台团队的业务需求，在开发过程中与 MLIR/MHLO/IREE 等社区同行之间的讨论和交流给了我们很好的输入和借鉴。在我们自身随着业务需求的迭代逐渐完善的同时，也希望能够开源给社区，在目前整个 AI 编译器领域实验性项目居多，偏实用性强的产品偏少，且不同技术栈之间的工作相对碎片化的情况下，希望能够将自身的经验和理解也同样回馈给社区，希望和深度学习编译器的开发者和 AI System 的从业者之间有更多更好的交流和共建，为这个行业贡献我们的技术力量。

我们希望能够借助开源的工作，收到更多真实业务场景下的用户反馈，以帮助我们持续完善和迭代，并为后续的工作投入方向提供输入。

后续我们计划以两个月为单位定期发布 Release 版本。BladeDISC 近期的 Roadmap 如下：

-   持续的鲁棒性及性能改进
-   x86 后端补齐计算密集型算子的支持，端到端完整开源 x86 后端的支持
-   GPGPU 上基于 Stitching 的大颗粒度自动代码生成
-   AMD rocm GPU 后端的支持
-   PyTorch 训练场景的支持

此外，在中长期，我们在下面几个探索性的方向上会持续投入精力，也欢迎各种维度的反馈和改进建议以及技术讨论，同时我们十分欢迎和期待对开源社区建设感兴趣的同行一起参与共建。

-   更多新硬件体系结构的支持和适配，以及新硬件体系结构下软硬件协同方法学的沉淀
-   计算密集型算子自动代码生成和动态 shape 语义下全局 layout 优化的探索
-   稀疏子图的优化探索
-   动态 shape 语义下运行时调度策略、内存 / 显存优化等方面的探索
-   模型压缩与编译优化联合的技术探索
-   图神经网络等更多 AI 作业类型的支持和优化等

**BaldeDISC 项目地址**：[https://github.com/alibaba/BladeDISC](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Falibaba%2FBladeDISC)

**更多开源项目合集：**[https://www.aliyun.com/activity/bigdata/opensource\_bigdata\_\_ai](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.aliyun.com%2Factivity%2Fbigdata%2Fopensource_bigdata__ai)

## **参考文献**

1. ["DISC: A Dynamic Shape Compiler for Machine Learning Workloads", Kai Zhu, Wenyi Zhao, Zhen Zheng, Tianyou Guo, Pengzhan Zhao, Feiwen Zhu, Junjie Bai, Jun Yang, Xiaoyong Liu, Lansong Diao, Wei Lin](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Farxiv.org%2Fabs%2F2103.05288)

2\. Presentations on MLIR Developers' Weekly Conference: [1](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fdrive.google.com%2Ffile%2Fd%2F1_uEISlV5MUWdG9faKAdKlCWnPtGjRC-D%2Fview%253Fspm%253Data.13261165.0.0.2ef0acc3KGryt4), [2](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fllvm.discourse.group%2Ft%2Fupdates-on-mlir-based-dynamic-shape-compiler%2F2384)

3\. "AStitch: Enabling A New Multi-Dimensional Optimization Space for Memory-Intensive ML Training and Inference on Modern SIMT Architectures", Zhen Zheng, Xuanda Yang, Pengzhan Zhao, Guoping Long, Kai Zhu, Feiwen Zhu, Wenyi Zhao, Xiaoyong Liu, Jun Yang, Jidong Zhai, Shuaiwen Leon Song, and Wei Lin. The 27th ACM International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS), 2022. \[to appear\]

4. ["FusionStitching: Boosting Memory Intensive Computations for Deep Learning Workloads", Zhen Zheng, Pengzhan Zhao, Guoping Long, Feiwen Zhu, Kai Zhu, Wenyi Zhao, Lansong Diao, Jun Yang, and Wei Lin. arXiv preprint](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Farxiv.org%2Fabs%2F2009.10924)