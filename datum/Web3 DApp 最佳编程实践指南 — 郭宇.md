自宣布进入创业间隔年以来，[CodeforDAO](https://mobile.twitter.com/codefordao)([GitHub](https://github.com/CodeforDAO/contracts)) 与 [Checks Finance](https://checks.finance/)([@checksfinance](https://twitter.com/checksfinance))两个项目进入了密集而紧张的迭代周期，在合约编写，单元测试，工作流自动化，前端与客户端方面都遇到了较多问题，对此，我总结出了一些经验。当前这两个项目还有大量细节等待优化，尚未正式 landing，我认为将开发过程中的经验和总结与大家进行分享，能帮助更多工程师转向 Web3，也有助于项目的长远发展。

这篇文章将会涉及到开发一个 DApp 所涵盖的几乎所有方面内容，因此，它会非常冗长繁琐，如果你对某一方面特别感兴趣，我建议你可以通过下边这个目录直接跳去感兴趣的章节阅读。另外，这篇文章并不是 Step by Step 的代码教学范例，因此，跳跃章节阅读并不会影响体验。

> 本文中提到的所有项目均列在我的 GitHub Star 清单中，可以在这里统一查阅：

___

1.  认识 DApp 技术栈
2.  智能合约编码
3.  开发工作流与单元测试
4.  前端与客户端开发
5.  开发、测试与生产环境调试
6.  服务端编码与集成
7.  合约部署方案 L1s & L2
8.  去中心化储存方案
9.  附录

___

## 1\. 认识 DApp 技术栈

与传统的 App（包括 Web App 与 Mobile App）最大的不同点在于，DApp 的大量功能依赖直接与智能合约（以下简称合约）进行交互。我们无法直接使用前端代码调用合约，因此，在开发 DApp 之前，我们必须理解这一技术栈中存在哪些技术细节以及它们分别扮演何种角色。

1.  **智能合约**：通常指代运行在 EVM 兼容网络中的 [Solidity](https://docs.soliditylang.org/) 或其他合约语言代码，他们负责与用户交易我们发行的资产并储存 DApp 的链上状态。
2.  **DApp**：整合合约接口以及其他功能的应用程序界面，目前，它们大部分是 Web App，你可以用流行的框架例如 [React](https://reactjs.org/)/[Vue](https://cn.vuejs.org/index.html) 来进行编写。
3.  **Provider/Signer**: 这是一个 DApp 架构中特殊的角色，它负责与区块链进行通信，并进行合约的读/写操作。[Metamask](https://metamask.io/) 是一个流行的 InjectProvider（Web3Provider）你也可以使用其他 JSON-RPC Provider 与区块链进行通信。
4.  **Relay**: 这个角色隐藏在 Provider/Signer 之后，是真正负责我们与区块链的某一个节点同步状态的服务器集群，它保存了所有账本（全节点）它通常是 [Infura](https://infura.io/)、[Alchemy](https://www.alchemy.com/)、[Quicknode](https://www.quicknode.com/)、[Moralis](https://moralis.io/) 或者 [Pocket](https://www.pokt.network/) 提供的服务。
5.  **服务端**（可选）：大部分 DApp 仍然有他们的服务端逻辑，这意味着，你需要自己搭建服务环境，或使用流行的 BasS/FaaS 服务，你可以使用深度整合区块链的 Moralis 来完成服务端的开发，也可以使用成熟的 [Firebase](https://firebase.google.com/docs?hl=zh-cn) 体系。当然，你也可以挑战完全不依赖服务端的方式来构建 DApp，就像 Uniswap 所做的那样。

现在，我们知道编写一个 DApp 大概需要哪些领域的知识，如果你已经决定迈向下一代互联网并打算闯荡一番，我会在接下来的内容中仔细介绍这些角色分别需要理解哪些编程语言，框架和库。

___

让我们先进入最重要的部分，**智能合约**。大量程序员望而却步的重要门槛是，他们认为智能合约需要学习一门新的编程语言，**Solidity**，这毫无疑问，我非常推荐入门 Web3 的程序员—— 无论你是从哪一个软件开发领域转型而来 —— 从 Solidity 入手学习 DApp 开发。

在智能合约的编码方面，我们目前有许多工具，但认识和理解 Solidity 非常有必要，大量的已经存在的，和流行的合约都使用它进行编码，因此，学习 Solidity 不但有助于帮助你理解区块链开发的基本知识和概念，还能让你在许多优秀的开发者已有的卓越工程上快速起步。

就编程语言而言，在目前的 EVM 兼容链上，你可以使用 [Solidity](https://docs.soliditylang.org/en/v0.8.13/) 或 [Vyper](https://vyper.readthedocs.io/) 进行开发，在其他 L1s 区块链上，例如 [Solana](https://solana.com/zh)，你可以使用 [Rust](https://www.rust-lang.org/) 来进行合约的开发；在 Layer2 方案 [StarkNet](https://starkware.co/starknet/) 中，你可以使用 [Cairo](https://starkware.co/cairo/) 来进行开发；在 Arweave 储存网络中，也存在着类似 [3em](https://github.com/three-em/3em) 这样的运行环境支持你使用 JavaScript 来编写合约。

在这些百花齐放的方案中，实际上存在着两种不同的合约运行环境，EVM 或非 EVM 方案，前者的代码都会被编译成 EVM bytecode，而后者则会采用各种各样的 runtime，各显神通。

这篇文章不会在合约编程语言上讨论太多，我认为，我们目前正处于合约 runtime 的战国时代，没有人能断言哪种合约编程语言的地位会成为 Web 世界的 JavaScript。但对于智能合约编码来说，我们必须要了解和熟悉 **Solidity**，这是毫无疑问的。

关于 Solidity，我推荐你从 [Solidity by Example](https://solidity-by-example.org/) 教程开始学习：

这一教程没有繁琐的语法介绍，而根据范例帮助读者掌握基本知识，因此，完成这一教程大约只需要**不到一个工作日**。Solidity 并不是一个特别复杂的语言，在使用它时，我们可以逐步理解每一项语句的语义，我推荐你设置好编码环境后按照网站上的范例来进行实践。当你已经掌握所有范例的写法之后，可以打开 Solidity 语言[官方文档](https://docs.soliditylang.org/en/v0.8.13/)（[中文](https://solidity-cn.readthedocs.io/zh/develop/)）对照编码中的错误来进行针对性的学习：

___

理解并掌握智能合约后，我们可以进入 DApp 的编码，这是许多互联网行业从业者的强项，我不会在此赘述关于前端编码的经验，如上所述，我们可以使用流行的前端框架，例如 React 或者 Vue 来进行 DApp 的编码。毫无疑问，你会需要一些前端的技术栈知识，主要是 JavaScript 与 CSS。

在此，我想向大家推荐一些优秀的前端库，使用这些代码库来进行合约交互，会使我们的开发效率事半功倍。

以 React 为例，我们可以使用 [wagmi](https://wagmi.sh/) 来帮助我们更好的操作合约，它集成了大量基础但够用的 hooks，并提供了与外部 `Provider/Signer` 交互的快捷函数。与此同时，[wgami](https://wagmi.sh/) 没有过多的外部依赖，它的核心依赖只有 [ethers.js](https://docs.ethers.io/v5/)

如果你不是一个框架爱好者，想要从零开始构建应用程序，不可避免的，你需要使用 [ethers.js](https://docs.ethers.io/v5/) 或者 [web3.js](https://web3js.readthedocs.io/en/v1.7.3/) 来进行基本操作。从我自己的使用经验来看，我更推荐 ethers.js

一般来说，我们并不需要其他的库为我们提供专门的 `Provider/Signer` 支持，如果你打算支持更多复杂的 Provider，或者同时支持多网络 Provider/Signer 的读写功能，类似 [Apeboard](https://apeboard.finance/) 为它的用户提供跨区块链的数据展现，可以参考 [react-web3](https://github.com/NoahZinsmeister/web3-react) 或者 [w3modal](https://github.com/Web3Modal/web3modal) 两个流行的模块，这些模块提供了一些好用的功能，但他们的设计不够解耦，有时会带来不必要的 bug，对此，我保持谨慎推荐。

进一步，如果你想不想支持外部 `Provider/Signer`，而为自己的用户构建一个 Web 钱包，你可以使用 ethers.js 从零开始构建。

如果你想为用户提供一个 onboard 体验更好（但更不去中心化）的托管钱包系统，让他们可以从普通的账户密码或者社交网络账户来登录你的 DApp，可以选择采用 [Web3Auth](https://web3auth.io/) 或 [MagicLink](https://magic.link/) 的方案。托管钱包系统是一个非常大的话题，感兴趣的读者可以参考上述两个解决方案自行研究。

___

在理解合约以及 DApp 使用何种方式与区块链进行交互后，开发者很快会意识到，我们并没有通过在本地建立一个节点的方式来与区块链进行操作。如果你在本地部署过 [IPFS](https://ipfs.io/)，你会很快发现它会默认在本地同步节点，就像 BT 下载软件那样。这是否意味着我们的 DApp 不够「去中心化」呢？

实际上，仍然有大量的软件基于本地的全节点来进行交互，只是，对于大部分开发者而言，他们放弃了这样的权利，而转而使用更便利的 **Relay Network** 与区块链进行通信，通过这种方式，我们节省了部署成本，并且不再需要维护节点的状态缓存，对于快速构建 DApp 来说，选择一个靠谱的 Relay，是无可非议的方案。

使用 Relay Network 不需要特殊的知识，在前端，我们使用上述提及的代码库（ethers.js 或者 web3.js）与 Relay 进行交互；在服务端，如果你使用 Node 运行环境，也可以直接拷贝前端的代码来使用。如果你使用其他的运行环境，你可能会需要一些特定的 JSON-RPC 函数包装，以访问这些 Relay。

**[Infura](https://infura.io/)** 是世界上最早和最大的以太坊 Relay Network，它提供一些公开的 Gateway 节点，但一般来说，我们需要获取属于自己的 DApp **Access Key** 并为这些访问权限设置 origin 和 IP 限制，以提升使用我们自己的 DApp 用户的访问速度体验。Infura 目前支持 ETH，ETH2 网络，以及 IPFS 和 Filecoin 两个分布式储存方案。

**[Alchemy](https://www.alchemy.com/)** 也是一个非常流行的 Relay Network，它在 Infura 的功能上更近一步，为开发者提供了相当多实用的功能，例如调试工具，区块状态推送与丰富的 Webhooks。从某种意义上说，Alchemy 不是一个单纯的 Relay Network，它更像是一个 SaaS 服务，它提供了丰富的自定义 JSON-RPC 方法，实际上，我们的函数库与它的缓存网络进行交互，而不是直接与区块链节点进行交互，这在很大程度上提升了视图（view）方法的访问速度，但依赖 Alchemy 独有的 JSON-RPC 方法，也让 DApp 变得更加中心化了。

[

![Alchemy | web3 development platform | Blockchain API and Node Service | Ethereum, Polygon, Arbitrum, Optimism, Flow and More Blockchains](https://static.alchemyapi.io/images/marketing/alchemy-banner.jpg)

Alchemy | web3 development platform | Blockchain API and Node Service | Ethereum, Polygon, Arbitrum, Optimism, Flow and More Blockchains

Whether you’re a beginner developer, startup, web3 market leader, or a large enterprise, Alchemy makes multichain web3 development easy with reliable and scalable node infrastructure, enhanced APIs, and developer tools. Get started for free!





](https://www.alchemy.com/)

我不会在这里评判去中心化的「道德问题」，各位读者可以根据自己的开发时间周期，风险偏好和使用习惯来决定何种服务适合自己，并为自己的客户与用户提供更好的服务。

在 Relay Network 方面，我想再推荐一个服务：

[Moralis](https://moralis.io/) 集成了许多 FaaS 的功能到他们的 Relay Network 中，这使得你可以快速在服务端访问区块链的状态，而不需要反复调用第三方网络的 API，这是一个非常有趣而实用的方案，他们的定位是 Web3 的 Firebase，我希望他们能够将软件质量和可用性真正提升到 Firebase 的水平，那这就会是一件非常棒的事儿。

在本文编写的过程中，我得知 Google Cloud Platform 也正在组建他们的 [Web3 团队](https://www.cnbc.com/2022/05/06/googles-cloud-group-forms-web3-product-and-engineering-team.html)，这意味着我们有可能在不久的将来能在 [Firebase](https://firebase.google.com/?hl=zh-cn) 或者 GAE 服务上使用到 Google 的 Relay 服务，我们可以保持适当的关注。

___

服务端方面，你可以使用任何你喜欢的编程语言，运行环境和软件架构，没有什么特殊的限制，只要保证你选择的技术栈能和本地节点或者（通常是）Relay Network 进行交互即可。

一般来说，我会选择 Node 运行环境。说些题外话，由于大部分合约使用 NPM 来进行包管理，并使用 [hardhat](https://hardhat.org/) 来做编译和测试工作流，使用 JavaScript 已经成为智能合约编码中必不可少的一个环节。既然如此，在服务端同时使用 JavaScript 语言有助于我们复用代码，留出更多的时间享受人生。

编写服务端并不意味着我们需要做完所有事，通常，我们使用 DApp 的服务端代码来储存没必要储存在合约中的「链下状态」。在合约中储存数据是十分昂贵的选择（至少目前看来）这种昂贵不仅涉及到我们部署合约中产生的费用，还涉及到每一次修改状态的函数请求带来的，用户需要付出的 gas 成本。所以，大部分时候，我们会使用自己的服务端来储存这些「链下状态」

使用一个健壮的 FaaS 对许多工程师来说是简单而且实用的选择，我推荐 Firebase，如果你想体验深度集成区块链的 FaaS，也可以参考上述提及的 Moralis。

我选择 Firebase 的主要原因是他们提供成本低廉，服务完善和稳定的健壮 API，同时，他们针对开发者开发了功能齐全的本地模拟测试套件，这会节省我们相当多的时间。

FaaS 在市面上有太多可选的方案，你可以依赖一个全功能 FaaS，也可以将自己为数不多的「链下状态」储存在 headless CMS 当中，例如 Vercel 或者 Netlify。

或者，如果你希望自己搭建 FaaS 服务器，以获得更完善的控制与更低的成本，我向你推荐一些 Firebase 的开源替代品，例如 Supabase:

___

## 2\. 智能合约编码

在这一章节，我们会从 Solidity 语言入手，理解编写一个智能合约与传统的应用软件或界面有何不同，你可以使用上一章节提到的其他智能合约编程语言，但本章节将使用 Solidity（以下简称 Sol） 作为范例阐述智能合约编码中应当注意的问题。

在此，我不会逐行逐句解释 Sol 语言的语义细节，因此，阅读这一章要求你有起码的 Sol 语言知识。我建议，在此之前，请参考并读完所有的 [Solidity Examples](https://solidity-by-example.org/)：

### 2.1 合约特征

**事务性**：我们可以将区块链看成是一个事务性数据库，这意味着，要么我们在合约中编写的函数全部被执行，状态依次被修改，要么，所有的状态都会回滚到当初未曾被修改的样子。这意味着，我们在对智能合约进行编码的过程中，要十分注意函数 API 的设计，在具体的函数中，不应当对参数进行重载。同时，也意味着我们在进行错误处理时要十分小心。

**错误处理**：我们可以选择两种常用的错误处理方式，`require(condition, ERR_MESSAGE)` 或者 `revert customError()`，前者传入一个字符串代表错误，后者可以自定义错误类型。两种方式并无本质上的不同，并且都会导致 `tx` 失败。对于前端而言，我们都需要自定义错误类型来捕获这两种错误。

**运行成本**：合约的状态储存会消耗 Gas 费用（区块链的激励机制，作为付给运行节点的计算与储存费用）为此，在设计储存对象时，如何善用声明的内存是需要被考虑的问题之一。简单的法则是，不要为不需要的状态声明过多的内存空间，如果你需要优化一个合约的运行成本，可以考虑参考许多合约使用内联汇编来优化内存占用。

为此，合约中的复杂数据结构必须声明储存空间位置，例如 `storage`, `memory`, `calldata`，每种位置所产生的费用会有很大不同。合约的函数也会有对应的函数类型声明，`view` 函数 与 `pure` 函数在外部调用时不需要承担 gas 费用，但改变状态的函数都需要消耗 gas。

> 注意：由于合约运行和储存成本高，许多对外部白名单进行管理的最佳实践是使用 MerkleProof，你可以在这里找到它的[合约实现](https://docs.openzeppelin.com/contracts/3.x/api/cryptography#MerkleProof)和 [JavaScript 实现](https://github.com/miguelmota/merkletreejs)。

**不可变**：合约一旦部署，就无法动态替换或进行升级，这意味着，你需要在部署前考虑是否要依赖可升级架构（`Proxy` 部署方案）这些方案所依赖的合约和抽象合约，都需要遵循同一种初始化范式，才能保证合约的可升级性。

**权限和可见性**：合约不同于服务端代码，它对网络中的所有人是透明的，这里的透明不仅指的是合约的字节码，还包括它的**公开**和**私有状态**。这意味着，你不应当在合约中储存任何敏感数据，也不应当依赖区块当中的任何状态（比如区块高度和时间戳）作为核心业务逻辑的判断基准。

为此，发布一个未经权限控制的合约是十分危险的，任何外部账户都可以轻易地对某个合约进行修改，并通过发送消耗合约指令将合约中的资产转走。所以，除了特定的治理合约不受权限管制以外，我推荐任何合约都必须至少依赖 [Ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) 来进行基本的权限配置，同时，复杂合约可以使用 [AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol) 来进行管理。

**安全性**：如上所述，合约的安全性是非常重要和严谨的问题，在将合约发布到生产环境网络之前，确保你已阅读 ConsenSys 编写的[合约代码安全最佳实践指南](https://github.com/ConsenSys/smart-contract-best-practices)并遵守其中所有的约定，同时，确保合约有足够的测试用例并且较高的测试覆盖度。请不要带有侥幸心理发布未经任何测试的合约代码，并主观地希望它能够正常工作。

### 2.2 合约依赖与调用

**依赖引入**：合约可以通过 `import` 引入依赖的外部合约，抽象合约，Interface 或者库。通常，我们使用 npm 管理合约的外部依赖，管理合约的依赖也有其他办法（例如 git submodule）这会在工作流章节中详细叙述。

**调用**：合约可以调用其他合约，只需知道地址和 ABI，我们就可以在合约内部调用其他合约，需要注意的是，调用合约也是事务性操作，因此，你不需要通过手动管理异步操作的方式来等待返回结果。在合约内部调用其他合约需要消耗额外的 Gas 费用。调用合约可能由于 ABI 错误或者不支持某个函数方法而导致失败，但 Gas 费用并不会返还，我们需要确保在调用其他第三方合约前理解对方合约的接口（包括参数类型，顺序，返回结构）

如果你试图调试本地合约调用某个生产环境的线上合约，可以使用 fork 的方式将某个高度的区块链下载到本地运行，这会在工作流章节中详细叙述。

**ABI**：也叫应用程序二进制接口（Application Binary Interface）ABI 是我们理解如何操作一个合约的具体方法的描述，通常在 Interface 文件中被定义（如果合约命名为 `Membership.sol`，那么它的 Interface 文件通常叫做 `IMembership.sol`）

> 注意：通过这种方式定义可以让任意合约通过引用 interface 的方式来调用你的合约，但如果你不在 Interface 中文件定义它，编译器也能帮助你编译出 ABI。

我们可以依赖完整的 ABI 来调用合约（对外部调用者来说，ABI 通常被编译成一个 `JSON` 文件），也可以使用它其中的一部分来调用，只要它满足真实合约所声明的函数（包括参数，参数类型，返回值，返回值类型都一致）后者通常被成为 human-readable ABI，例如：

```
calldatas[0] = abi.encodeWithSignature(
  'execTransfer(uint256,address,address[],uint256[])',
   memberId,
   memberWallet,
   payroll.tokens.addresses,
   payroll.tokens.amounts
);
```

**合约事件**：由于合约的函数调用是事务性的，并且无法为外部调用者（指代 DApp 或钱包用户）提供返回值，合约引入了事件的概念。

事件通过向日志系统中写入特定数据的方式来实现函数修改的记录。我们可以通过监听和查询的方式列出一个合约注册的所有事件，实现对函数异步结果的查询和前端 UI 状态变更。合约事件以某个单一合约为 key 来进行索引，同时，在声明事件时，我们可以指定不多于三个 `index key` 来确保 DApp 前端对这些索引 key 的查询效率，例如：

```
event ModuleProposalCreated(
  address indexed module,
  bytes32 indexed id,
  address indexed sender,
  uint256 timestamp
);
```

如果你期望的查询是非常复杂的，包括一系列相关联的合约事件，更好的方法是采用 Relay 提供的 graph/webhook 来进行查询。

**创建合约**：我们可以通过合约创建其他合约，这意味着，合约可以成为其他合约的工厂合约或者代理合约。我们也可以通过外部调用者（钱包账户）向 `0x00` 地址发送合约创建操作来新建网络上的合约，这是我们进行测试和依赖工作流创建合约的方法。

创建合约需要消耗大量 Gas 费用，通常，我们会使用特定工具在创建合约前预估并计算费用，这会在工作流章节中详述。

### 2.3 合约编程语言特征

Sol 需要依赖相应工作流被编译成字节码发布到对应环境的网络中才能被运行，因此，它不像 JavaScript 那样的动态类型语言有随处可见的 runtime，编译器在检查时会帮助我们发现大部分问题，因此，你需要一个 IDE，例如 [VSCode](https://code.visualstudio.com/) 之类的 IDE 或编辑器才能进入合约开发。

Sol 与大部分编程语言类似，支持基本多种数据类型（但不支持浮点数）、复杂数据结构（例如 `map`，`array` 和 `struct`）、合约支持继承和多重继承（`is`）、原型方法重写（`override`）等。合约有特殊的构造函数，合约声明的函数支持修饰器语法。特殊地，合约中可以通过 `payable` 声明或显示转换来实现对原生 Gas token (ETH) 的资金操作。

Sol 虽然是图灵完备的语言，但其中复杂结构的操作会带来相应的 gas 消耗，因此，在设计合约中的状态变量时，应当足够清晰和简单。

### 2.4 阅读优秀的合约代码

合约编程虽然不复杂，但大量的运行时限制和非冗余的设计，导致我们在进行合约编码时，不得不参考许多优秀的合约代码，才能保证我们的合约代码质量。

对于许多其他领域的程序员来说，这一步更是非常必要的。我推荐大家在合约编码的过程中，反复参考优秀合约项目的设计思路和编码思维。在这里，我为大家推荐一些我认为不错的智能合约开源项目。

首先，[OpenZeppelin](https://github.com/OpenZeppelin) 合约是进入 Web3 领域必须反复的阅读的圣经之一，自 2017 年以来，他们实现了大量的 EIP（以太坊改进提案），并成为了智能合约编码的实际标准。虽然，OZ 的合约在 Gas 费用和效率上存在一些问题，但他们在安全性、代码完成度、可维护性、注释和测试方面都做的很好，是值得信赖的合约基础库。最近，OZ 也发布了他们在 StarkNet 上的 [Cairo 语言版本合约](https://github.com/OpenZeppelin/cairo-contracts)。

[Solmate](https://github.com/Rari-Capital/solmate) 也提供了一系列对应的 EIP 实现，同时，他们更注重合约的运行效率，优化了执行中的 gas 费用，并且每个合约依赖更少，阅读起来更加简单。

[ERC721A](https://www.erc721a.org/) 是知名 NFT 项目 [Azuki](https://www.azuki.com/zh) 发布的 ERC721 改善版本，通过特定的位操作，他们实现了内存占用的优化，带来了批量 mint 低 Gas 费用的优势。如果你的项目涉及到大量 NFT 的铸造，可以参考它的合约代码来进行实现。

[Compond](https://compound.finance/) 是 DeFi 借贷领域的老牌项目，代码质量经过实践的检验，如果你的项目设计到 DeFi 相关的需求，请务必阅读他们的合约代码。

[Uniswap](https://uniswap.org/) 是世界上最大的 DEX，他们的合约实现的非常优秀，无论你是否有 DeFi 方面的需求，我都建议你完整阅读他们的合约代码。

[Lens](https://lens.dev/) 是 [AAVE](https://aave.com/) 推出的以 NFT 为核心的新型社交合约开发套件（或者他们称之为社交合约协议）如果你的项目设计到 SocialFi，可以参考他们的代码实现。

其次，我想给大家推荐的是 [Zora](https://zora.co/) v3 版本合约与 [Gonsis safe](https://gnosis-safe.io/)，前者是著名的 NFT 交易市场退出的交易合约，后者是著名的多签名钱包合约实现。这些都是我们在使用智能合约能够完成的产品当中非常重要的组成部分：

最后，如果你对 DAO 和链上治理感兴趣，我推荐你阅读我编写的 [CodeforDAO](https://twitter.com/codefordao) 的合约，在这个项目中，实现了传统的治理模式，多签积极治理与模块化合约。

___

## 3\. 开发工作流与单元测试

当我们掌握编写智能合约的编程语言后，便可以开始进行工程编码，在这一章节中，我会介绍流行的 DApp 合约开发工作流和编写单元测试的方法。

智能合于自 2017 年发展至今，已存在相当多的项目支撑合约开发中的工作流，如今，大部分项目使用 [Hardhat](https://hardhat.org/) 来支持本地开发工作流。

Hardhat 提供了一种简单的方式创建本地 EVM 兼容区块链开发的环境，并且支持直观的 debug 方式，此外，还有丰富的插件社区，帮助开发者完成一系列特定的需求。

依赖 Hardhat，我们可以在本地创建 block 快速确认的开发环境，使用公开私钥的调试钱包作为测试用户，编译合约并发布到本地测试网络，编写并在内存网络中快速运行单元测试，如果你是 DApp 开发的入门新手，使用 Hardhat 是作为工作流最简单和直接的方案。

Hardhat 还支持配置不同的区块链网络并通过工作流部署合约到生产环境，或者将某个高度的区块链 fork 到本地创建集成测试环境。它提供丰富齐全的文档，可以在他们的官网进行参考。

### 3.1 整合 Hardhat 工作流

有两种方法可以简单将 Hardhat 整合到你的合约项目，最简单的方法是采用 `npx hardhat` 向导，它会帮助你在本地建立一个特定的脚手架项目并安装对应依赖。

如果你打算在已经初始化的项目中引入 hardhat 工作流，手动的方式是新建一个配置文件 `hardhat.config.js` 并安装对应依赖 `npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers`

hardhat 工作流非常简单，我们不需要花太多时间理解它，对于前端工程师而言，它就像是智能合约领域中的 Webpack（但没有 Webpack 的配置那么复杂）它预设了一系列的 task（任务）帮助你将 Sol 文件编译为字节码并输出对应的 ABI 到本地，当然，它也帮助你整合单元测试环境并运行测试。

**Hardhat 任务：**

对于一般的合约开发而言，常用的任务是：

-   `npx hardhat node` 运行一个本地区块链节点并将 JSON-RPC endpoint 暴露给客户端。
-   `npx hardhat test` 运行合约的单元测试。
-   `npx hardhat deploy` 发布合约的字节码到某个区块链网络，网络地址和 deployer 账户，这些我们可以在 `hardhat.config.js` 文件中配置。

**整合插件：**

hardhat 工作流还支持插件机制，插件能够将特定逻辑作为 hook function （钩子函数）插入到对应的 task（任务）当中，所以，当我们执行某个任务时，需要确认它是否依赖了某个插件，否则它可能有与预期不同的行为。

引入插件的方式是，首先使用 npm 安装这个插件，再于 `hardhat.config.js` 配置文件头部引入即可。

**HRE 运行环境变量：**

当我们在 JS 文件中引入 hardhat 时，HRE 会被插入运行环境。一些插件可能会拓展 HRE，将他们的实用函数方法插入到 HRE 中，类似地，我们也可以使用这种方法构建特定的 hardhat 插件。但一般来说，我们不需要这么做。

HRE 会在我们执行 `npx hardhat run` 任务时被自动插入到全局变量中去，我们可以通过这种方法编写某些简单的合约发布脚本或合约交互脚本。

### 3.2 单元测试

编写合约的第二步是编写合约的单元测试。当我们运行 `npx hardhat test` 任务时，hardhat 会自动寻找 `./test` 文件夹下的单元测试并运行它们。这个默认的地址可以在 `hardhat.config.js` 配置文件中使用 `path.tests` 修改：

```
// Rewrite the `./test` folder to `./tests`
paths: {
  tests: './tests',
},
```

运行测试所需要安装的依赖可以在这个指南上找到：

与传统的单元测试一样，使用 `Mocha` 作为单元测试框架。对于合约特定的变量类型，我们使用 `Waffle` 和 `chai` 作为断言库。

一般来说，我们在 `hardhat.config.js` 配置文件头部引入测试辅助插件 `@nomiclabs/hardhat-waffle` 可以帮助我们解决大部分问题，而不需要额外手动安装 mocha, waffle 和 chai 并进行配置，与前一小节所提到的 HRE 相关，它们会被自动插入 HRE 运行环境。

关于合约事件，合约方法调用，BigNumber 等完整的断言库范例可以在这个文档中找到：

> 注意：合约的单元测试中可以使用 `contractInstance.connect(signer)` 来随意改变调用合约的外部账户。

### 3.3 改善测试效率

编写单元测试首先需要我们在测试钩子中编写发布合约的代码，这意味着，我们需要在每次 `beforeEach` 钩子中重新发布我们的合约并使其从零状态开始运行。

即使 hardhat 支持在内存中运行区块链并整合了单元测试流程，但这样反复的发布合约也会极大拖慢测试速度。

因此，就单元测试的最佳实践，我向大家推荐 `hardhat-deploy` 插件。

`hardhat-deploy` 插件支持使用 `evm_snapshot` 快速地跳转到某个高度的区块链状态，因此，我们可以使用它在单元测试中维护测试前、中、后以及各种特定高度状态，极大地加快测试速度。

> 注意，引入 `hardhat-deploy` 插件，需要修改对应的 `@nomiclabs/hardhat-ethers` 插件来源，这可能会导致在未来的 `npm install` 中带来版本冲突，如果你遇到了版本冲突，可以使用 `npm install --force` 跳过版本依赖检查，强制安装两者。

```
"devDependencies": {
    "@nomiclabs/hardhat-ethers": "npm:hardhat-deploy-ethers",
    "hardhat-deploy": "^0.11.2",
    ...
}
```

简单来说，在单元测试中，我们可以使用：

```
await deployments.fixture(['SomeContractName']);
```

来确保在测试执行前跳回某个状态。如果你需要更加复杂和自定义的 fixture（而非直接跳回某个合约发布后的干净状态）可以使用 `deployments.createFixture` 来创建自定义 fixture，具体的范例代码和指南可以在这里寻找到：

值得注意的是，使用 `hardhat-deploy` 插件会同时改变我们发布合约的代码逻辑（正如其名）它支持在 `./deploy` 文件夹下编写每个合约的发布脚本。事实上，默认的 `deployments.fixture` 正会退回这些发布脚本所叙述的合约状态。

不同于 hardhat 默认的发布脚本（使用 `npx hardhat run`）我们可以使用 `hardhat-deploy` 插件提供的功能在发布脚本中做更多工作，例如使用 `execute` 函数立即修改发布后的合约状态，这会在对权限敏感的合约当中非常有用：

```
const { deploy, execute } = deployments;
const shareGovernor = await deploy('ShareGovernor', {
    contract: 'TreasuryGovernor',
    from: deployer,
    args: [name + '-ShareGovernor', share.address, treasury.address, settings.share.governor],
    log: true,
  });

  // Setup governor roles
  // Both membership and share governance have PROPOSER_ROLE by default
  await execute(
    'Treasury',
    { from: deployer },
    'grantRole',
    PROPOSER_ROLE,
    membershipGovernor.address
  );
```

除此之外，`hardhat-deploy` 插件还提供了非常多的 HRE 实用函数，例如`getNamedAccounts` 能帮助我们命名本地测试账户，而非使用数组下标访问它们。你可以参考该插件的 GitHub 主页了解这些实用功能。

### 3.4 测试覆盖率与 Gas 报告

当合约的单元测试编码到一定程度之后，我们会希望这些合约被发布到某个测试或生产环境（例如测试网络或 ETH 主网）时是否是健壮和低成本的，在此时，我向你推荐两个 hardhat 插件 `hardhat-gas-reporter` 与 `solidity-coverage`

![hardhat-gas-reporter](https://user-images.githubusercontent.com/7332026/59982003-c30a4380-95c0-11e9-9d93-e3af979df227.png)

hardhat-gas-reporter

`hardhat-gas-reporter` 插件帮助你了解运行单元测试中部署和执行合约方法消耗的 gas 费用，如果在本地环境变量中提供 `COINMARKETCAP_API_KEY`，它会自动将这些成本折算为美元或其他法币计价。

`solidity-coverage` 插件提供单元测试覆盖率报告，这有助于开发团队理解合约是否得到了应有的测试。

### 3.5 其他实用插件

如前所述，hardhat 并不需要以来太多的插件就可以正常工作，满足大部分合约开发团队的需求，我在这里推荐两个其他的使用插件，他们是 `@nomiclabs/hardhat-etherscan` 和 `@tenderly/hardhat-tenderly`

这些插件都是可选的，并依赖第三方服务的 API Key，各位读者可以根据自己的情况选择是否使用他们。

`hardhat-etherscan` 插件将 etherscan 网站的源码 verify 功能整合到发布工作流中，能够将所发布合约的源码和 ABI 都展示在合约地址页面。

`hardhat-tenderly` 插件整合了 Tenderly 工作流，后者是一个新兴的 CI/监控 平台，能够帮助我们监控线上合约的状态并提供 debug 建议。

### 3.6 更快的工作流方案：Foundry

尽管在过去的三年中 hardhat 逐渐垄断了 EVM 兼容链中的智能合约开发工作流市场，但最近也有很多强有力的竞争者出现，[Foundry](https://book.getfoundry.sh/index.html) 就是其中之一：

[Foundry](https://book.getfoundry.sh/index.html) 由 Rust 语言编写，并在很多方面极大地提升了合约单元测试的运行效率：

![Forge VS Harhat](https://github.com/foundry-rs/foundry/raw/master/.github/compilation-benchmark.png)

Forge VS Harhat

Foundry 由它的命令行工具 `Forge` 与 `cast` 组成，前者帮助我们安装第三方依赖组件（使用 git submodule 方式）运行测试，发布合约，后者帮助我们与合约进行 RPC 通信交互。

同时，它也改变了我们编写测试的方法，让开发者可以直接编写 Sol 而不再依赖 JavaScript 就可以编写合约的单元测试，测试文件与合约源码在同一个文件夹中管理，通常以 `ContractName.t.Sol` 特殊后缀结尾。

它也提供了一系列的测试套件工具帮助我们编写基于 Sol 的单元测试，包括可继承的 `Test` 合约，和一个特殊的，与 vm 通信的合约 `Cheatcodes` 帮助我们改变外部调用者地址，进行错误断言等等功能。

如果你对 Foundry 感兴趣，我推荐你阅读他们的文档，它的学习曲线并不陡峭，但考虑到使用它会改变绝大部分智能合约项目的开发流程，我推荐你在本地分支中支持它并同时兼容 hardhat 工作流。

___

## 4\. 前端与客户端开发

由于第三方钱包软件的盛行，Web3 大部分产品（DApp）的用户节目实际上都由前端网页所构成，这与移动互联网的开发流程相悖，但很像早期 Web2.0 历史发展进程的一部分。

许多 DApp 并不提供 Mobile App 版本，部分原因是由于构建一个跨平台钱包方案过于复杂，以及大部分 Web3 领域内的用户都在使用诸如 MetaMask 这类浏览器插件钱包，而它的移动端 App 体验并不好用。

我们当然可以使用托管钱包服务来进行开发，让更多不熟悉 Web3 的用户使用邮箱或者密码登录，但这会带来一系列的安全问题与风险，况且，就算我们能够使用简单易用的钱包降低用户准入门槛，在许多国家，用户仍需要复杂的 kyc 才能获取到某些 token。

所以，我建议你也采用 Web 前端作为第一个 DApp 的界面方案。

在本章节，我们所涉及到的大部分内容都是基于这样的假设，因此，这部分内容需要你熟悉 JavaScript，React.js/Vue.js 和它们相关的工作流。

### 4.1 前端框架的选择

使用何种视图框架并不会影响你的 DApp 体验，但是，这会影响到开发效率。

在本文的第一部分「认识 DApp 技术栈」我们介绍了 [ethers.js](https://docs.ethers.io/v5/) 和 [web3.js](https://web3js.readthedocs.io/)，这两者都是构建 Web 前端的基础类库。如上所述，我建议使用 ethers.js 入门进行开发。

事实上，相较于 Vue 而言，React 的生态系统中目前拥有较多的活跃 Web3 开发者和相关依赖库，如果你没有特殊的偏好，可以尝试先采用 React 作为框架来进行开发。

另外，大部分对于 ethers.js/web3.js 的包装项目，诸如 [web3-react](https://github.com/NoahZinsmeister/web3-react)，[wagmi](https://github.com/tmm/wagmi) 等都依赖你理解 React hooks 的概念，所以在你着手进行编码之前，需要查阅[它的文档](https://zh-hans.reactjs.org/docs/hooks-intro.html)。

### 4.2 搭建脚手架项目

我们可以使用任何蓝图工具创建对应的视图框架的脚手架项目，但是，我们也可以参考现有的 Web App 项目来进行开发。

在编写前端代码之前，我推荐你参考流行的脚手架项目 [scaffold-eth](https://github.com/scaffold-eth/scaffold-eth)

[scaffold-eth](https://github.com/scaffold-eth/scaffold-eth) 是一个完整的合约脚手架，它的 `packages/react-app` 文件夹是它的 Web 前端代码，对于熟练合约开发的工程师而言，这个脚手架的组织方式有些不太理想，但对于刚入门 Web3，打算搭建第一个测试项目的朋友来说，这是个很棒的入门工具。

当我们决定了使用何种视图框架来开发前端 Web App 之后，就可以着手使用它对应的脚手架 cli 来创建项目了。对于没有任何偏好的读者，我推荐你使用 React + Next.js 来初始化新项目，Next.js 使用 React 作为基础视图框架，并提供了丰富的工作流，简单的路由系统，好用的 SSR 与 FaaS 支持，当然，它也是一个非常好用 site builder 工具。

### 4.3 前端与合约的交互流程

当我们在 DApp 的业务逻辑编码进行到一定程度后，需要与合约 ABI 进行读写，或者，我们需要连接用户的钱包，为其铸造一个 NFT，这里就涉及到前端与合约的交互。

在本文的第一章节「认识 DApp 技术栈」中，我们提到与区块链网络进行交互最终会使用 `Provider/Signer`（前端） + `Relay Network`（区块链端）因此，这个流程最终会使用 [ethers.js](https://docs.ethers.io/v5/) 或 [web3.js](https://web3js.readthedocs.io/) 发送对应的 XHR 请求到 Relay Network 的 API Endpoint。但是，具体而言，开发者和用户如何理解这一与众不同的流程呢？

一般来说，我们可以将这个交互流程简述为：

1.  用户打开网页，默认进入只读模式。我们只能通过 `Provider + Relay Network` 访问到我们所设定的默认网络的合约的 `view` 方法返回值。
2.  用户点击「连接钱包」时，我们和对应的 `connector` 进行通信，获取连接状态。如果用户授权连接，我们可以通过 `connector` 获取到用户钱包的地址（`0x`）在此之后，用户钱包的 `Provider`只读模式将只支持用户钱包中选定的网络，如果网络不符合我们的期望，我们可以通过 `connector` 的特定通信来请求用户修改它。
3.  用户点击某个表单，希望与合约的写操作函数进行交互。此时， [ethers.js](https://docs.ethers.io/v5/) 或 [web3.js](https://web3js.readthedocs.io/) 会将交易信息打包发给 `connector`，后者将引导用户进行签名和确认交易的操作。如果交易成功提交到区块链网络（经由我们配置的 `Relay Network`）我们需要监听该交易的状态和合约事件，进行下一步前端状态更新。
4.  用户进行钱包登录操作或其他签名操作时，我们可以不与合约通信仅在本地请求 `connector`来发行使用用户钱包私钥签名后的加密数据。

在进行以上所述的流程之前，我们需要回顾 「认识 DApp 技术栈」中的内容，并且准备好 `Relay Network`的 access key，无论操作是读或者写，我们都需要准备这些 access key 才能为用户提供高质量和稳定的请求访问。

我们可以通过提供多个 Provider 实体的方式来访问多个区块链网络的合约 ABI，但一般而言，我们只能依赖 `connector`中用户选定的网络来进行写操作。

对于合约多个 ABI 的写操作可以合并请求，这样可以减少用户在进行操作时的 Gas 费用，如果你有这个需求，可以参考 Multicall.js 的实现：

### 4.4 前端依赖

大量的重复工作都建立在优秀的开源项目基础上，在 DApp 编写过程中，我推荐你使用一些优秀的前端库来减少工作量，并实现更好的代码交付质量。

wagmi 是我推荐的核心依赖之一，它提供了丰富的 React hooks 来完成 DApp Web 前端与合约交互的所有流程。它的实现简单，测试健全，而且没有多余的冗余依赖库。

谈到 React hooks，我也推荐 useDApp，相比于 wagmi，它更加复杂，但默认支持 multicall.js

如果你的网站将要集成钱包登录的功能，那你则需要考虑引入 Siwe（Sign-In with Ethereum）它实现了 EIP-4361 中的钱包登录流程。

如果你的 Next.js DApp 计划提供多语言版本和检测，我推荐你使用 `i18next` 与 `react-i18next` 与 `i18next-browser-languagedetector` 这些依赖与 DApp 的核心交互逻辑没有关系，因此不再赘述。

在 UI 库方面，我推荐基于 Google Materials UI 设计系统的 MUI 与 NextUI：

### 4.5 客户端开发

客户端开发的方案比较多样，流行的方案是 React Native（跨平台）Flutter（跨平台）Swift（iOS）和 Java (Android) 这些方案都有一些流行的依赖库可以借鉴。

考虑到 React Native（跨平台）的实现，它的依赖库与 React 应当并无差异，可以使用上述针对 React 的方案。

对于 Flutter（跨平台） 而言，我推荐你参考这一官方指南：

其中，我们可以使用 web3dart 来与区块链 Relay 进行通信：

对于 Swift（iOS）而言，我们可以使用 Argent labs 团队提供的 [web3.swift](https://github.com/argentlabs/web3.swift) 方案：

对于 Java (Android) 而言，流行的方案之一是 Web3j：

___

## 5\. 开发、测试与生产环境调试

与其他软件一样，DApp 在正式上线过程前也会经过几个环节的调试与测试过程。与其他软件不同的时，我们通常无法简单地在本地搭建所有测试环境。

### 5.1 开发环境调试

在「开发工作流与单元测试」章节中，我们提到使用 `hardhat node` 能够快速在本地运行一个自动 mining 的区块链调试网络。那么，我们如何将每次修改的合约 ABI 同步给前端项目呢？

默认地，hardhat 会将编译后的 ABI 文件和合约字节码放在 `./artifacts` 文件夹，但不包括合约地址，文件组织方式对前端项目也并不友好。

借助 `hardhat-deploy` 插件，我们可以简单地使用 `--export-all` 导出所有被发布的合约 ABI（包含地址信息）为一个完整的 json 文件，例如：

```
npx hardhat node --export-all ../website/contracts/localhost.json
```

这个 json 文件的结构看起来是这样：

```
{
  "31337": [
    {
      "name": "localhost",
      "chainId": "31337",
      "contracts": {
         "Membership": { address: "...", abi: [...] }
         ...
    }
  ]
}
```

可以看出，它是一个由 Chain ID（31337 是 hardhat Chain ID）索引的合约 ABI 清单，根据这个清单，我们可以很方便的构建出对应的合约实例并与本地合约进行交互。

> 注意：前端项目与本地合约进行调试时，请特别注意 `Provider/Signer` 当前连接的网络。另外，默认地，hardhat 网络的区块确认是即时的，如果你需要模仿公开网络的行为，可以在这里寻找到[修改它的配置](https://hardhat.org/hardhat-network/explanation/mining-modes.html)。

### 5.2 测试与生产环境调试

当我们的合约准备发布到公开测试网络，例如 Rinkeby, Kovan, Ropsten 或者 Goerli 时，我们只需要在 hardhat deploy 中指定对应的 network 选项即可:

```
npx hardhat deploy --network rinkeby
```

需要注意的是，我们需要确保对应的 deployer 钱包账户中有足够的 Gas Token 余额，对于上述网络来说，即是 ETH 余额。

合约一旦发布到公开网络，它的状态就不受到我们的控制，任何用户都可以根据我们提供的 ABI 修改某个合约的状态，因此，如果你需要稳定的，某种状态的测试合约充当不同版本的测试环境，确保你在发布之前使用了特殊的权限管理或是地址硬编码。

如果你不想如此麻烦地管理测试网络，我推荐你使用 `hardhat node --fork URI` 功能在本地计算机或者服务器集群中 fork 主网状态充当测试环境。你可以在这里找到它的[详细指南](https://hardhat.org/hardhat-network/guides/mainnet-forking.html)。

公开测试环境中的第三方合约状态是未可知的，因此，我们需要再三确认调用的地址是否正确。其次，大部分流行的协议或者 DEX 在几大公开测试网络都提供了测试合约，包括 OpenSea 在内，部分流行的 DApp 前端也提供了测试网络的版本，以帮助开发者在发布到线上网络前发现问题：

此外，你需要一些测试 ETH 才能确保公开测试网络中合约与逻辑正常工作，同时，你的用户也会需要它们。这里是一些可以获得测试 ETH 的网站和服务：

___

## 6\. 服务端编码与集成

服务端一直是 DApp 被认为没那么「去中心化」的原因之一。就我所知，世界上绝大多数 DApp 都有服务端 API 提供支持，只有少数类似 [Uniswap](https://app.uniswap.org/) 这样的产品，仅依赖前端与合约进行通信。

但绝大多数 DApp 的 Web UI 实现了去中心化。因此，我们需要区分服务端 API 在 DApp 开发中所属的地位，不能将核心逻辑放在私有服务器中依赖，或者一味使用服务端储存的机器人钱包私钥来操作区块链。

我们需要编写服务端 API 的原因之一是，链上状态储存的成本过高，以及反复地签名与交互对用户来说体验不佳。另外一些原因是，一些不重要的，可以被丢弃的数据并不需要放在合约中储存。

DApp 的服务端 API 并不要求特殊配置。因此，你可以使用任何你喜欢的编程语言运行环境来编写它。一般来说，我们使用 Node，因为这样可以复用一部分 `Provider/Signer` 的前端业务逻辑。

### 6.1 开发环境

如果你的 DApp 并不复杂，不需要储存太多状态，我推荐你使用上述章节提到的 Next.js 方案，它可以直接被 push 到 [Vercel](https://vercel.com/)，后者将能够自动地将你 Next.js 项目中的 API 部署到对应的服务环境。

如果你的 DApp 依赖较多的数据库和服务，我推荐你使用 FaaS，我常用的一个 FaaS 服务是 Firebase，你可以使用 Firebase 快速连接实时数据库，整合 Twitter 或 GitHub 的第三方登录，它还提供非常好用的本地模拟器工具套件，以及，它能够非常好地支持跨平台。

服务端编码没有太多与 DApp 开发相关的内容，但其中有两个我们需要提及的部分：

1.  理解使用 Siwe 进行钱包登录的流程
2.  在服务端使用 `Provider/Signer` 与区块链 Relay 进行通信

接下来我们会简单介绍这两部分内容。

### 6.2 关于钱包登录

许多刚入门 Web3 的开发者会认为，钱包登录只需要使用前端脚本连接钱包即可，但这种逻辑很容易被 hack，因为任何客户端状态都能够被低成本修改。

Siwe（[Sign-In with Ethereum](https://docs.login.xyz/general-information/siwe-overview/eip-4361)）将 `EIP-4361` 草案引入了以太坊改进协议，目的是标准化开发者使用钱包登录授权 Off-chain 产品的逻辑。它的流程与 [JWT](https://jwt.io/) 的发行相似。

Siwe 支持绝大多数编程语言和它们的运行环境（JavaScript, Rust, Python, Golang, Ruby 等）因此你可以直接使用官方提供的依赖库来完成大部分流程。如果你使用 Next.js 也可以采用 [NextAuth](https://next-auth.js.org/) 来快速整合它。

简单来说，Siwe 通过对服务端发行的随机 `nonce` 和其他标准输入进行签名，再通过服务端验证签名内容的方式来确认提交方的地址。

因此，我们需要提供两个 API 来实现 Siwe，分别是 `/nonce` 和 `/vertify` 你可以在这里找到它们的代码范例：

### 6.3 服务端与区块链的通信

在某些情况下我们需要在服务端进行与区块链 Relay Network 的通信，如果你使用 Node 作为运行环境，它的逻辑代码和前端是一致的。

如果服务端只需要使用到 Provider 与 Relay Network 通信（只读模式），比如，我们通过整合区块高度，某个合约状态和订阅事件，来组建我们自己的数据缓存服务并提供 API，那我们只需要按照前端的方式与 Relay Network 建立通信即可。在这种模式下，我们可以把 access key 存放在生产环境的环境变量中，我推荐你使用 [dotenv](https://github.com/motdotla/dotenv) 去处理它们。

如果你使用 Next.js 它会自动读取 dotenv 的环境变量文件，因此你可以在 `process.env` 中快速使用到它们。这里是 Next.js 的[相关文档](https://nextjs.org/docs/basic-features/environment-variables#environment-variable-load-order)。如果你使用其他服务端框架或 FaaS，你可以自己维护这些环境变量文件。

如果服务端需要使用到 Provider 和 Signer 与 Relay Network 进行通信（读写模式）我们就需要好好考虑如何存放机器人钱包的密钥的问题。

> 注意：在做这些事情之前，我们应该提前思考为何需要在服务端使用机器人钱包对某些合约进行写操作，以及这样的设计带来的合约权限问题与安全风险。

我认为，将私钥放在环境变量中不是一个好办法，我们无法控制第三方模块是否会将环境变量中的内容计入日志或者远程统计。因此，我们可以采用专业的私钥管理服务来管理，例如使用 [Google Secret Manager](https://cloud.google.com/secret-manager) 或者 [AWS Secrets Manager](https://aws.amazon.com/cn/secrets-manager/) 来进行管理，而前者可以与 Firebase 很好地进行整合。

通过 Provider 和 Signer 与 Relay Network 进行通信只需要传递钱包私钥给对应的 `Provider/Signer` 实例即可完成操作，与本地进行单元测试的机器人钱包一样，它不会有额外的确认过程，因此，我们需要确认钱包中有足够的 ETH 或其他 Gas token 余额，否则该交易会失败。

一般来说，如果我们的服务端接口中有对应的合约写操作，我们不会等待交易完成再返回数据，因此，我们需要返回对应的 `tx.hash` 方便前端界面处理后续逻辑。

### 6.4 实用的 SDK

一些 SDK 和对应服务可以帮助我们更方便地在服务端与合约进行通信，例如 [ThirdWeb](https://thirdweb.com/)：

Thirdweb 提供了一个 SaaS 合约开发平台，你可以通过它的前端 App 发布预设功能的合约，例如 NFT Drop 或者 NFT 交易平台，也可以使用它提供的第三方 access key 与已发布的合约进行通信（而无需依赖合约的 ABI）：

```
import { ThirdwebSDK } from "@thirdweb-dev/sdk";

// The RPC url determines which blockchain you want to connect to
const rpcUrl = "https://polygon-rpc.com/";
// instantiate the SDK as read only on a given blockchain
const sdk = new ThirdwebSDK(rpcUrl);

// access your deployed contracts
const nftDrop = sdk.getNFTDrop("0x...");
const marketplace = sdk.getMarketplace("0x...");

// Read from your contracts
const claimedNFTs = await nftDrop.getAllClaimed();
const listings = await marketplace.getActiveListings();
```

你可以在这里找到它的 [JavaScript SDK](https://github.com/thirdweb-dev/typescript-sdk)（Typescript）

## 7\. 合约部署方案 L1s & L2

在 DApp 开发中，与传统产品最大的不同点之一是我们需要决定将产品核心逻辑的智能合约发布到那个网络（或者哪些网络）这意味着，DApp 需要有「跨平台跨网络」的支持能力。

对于传统互联网开发人员来说，我们很容易理解「跨平台」，它是指我们需要为 App 界面提供 PC Web/Mobile Web，iOS/Android App 的各种版本。「跨网络」在 DApp 的开发中指的是，我们需要让 DApp 前端/客户端支持多个区块链网络。在那之前，我们需要决定哪些区块链网络是我们首选的发布环境。

自 2017 年发展至今，目前市场中有大量的区块链网络供我们使用。按照它们的共识证明种类，可以被分为 PoS 和 PoW；按照它们的角色定位，可以分为 L1 与 L2；按照它们对 EVM 兼容的类型，可以分为 EVM 兼容链和非 EVM 兼容链。

一般来说，我们可以选择发行到这些流行的区块链网络：

1.  [Ethereum](https://ethereum.org/zh/) (ETH) 主网：Gas 费用昂贵，但其中储存了大量资产，如果你的项目与 NFT 相关，许多人会选择发布到主网。
2.  [Polygon](https://polygon.technology/) (Matic)：类 ETH 的 PoS 侧链，EVM 兼容，在许多国家都有一定的用户基础，有限的 TPS 支持与可接受的成本，开发者友好。
3.  [BNB Chain](https://www.bnbchain.world/) (BNB): 币安的区块链网络，EVM 兼容，开发者友好。
4.  [Solana](https://solana.com/zh) (SOL): 高性能区块链网络，支持[多种编程语言](https://solana.com/zh/news/getting-started-with-solana-development)编写合约，EVM 兼容（使用 [Neon](https://neon-labs.org/)）
5.  [AVAX C-chain](https://www.avax.network/) (AVAX)：AVAX 的应用链，EVM 兼容，提供快速区块确认，相当程度的 TPS，可以自己搭建 C-chain 作为应用链 sub-chain（例如 [DFK Crystalvale](https://defikingdoms.com/crystalvale/)）
6.  [Cosmos](https://cosmos.network/)(ATOM): 连接应用链的区块链网络，非 EVM 兼容（Evmos 除外）提供快速的 IBC 跨链桥支持，可以自定义应用链的 Gas token，适合 GameFi 与需要定制 TPS 的大型应用。
7.  [Near](https://near.org/zh/) (NEAR)：提供完善的开发者套件和网页钱包一整套方案，因此用户入门难度最低，非 EVM 兼容（使用 Rust 编写合约）。
8.  [StarkNet](https://starkware.co/starknet/) (ETH Layer2): 使用 [zkRollup](https://www.zkrollups.xyz/) 技术支持的 L2 网络，非 EVM 兼容，由 Starkware 提供技术支持（它同时支持了 [IMX](https://www.immutable.com/) 与衍生品 DEX [DyDx](https://dydx.exchange/)）支持与 L1 的合约进行通信，支持使用 [warp](https://github.com/NethermindEth/warp) 工具将 Sol 代码转换为 Cario 语言的合约代码。
9.  [zkSync2.0](https://zksync.io/) (ETH Layer2): 使用 [zkRollup](https://www.zkrollups.xyz/) 技术支持的 L2 网络，同时支持了 DEX [ZigZag](https://info.zigzag.exchange/)。支持与 L1 的合约进行通信。
10.  [scroll](https://scroll.io/) (ETH Layer2): 使用 [zkRollup](https://www.zkrollups.xyz/)(zkEVM)技术支持的 L2 网络。
11.  [xDai](https://www.xdaichain.com/)(Gnosis Chain): 它支持了著名的到场证明合约 [POAP](https://poap.xyz/)。
12.  [Harmony](https://www.harmony.one/)(ONE)：高性能区块链，它支持了 [DFK](https://defikingdoms.com/) 的第一个版本。
13.  [Dfinity](https://dfinity.org/) (ICP)： 一个完整的 DApp 生态系统。

我们可以选择发布到某个区块链网络，或者发布到所有支持 EVM 兼容的网络中。不过，不同的区块链网络中的合约无法直接通信，资产也无法随意互换（可以采用跨链桥合约进行锁定和重新铸造）目前，大部分 DApp 只会选择某一个区块链网络进行发布。

如果你的项目涉及到 NFT，我会推荐发布到 ETH 主网或储存了相当数量资产的网络，如果你的项目涉及 GameFi，可以考虑 TPS 高的区块链网络。如果你考虑 TPS 又同时注重资产安全，可以考虑使用 Layer2 网络。

我们正处于一个区块链网络的战国时代，因此，选择部署的网络不存在绝对的最佳实践，可以参考个人的需求进行选择。

___

## 8\. 去中心化储存方案

在 DApp 开发中，我们通常会将资产的元数据、DApp UI 界面等储存在去中心化储存网络当中，以防止单点故障导致的资产损失和不可用。

简单来说，资产的元数据通常指的是 NFT 中合约储存的 `tokenURI()` 返回的内容，它可能是一个 JSON 编码的字符串，将这些字符串储存在合约当中需要耗费相当大的成本，因此，最佳实践是将他们部署到去中心化储存网络中，再保存储存对象的索引 ID（例如 [IPFS CID](https://docs.ipfs.io/concepts/content-addressing/)）

流行的去中心化储存方案有：

1.  [IPFS](https://ipfs.io/)：最早和最流行的去中心化储存网络。
2.  [Filecoin](https://filecoin.io/zh-cn/)：以 IPFS 为基础的储存网络。
3.  [Arweave](https://www.arweave.org/)(AR)：去中心化的永存网络，一次写入付费，读数据免费。你正在阅读的这篇文章即由 Mirror 代为保存在 AR 上。

由于 IPFS 等方案需要多个节点保持（Pin）储存对象的状态，因此，上述服务都有针对开发者的高级包装储存服务（类似 AWS S3）我向大家推荐：

1.  [Web3.storage](https://web3.storage/) 基于 Filecoin 的免费储存服务。
2.  [NFT.storage](https://nft.storage/) Web3.storage 提供的 NFT 元数据针对性储存服务，提供网页界面上传与 SDK。
3.  [Filebase](https://filebase.com/) 整合了多个去中心化储存网络的服务，接口类似 AWS S3，提供丰富的 SDK 与 API，支持信用卡付费。
4.  [Bundlr](https://docs.bundlr.network/) 基于 Arweave 构建的永久储存服务，支持用多链 token 结算。

如果你考虑为 NFT 元数据寻找去中心化储存方案，需要[确保](https://docs.opensea.io/docs/metadata-standards) OpenSea 支持它们，否则你的 NFT 与合约将无法正常在 OpenSea 页面中显示。OpenSea 目前支持 IPFS 与 Arweave 的储存协议。

另外，如果你在编写合约时预先硬编码写入 NFT 和其他元数据，可以考虑使用 IPFS CAR（Content-Addressed Archive） 来预先计算他们的 CID hash，这有助于保存一些 OpenSea 要求的非标准数据，例如 NFT 合约描述和 Banner 背景图地址。

___

## 9\. 附录

本文中提到的所有项目均列在我的 GitHub Star 清单中，可以在这里统一查阅：

此外，附录中列出了许多我认为有助于帮我们理解 DApp 与智能合约开发的指南，如果你感兴趣，可以参阅这些指南（这个列表也会不定时更新）：

1.  [合约安全风险指南 Ethereum Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
2.  [合约 Gas 优化指南 Awesome Solidity Gas-Optimization](https://github.com/iskdrews/awesome-solidity-gas-optimization)
3.  [brownie](https://github.com/eth-brownie/brownie)：一个 Python 语言的合约工作流工具。
4.  [ethers-rs](https://github.com/gakonst/ethers-rs)：一个 Rust 的钱包实现。
5.  [juice-interface](https://github.com/jbx-protocol/juice-interface)：著名众筹网站 juicebox 的前端界面。
6.  [starknet.js](https://github.com/0xs34n/starknet.js): StarkNet 的 JS SDK。
7.  [DFK/contracts](https://github.com/DefiKingdoms/contracts): 著名 DeFi 游戏 DFK 的智能合约。
8.  [thirdweb-dev/contracts](https://github.com/thirdweb-dev/contracts): ThirdWeb 开源的合约代码。
9.  [OpenZeppelin/nile](https://github.com/OpenZeppelin/nile): OZ 为 StartNet Cairo 语言编写的工作流工具。
10.  [Argent X](https://github.com/argentlabs/argent-x)：StarkNet 的开源钱包。