编注：本文是《[100 小时后请叫我程序员](https://sspai.com/series/271)》栏目的「技能拓展」模块的试读文章。《[100 小时后请叫我程序员](https://sspai.com/series/271)》栏目希望让你学会用编程思维来理解世界，掌握大数据分析的能力，最终做到用代码解决实际需求。

___

最开始使用 Python 时，相信很多读者都会安装一个全局的（Global）Python 解释器，然后就用这唯一的解释器去学习、并运行 Python 代码。

由于 Python 有着丰富的社区生态，在使用 Python 的过程中我们不可避免地会去安装别人写好的模块、库或包，而它们都可以被统称为依赖（Dependency）。依赖一词通俗易懂，它就好比是深陷于恋爱泥潭之中无法自拔的小情侣，如果离开了另一方就活不下去的那种状态。在我们的 Python 代码中如果缺少了这些第三方模块、库或包也同样无法正常运行。

为了安装依赖，我们也因此会接触到一个名为 [pip](https://github.com/pypa/pip) 的安装与管理工具，并且不论是在正文还是在综合案例实践中，我们都会经常用到该工具。只有通过 `pip` 工具安装之后我们才能在代码中导入并使用第三方内容。`pip` 工具通常会随着 Python 解释器一起被安装，并且与对应的解释器版本相绑定。

但对于 Python 工程师来说，在工作时通常会接触到不同的工程项目（Project），而每个项目可能会使用到不同的 Python 版本，比如老旧的项目 A 使用 Python 2.7，最新的项目使用 Python 3.9，而不旧不新的已经上线的项目一般都使用 Python 3.7 等等。不同的 Python 版本具有不同的功能特性，相关依赖对这些版本的兼容情况又有所不同，所以我们无法直接用一个最新的版本来充当「万金油」。

在这种场景之下，只有一个唯一的全局解释器仅仅只能满足其中一个项目的需要。所以就需要一种机制可以让我们随时创建或删除不同的 Python 解释器，于是虚拟环境（Virtual environment）也就应运而生。

使用虚拟环境的好处在于：

-   一方面，可以帮我们快速创建干净、完全隔离的且不同版本的 Python 解释器以便我们能在不同项目中开发；
-   另一方面，可以避免因在不同项目中只使用唯一的全局解释器而导致的**依赖冲突问题**（Dependency Conflicts）。

依赖冲突是一个情况复杂的问题，不仅存在于 Python 中，在其他编程语言中也它也「声名远扬」，这里笔者以一个简单的例子说明：

假设我们现在有两个工程项目 `PA` 与 `PB`，它们分别使用到了 Django 2.0 和 Django 3.1（假设为最新版本）依赖。

在不使用虚拟环境而仅使用一个全局的 Python 3.6 解释器的情况，仅能存在一个 Django 版本：

-   在 `PA` 项目中使用 `pip install django==2.0` 只会令当前解释器安装 Django 2.0 版本；
-   倘若此时在 `PB` 项目中使用 `pip install django` 不指定版本号时，默认安装最新的 Django 3.1 版本，于是此时全局解释器中已经存在的 Django 2.0，则会为 Django 3.1 版本所覆盖并升级。

此时问题来了：如果我们要让 `PA` 项目能正常运行，就必须使用 Django 2.0，如果要令 `PB` 项目正常运行，那么就必须使用 Django 3.1。

而这个问题就是典型的的依赖冲突。

所以，为了避免这种依赖冲突的情况发生，在开发或运行项目前，通常会使用虚拟环境来创建一个纯净的 Python 运行环境，当中仅仅包含了 Python 解释器和必要的 `pip` 工具而没有任何第三方依赖。这个环境就类似于我们常说的「沙盒」（SandBox），在盒子内的东西既不会影响外部的事物，也不会被外部事物所影响。

于是乎为了解决上面所提到的依赖冲突问题，我们只需要分别新建一个包含了 Django 2.0 和包含了 Django 3.1 的虚拟环境，然后运行项目代码时切换到对应的虚拟环境即可。

这也就是为什么我们需要一个虚拟环境，因为只有**统一的开发环境，才能确保程序在每个能拿到这份代码的协作者手中都是能跑起来的**。就好比在中学时一个班里的同学水平有高有低，但都是要在同一个考场考试、做的都是同一套试卷一样。

尽管人们常说 Python 是门规范很宽松的语言，因此易于学习与使用，但也正是因为这样的「放任」才会出现了四分五裂的 Python 环境解决方案：

![](https://cdn.sspai.com/2022/09/28/article/27f83a110fd22e1730cbe80d8e3767ff?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

经过前一节的介绍，相信读者已经对「为什么需要有虚拟环境」这一问题有了基本的了解。

由于 Python 是门规范较为宽松的编程语言，而 Python 官方在早期对于虚拟环境或依赖管理并没有一个统一的解决方案，因此也就在民间出现了各种「非官方实现的」解决手段。

在 Python 社区生态里，我们可以有多种方式去创建一个干净虚拟环境，本小节笔者主要会简单介绍四个：

1.  venv
2.  virtualenv
3.  conda
4.  poetry

## venv

[venv](https://docs.python.org/zh-cn/3/library/venv.html) 是 Python 官方在 Python 3.3 版本内置的一个标准库模块，其主要功能移植自 Virtualenv（在下一小节中展开），让我们可以快速生成一个虚拟环境。现在我们只要下载的 Python 解释器版本在 3.3 及以上就能直接使用，而无需安装其他依赖。

使用它的方式十分简单，只需要通过使用对应版本的 Python 解释器并运行以下命令即可，比如这里我们以 Python 3.10 版本解释器为例：

```
$ python3.10 -m venv <your-virtual-environment-name>
```

通常我们在创建虚拟环境时需，要简单指定一个名称用以存放虚拟环境的内容，所以上述代码中的名称通常就保持和 `venv` 模块名称一致：

```
$ python3.10 -m venv venv
```

通过对应版本的 Python 解释器调用 `venv` 模块得到的虚拟环境，其版本也与解释器保持一致，也即在创建的 `venv` 目录中期 Python 解释器也为 3.10。并且在不指定目录的情况下，默认会在当前运行命令的目录中创建虚拟环境。

目前像 Pycharm 这样的 IDE 或 VS Code 类似的文本编辑器（需要安装插件）已经会正确识别到 `venv` 目录，并在运行代码时自动帮我们切换到虚拟环境中。

但如果我们是在终端上手动切换虚拟环境，那么就需要自己进行手动激活（Active）。由于 `venv` 模块创建的虚拟环境目录在 macOS 和 Windows 内容有所不同，所以就导致二者在激活虚拟环境时的命令也有所差异。

在 macOS 中，激活命令 `active` 会存放在同名的 `venv` 目录下的 `bin/` 目录中。但在使用前需要如下图所示，通过 Unix 中的 `chmod +x` 命令赋予激活命令执行权限。如果读者有像笔者一样事先对终端命令行界面进行了美化相关的设置，那么激活之后就能看到如图所示的环境名前缀（或后缀）提示符，这就表示我们已经成功切换到虚拟环境中；

![](https://cdn.sspai.com/2022/09/28/article/b4503099a78b04b86feef8fa4ea0af90?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

而在 Windows 系统 `bin/` 目录则为 `Scripts/` 目录，操作上同理。

后续我们陆续介绍到的虚拟环境管理工具，除了 IDE 帮我们自动切换之外，大部分时候如果是在终端运行，那么就需要自己通过对应的激活命令手动切换。

当然，如果我们想要退出虚拟环境，要么直接关闭 IDE，要么直接在终端命令行界面输入 `deactivate` 命令即可（或是同样关闭终端）。

不过使用 `venv` 模块存在一个令人头疼的问题：如果我们有多个项目，那么就会存在多个由 `venv` 创建的虚拟环境，因此我们无法做到**统一集中管理**。

所以，Python 社区的里的其他方案也意识到了这一点，也都具备了统一在一个命令中管理虚拟环境的功能。

## Virtualenv

[Virtualenv](https://virtualenv.pypa.io/en/latest/) 是 Python 社区一款老牌、成熟的虚拟环境管理工具，经过多个版本迭代也具备丰富的功能。并且自从 Python 3.3 版本开始，它的部分功能已也被集成到了 venv 标准库中，足见其对于 Python 虚拟环境管理工作贡献的份量如何。

从某些程度上来说，Virtualenv 和 venv 的功能十分类似，但 Virtualenv 在其官方文档中也指出了 venv 的不足之处：

> \*is slower（创建速度慢）

> \*is not as extendable（可扩展性差）

> \*cannot create virtual environments for arbitrarily installed python versions（无法创建任意 Python 版本的虚拟环境）

> \*is not upgrade-able via pip（无法通过 pip 进行升级）

> \*does not have as rich programmatic API（没有丰富的 API 编程方法扩展）

而这些不足之处在 Virtualenv 里都有了比较完善的解决方案。

不过由于 virtualenv 本质上属于一个第三方工具，因此我们要使用 virtualenv 首先就得通过 `pip` 命令安装它。这里的 `pip` 命令存在于在全局解释器中：

```
$ pip install virtualenv --user
```

安装成功后，它的用法也和 venv 十分类似，例如：

```
$ virtualenv myenv
```

上述命名会在当前路径中多出一个名为 `myenv` 的 Python 虚拟环境，使用它时我们同样可以像使用 venv 模块那样，通过同样的命令去激活环境。

不过这样本质上和我们用 venv 没有太大的区别。由于 Virtualenv 提供了可供扩展的 API 接口，因此我们通常还需要借助第三方的 Virtualenv 扩展，来帮助我们完成统一管理环境的操作，而 Virtualenvwrapper 就是这样一个能满足我们需求的第三方扩展库。

见名知意，[Virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) 实际是对 Virtualenv 进行包装（Wrap）的扩展库，我们在使用时一样需要通过 `pip` 工具进行安装安装。完成之后需要简单配置一下环境变量才能继续使用，以下是来自于官方的示例：

```
$ pip install virtualenvwrapper
...
$ export WORKON_HOME=~/Envs
$ mkdir -p $WORKON_HOME
$ source /usr/local/bin/virtualenvwrapper.sh
$ mkvirtualenv env1
```

上述的 `WORKON_HOME` 变量即指定的是统一存放虚拟环境的文件夹路径，创建之后再调用 Virtualenvwrapper 提供的初始化命令进行自动配置，之后就通过 `mkvirtualenv` 来创建相应的虚拟环境。

如果我们需要激活环境，则是需要通过 Virtualenvwrapper 帮我们设置好的 `worken` 命令来激活并切换：

```
$ workon env1
```

虽然 Virtualenvwrapper 是对 Virtualenv 的包装，但它并没有直接使用 Virtualenv 的命令，而是额外构造并提供了一批与虚拟环境相关的命令，读者在使用时最好需要参考其[文档](https://virtualenvwrapper.readthedocs.io/en/latest/command_ref.html)。

需要注意的是，Virtualenvwrapper **主要面向的是 macOS 和 Linux 操作系统**，而如果我们想要在 Windows 上使用这一工具，那就必须要再安装一个由其他开发者二次封装的 [virtualenvwrapepr-win](https://github.com/davidmarble/virtualenvwrapper-win)，但该库也仅支持在 Windows 上的 CMD 终端上使用，而不支持在 Powershell 上使用。

## Conda

相信有相当一大部分比例的已经事先学习过 Python 的新手，又或是如果从事数据分析、机器学习等数据科学相关工作的人，在学习或使用过程会经常接触到一个名为 [Anaconda](https://www.anaconda.com/) 的 Python 发行版本。在 Anaconda 中不仅内置了一个 Python 解释器，同时还内置了许多常用的数据科学软件包或工具。

但 Anaconda 为人所诟病的地方也在于它内置了太多东西，其中的大多数又用不到，导致最终体积较大，无异于让配置本就不富裕的雪上加霜。当然 Anaconda 还有另外一个精简版 [Miniconda](https://docs.conda.io/en/latest/miniconda.html)，它只包含了少量的一些依赖库或包，有效减轻了电脑磁盘的负担。

但不论是使用 Anaconda 还是 Miniconda，我们都会进一步用到它们当中的核心工具——[Conda](https://docs.conda.io/en/latest/)，一个跨平台的包（或库）和虚拟环境管理系统，它能轻易地帮我们构建程序运行所需的环境、依赖、更新等步骤。

![](https://cdn.sspai.com/2022/09/28/article/6e425fe5b14f4c2afd1f7d6809d95840?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

Conda 最大的一个优势就是它除了能安装 Python 依赖库或包之外，还能安装其他语言的一些依赖（比如 R 语言）；同时像 [Tensorflow](https://tensorflow.google.cn/)、[Pytorch](https://pytorch.org/) 这种业内常见的深度学习框架，往往会存在由 C/C++ 语言编写的部分，这些部分在安装时需要预先编译，而使用 Conda 安装时会自动连同已经事先编译好的二进制部分一起安装到虚拟环境中，避免了因操作系统不同而导致的编译问题。

不像 `pip` 或 `virtualenv` 这样的工具我们能够单独安装，要使用 Conda 我们通常会捆绑使用 Anaconda 或者 Miniconda，读者可以根据自己电脑的配置情况选择其一，具体的安装步骤可以直接参考[官方文档](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html#installing-conda-on-a-system-that-has-other-python-installations-or-packages)，上面有着详细的操作过程。

安装完成之后，当我们在终端命令行中输入如下命令，并能获取到帮助信息就说明安装成功：

```
$ conda --help
```

Conda 命令行工具集成了众多的功能，以至于我们可以用 Conda 命令行工具来完成大部分操作，读者可以参考 Conda 官方的 [Command reference](https://docs.conda.io/projects/conda/en/latest/commands.html) 来获取有关命令的详细说明。

比如使用 Conda 创建虚拟环境：

```
$ conda create --name myenv python=3.9
```

然后激活虚拟环境时只需要通过同样的 `active` 命令来操作即可：

```
$ conda active myenv
```

当然我们也可以将 Conda 作为 `pip` 工具来安装依赖：

```
$ conda install pandas seaborn
```

不过需要注意的是，尽管我们是在 Anaconda 或 Miniconda 中用到 Conda，但 Python 解释器本身就会自带 `pip` 工具。因此我们在使用 Conda 的过程中，切勿既通过 `conda install` 来安装依赖，又通过 `pip` 工具来安装依赖。

因为这就好比我们在驾驶时只用右脚来控制刹车和油门，而左脚要么控制离合器要么什么也不做一样，如果混用则极易引发事故。

这个道理也同样适用于 Conda，倘若我们将 `conda` 命令与 `pip` 命令作为安装依赖混用，那么 **有一定几率产生的依赖冲突的 BUG**。除此之外，通过 Conda 安装的版本有时候会比 `pip` 安装的版本会落后一些，并且 Conda 在安装前会检查环境依赖或更新情况，因此速度会相对较慢（不知道现在有没有改善）。

所以如果读者有打算使用 Conda 的想法，那么笔者**强烈建议读者只将其用来创建、管理虚拟环境**，而不是用来安装第三方依赖库或包。具体来说，就是通过 Conda 创建一个用于编写代码或程序的虚拟环境，然后再通过虚拟环境中的 `pip` 工具安装需要使用到的第三方依赖库或包即可。

使用 Conda 创建特定 Python 版本的虚拟环境时，Conda 会从默认的服务器上下载相关资源并构建。但由于 Conda 的服务器位于国外，在国内访问时会受到网络限制，这也进一步而影响到 Conda 的更新功能，因此在使用 Conda 之前我们最好像配置 `pip` 工具一样为 Conda 添加镜像源。

但好在清华大学开源软件镜像站包含 Conda 的镜像源，我们只需要按照 [《Anaconda 镜像使用帮助》](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/) 文档中的内容进行配置即可，本节就不过多赘述。

## Poetry

[Poetry](https://python-poetry.org/) 是一个面向未来的 Python 依赖管理和打包工具（笔者按：这里的「包」是指能够被其他人作为依赖安装的安装包，而非单独的应用程序），它支持了 Python 社区的 [PEP 518](https://www.python.org/dev/peps/pep-0518/) 提案里的 `pyproject.toml` 新标准文件，用以管理我们与 Python 项目有关的内容（比如我们在 Click 实践案例中见到的 `setup.py` 打包文件）。

有了 `pyproject.toml` 文件之后我们再也不需要在 Python 项目中包含额外的说明、许可证文件、依赖文件 `requirements.txt` 等，统统由这个文件统一管理。

Poetry 就像 Go 语言的 [Go mod](https://github.com/golang/go/wiki/Modules)、Rust 语言的 [Cargo](https://doc.rust-lang.org/cargo/) 工具一样，都是顺应着未来依赖管理更容易、开发更简单的趋势。目前已经可以看到有许多知名的 Python 开源项目都逐渐开始通过这一工具来管理，并且也都开始采用 `pyproject.toml` 文件。

虽然 Poetry 主要用于管理依赖和打包 Python 项目，但我们在使用 Poetry 时它也会自动创建一个包含了特定版本的 Python 解释器的虚拟环境，所以也可以算作是一个虚拟环境管理工具。

当然我们在使用 Poetry 时依旧需要通过 `pip` 命令对其进行安装：

```
$ pip install poetry

# macOS
$ pip3 install poetry
```

使用 Poetry 步骤其实也和笔者前面所介绍的其他虚拟环境管理工具类似，即在使用时需要像 Conda 那样通过命令行来操作：

```
$ poetry <command> ...
```

使用 Poetry 的方式很简单，对于我们个人来说通常都是几步：

1.  通过 Poetry 初始化项目。这里的初始化包括新建项目并初始化和对已创建项目初始化两种情况。
2.  根据实际情况选择对应版本的 Python 解释器。默认情况下跟 Poetry 所使用的解释器版本相一致。
3.  添加必要依赖（Required Dependencies）或开发依赖。
4.  通过 Poetry 运行或启动项目和我们的代码。

### 初始化项目

如果我们已经存在了项目文件夹，那么就可以再使用 `poetry init` 来为已存在的项目生成 `pyproject.toml` 文件。当然我们也可以使用 `poetry new` 来直接创建一个项目，不过得到的结构文件需要自己手动调整以符合需要：

```
$ mkdir myproject
$ cd myproject
$ poetry init
```

之后我们在 `myproject` 这个项目文件夹下，就可以看到 `pyproject.toml` 文件，其内容如下：

```
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = ""
authors = ["None"]

[tool.poetry.dependencies]
python = "^3.9"

[tool.poetry.dev-dependencies]

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

通过 `poetry` 初始化生成的 `pyproject.toml` 文件内容并不多，它会在我们后续添加相关依赖的时候被不断丰富。

所以一般情况下我们也不需要手动修改它，因为我们通过 `poetry` 命令行工具进行某些操作时，Poetry 会帮我们自动增删当中的内容。

需要注意的是：因为 Poetry 的大多数操作都是围绕 `pyproject.toml` 进行展开的，因此**一定要在包含该文件的目录环境中使用 Poetry 操作**，否则会报错。

### 选择相应版本的 Python 解释器

初始化项目之后 Poetry 还并未为我们构建一个 Python 的虚拟环境。因为它会在我们添加依赖的时候默认创建，而这个虚拟环境的 Python 解释器版本和调用 Poetry 时所用到的 Python 释器版本一致，因此通常我们并不需要额外指定。

但如果你有特殊需要，需要 Poetry 为项目构建一个使用指定版本 Python 解释器的虚拟环境，那么你可以使用 `poetry env use <special-version-python-interpreter-path>` 来进行构建。其中 `use` 后面输入的是指定版本的 Python 解释器路径。

但因为 `pyproject.toml` 中 `tool.poetry.dependencies` 部分指明了 `python="^3.9"` 参数，即该项目使用的 Python 解释器最低版本为 3.9，因此我们如果指定的特定版本不在兼容范围内，那就需要我们手动修改。这里直接使用一般的文本编辑器修改完成后保存即可。

比如我们目前需要使用系统里存在的 Python 3.7 解释器，那么所以需要将 `python="^3.9"` 改成 `python="^3.7"` 然后再指定解释器路径即可。

`^` 符号是一个特殊的版本限定符号，在 Poetry 官方文档的[这一节](https://python-poetry.org/docs/dependency-specification/#version-constraints)内容中可以找到关于这个及其他同类符号的所有说明。

在 Windows 下：

```
PS C:\Users\Linxiaoyue\Desktop\myproject> poetry env use 'D:\Program Files\Python\python37\python.exe'
```

在 macOS 或 Linux 下：

```
$ poetry env use /usr/local/python/python37/bin/python
```

指定完成并运行命令之后，我们就可以在终端命令行界面上看到 Poetry 帮我们创建了一个虚拟环境及其存放的路径。

### 管理依赖

如果我们没有指定特定版本的 Python 解释器，那么直到我们添加依赖时 Poetry 才会帮我们创建虚拟环境并将依赖添加到 `pyproject.toml` 文件中。

使用 Poetry 管理依赖比较简单，而且有点类似于 JavaScripts 的包管理工具 [Yarn](https://yarnpkg.com/)；但通常情况下我们要做的也就只有两个操作：**添加和移除**。

#### 配置镜像路径

和使用 `pip` 工具以及 Conda 类似，在安装依赖之前，我们还是需要简单地配置一下 Poetry 安装依赖的镜像地址，避免我们下载依赖时走的是国外镜像仓库的网络，以加快安装过程。

但默认情况下，如果我们已经有配置了 `pip` 的镜像，那么 Poetry 则会直接使用该镜像来加速依赖下载过程，这一步配置镜像的过程可以跳过；反之，我们需要在 Poetry 的全局配置中进行设置，配置文件的具体路径如下：

-   macOS/Linux：`~/Library/Application Support/pypoetry`
-   Windows：`C:\Users\<username>\AppData\Roaming\pypoetry`

进入到对应的文件夹后，就会看到一个名为 `config.toml` 的文件，此时我们可以用电脑中的编辑器打开它（推荐 VS Code 之类比较现代的编辑器），之后在当中我们可以选择相应的镜像源，这里我推荐使用豆瓣和清华的镜像源，任选其一即可，编辑完成后保存即可。

```
repositories.douban = "https://pypi.doubanio.com/simple"  # 豆瓣镜像源
repositories.tuna = "https://pypi.tuna.tsinghua.edu.cn/simple"  # 清华镜像源
```

关于 Poetry 配置文件的内容读者可以查阅 [Configuration](https://python-poetry.org/docs/configuration/) 一节文档。

#### 添加依赖：add

配置好镜像路径之后，我们就可以直接使用 `poetry add <package-name>` 就可以安装依赖：

```
poetry add requests
```

但默认情况下，使用 `poetry add` 添加的是项目的必要依赖，也即运行项目必须要安装的依赖。

如果在开发的时候我们不想导出某些在开发环境中使用的依赖，比如（Pytest 之类的第三方测试库），那么在添加依赖时需要使用 Poetry 为我们提供的开发环境依赖（Development Environment Denpendencies）选项 `-D`：

```
poetry add -D pytest
```

安装成功后我们打开 `pyproject.toml` 文件中的 `tool.poetry.dev-dependencies` 就会多出一行 `pytest="^6.2.3"`

```
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = ""
authors = ["None"]

[tool.poetry.dependencies]
python = "^3.7"
requests = "^2.25.1"

[tool.poetry.dev-dependencies]
pytest = "^6.2.3"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

当然我们也可以一次性添加多个依赖，和 `pip` 工具的使用方式类似，只要通过空格隔开即可：

```
poetry add -D black flake8 mypy
```

#### 移除依赖：remove

移除依赖的方法也很简单，我们只需要使用 `poetry remove <package-name>` 即可。

一旦运行上述命令，Poetry 会连同依赖的依赖一起移除，确保我们的虚拟环境里没有残留的第三方库或包，这一功能在 `pip` 工具中并未实现。

和 `add` 类似，默认是操作公共依赖，如果只是要移除开发环境依赖，一样需要加上 `-D` 选项：

```
poetry remove -D pytest
```

### 激活虚拟环境

使用 Poetry 激活虚拟环境也比较简单，它和我们前面说的 Conda 类似，即使用：

```
poetry shell
```

除此之外，如果我们没创建环境，那么这个命令同样会帮助我们创建一个新的虚拟环境，然后再切换到环境中。

当然我们也可以不用切换到虚拟环境中，直接使用相关命令执行对应的命令或代码，即使用 `poetry run <command>`，但这种情况往往**适用于命令较少**的情况下。比如现在我们的项目文件夹中已经存在了一个名为 `main.py` 文件，此时我们想要在 Poetry 为我们创建好的虚拟环境来运行它，那么就根据命令操作：

```
poetry run python main.py
```

这里在 `run` 之后需要接具体运行的命令行工具名称，之后才是参数。

### 导出依赖列表

由于我们使用 `pyproject.toml` 文件并由 Poetry 来帮助我们管理依赖，如果要是其他人也使用了 Poetry，那么在安装过程中倒是没什么问题，直接运行 `poetry install` 命令即可；但如果有的人仍使用传统的 `pip` 方式来安装依赖，那么我们为了兼顾别人，就可能需要额外导出一下我们现有的依赖文件，将其放入传统的 `requirements.txt` 文件中。

此时 Poetry 也给我们提供了一种导出的方式，即 `poetry export` 命令。这里我们需要加上 `-o` 选项输出到 `requirements.txt` 文件中，这个文件会自动在这个过程中被创建。

```
poetry export -o requirements.txt
```

默认情况下，Poetry 会连同依赖对应版本的校验哈希值一起导出，并且只导出全局依赖；因此我们可以分别通过加上 `--without-hashes` 和 `--dev` 选项来导出适合的依赖文件：

```
# 不导出 Hash 值
$ poetry export --without-hashes

# 导出开发依赖
$ poetry export --without-hashes --dev
```

以上就是 Poetry 最常见的一些用法，更多关于 Poetry 的操作和用法可以进一步参考 Poetry 的[官方文档](https://python-poetry.org/docs/)。

## 作为一个 Python 工程师，我是如何选择？

通过以上的介绍，可能读者已经对 Python 社区中虚拟环境管理的方案已经有了一定的认识。读者可能也想知道像笔者这样作为一个日常使用 Python 的开发工程师，会选择上述哪种方案的以及当中的优劣如何。

因此在本章这最后的一小节里，笔者主要谈谈不论是在工作还是个人项目中，使用上述虚拟环境管理工具时的体验感受以及使用建议。

### venv：快速实验与使用的好帮手

如果读者不是专门的 Python 开发工程师，那通常也很少接触到大型的或结构固定的项目代码，更多时候可能也只是出于个人兴趣或是在工作中将 Python 作为脚本使用。

因此如果不想太过折腾虚拟环境，又想避免把当前操作系统中的 Python 环境搞乱，那么最好的方式就是每次实践的过程中，用 Python 解释器自带的 `venv` 模块来快速生成一个干净的 Python 虚拟环境。

默认情况下它会直接根据当前所使用的 Python 解释器进行克隆，同时也会将生成的虚拟环境文件夹放在当前的文件路径中。比如我们现在桌面上有一个用于练习的 `lab` 文件夹当中存放着我们的代码，那么通过解释器自带的 `venv` 模块创建一个同名的 `venv` 虚拟环境文件夹之后，就会可以基于当中命令来运行我们的代码：

```
lab
 ├── main.py
 └── venv
     ├── bin
     ├── include
     ├── lib
     └── pyvenv.cfg
```

注：venv 在 Windows 下创建的目录可能会和上述内容有所不同，即用一个名为 Scripts 的文件夹来存放相关命令而非上述内容中的 bin 文件夹。

`venv` 模块的缺点在于创建后的虚拟环境是完全独立的克隆体，无法被很好地统一管理，年代久远的情况下，可能就散落在自己都不曾注意的犄角旮旯文件夹中占用磁盘空间。但换句话来，说如果我们也只要将存放了 `venv` 生成的虚拟环境的项目文件夹删除，那么虚拟环境也就会被清理。

不过好处在于目前大部分的 IDE 或编辑器——比如 Pycharm、VS Code 等——都支持探测 `venv` 所生成的虚拟环境的功能。如果发现当前项目文件夹中存在了由 `venv` 模块创建的虚拟环境，那么也会自动将执行代码需要的 Python 解释器命令切换为虚拟环境中的解释器命令。

### Virtualenv：有口皆碑的传统厂牌

Virtualenv 作为历史悠久、到目前也一直维护更新的虚拟环境的创建工具，一直都是有口皆碑且在使用中也是「稳如泰山」。虽然 `venv` 模块脱胎于 Virtualenv，但却不是完全移植，因此缺乏某些 Virtualenv 才有的功能，比如不能手动基于其他 Python 版本来创建虚拟环境。

因此，不论是刚接触 Python 的新手，还是在日常的开发中，使用 Virtualenv 这样老牌的工具来创建虚拟环境一般也没有什么问题。因为 Virtualenv 本身就历史悠久，已经更新迭代多个版本，其 API 方法或使用方式也相对稳定，对于已经使用过该工具来管理的开发者来说无疑是降低学习成本的好事情。即便在使用 Virtualenv 的过程中出现了一时无法解决的问题，那么在网上也能找到相对多的参考资料提供相应的解决方案。

Virtualenv 通常需要搭配 `virtualenvwrapper` 这一扩展工具来使用，特别是当我们系统中存在多个虚拟环境时，并且我们需要经常在这些虚拟环境中进行切换的场景时尤为有用。通过 `virtualenvwrapper` 我们可以快速实现查看当前虚拟环境以及创建、删除虚拟环境等，不过 `virtualenvwrapper` 在前期配置上会稍微有些繁琐。

### Conda：数据科学和离线 Linux 环境最好的选择

对于数据科学，尤其是会涉及到机器学习、深度学习领域的使用者而言，Conda 应该是最好的选择。

因为不论是 Anaconda 还是 Miniconda 它们都是在 Conda 基础之上发展而来，都已经事先内置了一系列开箱即用的依赖库或第三库，对于新手而言比较友好，一定程度上可以规避掉安装依赖时的某些问题。并且 Conda 最大的优势在于，如果所使用的依赖库核心会使用类似于 C 或 C++ 语言的额外代码比如 [PyTorch](https://pytorch.org/) 或 [TensorFlow](https://www.tensorflow.org/?hl=zh-cn) 这样深度学习框架，那么通常在 Conda 的官方仓库上都有事先已经编译并被打包好的版本，可以直接通过命令一键安装而无须使用者手动编译。

同时，如果要在**无网络环境**的 Linux 服务器上使用 Python，那么 Conda 毫无疑问是有着绝对优势。

这里的无网络环境也就是所谓的「Offline」离线环境，这通常在 **信息安全保密性要求较高** 的企业或机构部门中最为常见。如果我们需要在该环境中安装东西，就必须事先在自己的电脑上提前下载并传入类似于光盘、U 盘之类的传输介质之中，导入至指定的文件路径再进行操作，十分繁琐。

而 Python 官方目前仅提供了 Windows 和 macOS 两个版本的便捷安装包，而 Linux 版本则不存在便捷安装包一说，需要使用者自己手动编译。除此之外，由于 Python 的官方版本解释器是由 CPython 来实现（即由 C 语言实现），编译时需要涉及到众多的动态链接库，但凡缺少一个最后都会导致某个功能无法正常使用。

所以这时候 Conda 也再次派上用场。

我们只需要通过 Conda 来创建指定 Python 版本的虚拟环境，然后将各种依赖事先安装，最后直接将整个环境打包成压缩包，最后再走一遍传输的流程并在 Linux 服务器上直接解压并使用即可。

通过 Conda 所创建的虚拟环境其实也就对应了一个 Python 解释器，并且它会自动为我们链接运行解释器所需要的一些动态链接库，这就是为什么我们可以直接拷贝解压之后就可以直接使用。

### Poetry：标新立异的个人首选

如果是**现在**个人项目或者用于开源项目，那么目前被广泛的使用的 Poetry 可以算是依赖与虚拟环境管理的首选。比如我们在实践案例中用到的 FastAPI 就主要使用 Poetry。

一方面 Poetry 会在首次初始化时自动为项目创建一个虚拟环境，之后我们只需要在 IDE 或编辑器中指定该环境的解释器即可；另一方面，Poetry 不同于单一的 `pip` 工具，它除了提供依赖解析与安装功能之外，还支持对项目代码进行打包，而与之相关的内容或参数都统一放到来自于 PEP 518 这一 Python 增强提案的 `pyproject.toml` 配置文件中。

同时，Poetry 在安装和使用上类似于 [Yarn](https://yarnpkg.com/) 这一 JavaScripts 包管理工具，不论是添加、更新还是删除依赖，都会生成一个名为 `poetry.lock` 的版本锁或依赖锁文件：

```
# poetry.lock

[[package]]
name = "click"
version = "8.1.2"
description = "Composable command line interface toolkit"
category = "main"
optional = false
python-versions = ">=3.7"

[package.dependencies]
colorama = {version = "*", markers = "platform_system == \"Windows\""}

...
[metadata.files]
click = [
    {file = "click-8.1.2-py3-none-any.whl", hash = "sha256:24e1a4a9ec5bf6299411369b208c1df2188d9eb8d916302fe6bf03faed227f1e"},
    {file = "click-8.1.2.tar.gz", hash = "sha256:479707fe14d9ec9a0757618b7a100a0ae4c4e236fac5b7f80ca68028141a1a72"},
]
colorama = [
    {file = "colorama-0.4.4-py2.py3-none-any.whl", hash = "sha256:9f47eda37229f68eee03b24b9748937c7dc3868f906e8ba69fbcbdd3bc5dc3e2"},
    {file = "colorama-0.4.4.tar.gz", hash = "sha256:5941b2b48a20143d2267e95b1c2a7603ce057ee39fd88e7329b0c292aa16869b"},
]
...
```

目前大部分编程语言的依赖管理工具都会自带和 `poetry.lock` 文件这样的版本锁机制，其目的在于能够将当前的依赖版本记录下来（这个过程主要由工具自动生成对应的校验内容），因而不论是在与别人协作、使用 CI/CD 工具甚至部署到生产环境时都 **能确保所有使用的依赖版本及内容一致**，进一步减少因环境不一致导致问题排查困难的问题。

虽然 Poetry 是一个不同于以往 venv、Virtualenv 以及 Conda 的新模式，并且到目前为止（截止至 2022 年 7 月 28 日）也已经演进到了 1.1.x 版本，但从 Github 项目仓库的 Issue 模块（笔者按：指与项目有关的问题反馈、功能要求、讨论的页面板块）未解决的数量来看，Poetry 依旧有着这样或那样的问题或小毛病，例如一直为人诟病的依赖解析时间过长的问题。

因此如果是在公司内部项目中，如果部门或小组本就有了被长期使用的工具，那么就最好不要与其他人背道而驰地使用 Poetry 来「标新立异」。但除此之外的情况，笔者却十分鼓励正在学习的读者去踊跃尝试 Poetry，毕竟个人往往都是在折腾中学习、成长。同时，Poetry 的命令并不算多，不论是学习还是使用的成本相对而言没有那么高，学习它也同样是为了与时俱进、拥抱未来。

## 结语

Python 社区里各种虚拟环境与依赖管理的工具层出不穷也一直是为人所诟病的地方，并且和 Python 一样有着悠久历史的编程语言比如 JavaScript（Node.js）、Java 等也会面临同样的问题。因此像 Rust 这样后起之秀的编程语言，就积极吸取了以往的经验教训，在设计上会将环境、依赖管理等工具统一整合到一起。

除了本章所介绍的工具之外，在 Python 社区中还存在 [pipx](https://github.com/pypa/pipx)、[pyenv](https://github.com/pyenv/pyenv)、[pipenv](https://pipenv.pypa.io/en/latest/) 等等功能类似的工具，这也侧面反映出 Python 社区在依赖管理方案上群雄割据的混乱情况。

在 2018 年出现的 Python 增强提案 [PEP 582](https://peps.python.org/pep-0582/) 中又出现了一种和本文内容虚拟环境管理有所不同的一种依赖管理方式，使得 Python 项目再也不需要虚拟环境，而是在项目目录中存在一个名为 `__pypackages__` 的特殊文件夹即可直接使用相应的 Python 解释器执行代码；其模式类似于前端工具 [npm](https://www.npmjs.com/) 的 `node_modules` 文件夹，会在每次添加依赖时自动在本地目录中下载依赖。

所以在 PEP 582 提案基础上又出现了类似于 [PDM](https://github.com/pdm-project/pdm) 的工具。

但不论这些工具如何的丰富，我们在对它们进行选择时，除了考虑场景之外，更多时候还主要是以稳定性为主。