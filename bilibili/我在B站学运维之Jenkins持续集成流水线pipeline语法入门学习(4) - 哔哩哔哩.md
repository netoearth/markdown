**本章目录:**

0x00 前言简述

-   Pipeline 介绍
    
-   Pipeline 基础知识
    
-   Pipeline 扩展共享库
    
-   BlueOcean 介绍
    

0x01 Pipeline Syntax

-   (0) Groovy Basic Syntax
    
-   (1) Scripted Pipeline Syntax
    

-   Hello-World 实践
    
-   变量名-Identifiers
    
-   字符串-String
    
-   数字 - Numbers
    
-   列表-List
    
-   字典 - Maps
    
-   条件语句 - Condition
    
-   异常 - Exception
    
-   函数 - Functions
    
-   语法总结
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

## 0x00 前言简述

## Pipeline 介绍

**Q: 什么是 Pipeline？**

> 答: Pipeline（流水线）是 Jenkins 2.0 的精髓它基于Groovy语言实现的一种DSL(领域特定语言)，简而言之就是一套运行于Jenkins上的工作流框架，用于描述整条流水线是如何进行的。  
> 它将原本独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的`复杂流程编排与可视化`。

**Q: 什么是DSL?**

> 答: DSL即 `(Domain Specific Language)` 领域专用语言，专门针对一个特定的问题领域，具有建模所需的语法和语义的语言。在与问题域相同的抽象层次对概念建模。  
> DSL  是 Jenkins 服务特有的一个语言，底层通过 Groovy 编程语言来实现。在使用过程中，可以很好的结合 Groovy。Jenkins  Job DSL Plugin 提供了丰富的API，我们可以通过这些API实现对 Jenkinis 中View、Job 等管理。  
> Tips: Jenkins 内置了 Groovy 的引擎，我们可以通过 Groovy 编程语言在 DSL API 中添加逻辑编程。

**Q: 什么是 Groovy 语言**  
答:  Groovy 是 Apache 旗下的一门基于 JVM 平台的动态/敏捷编程语言，在语言的设计上它吸纳了 Python、Ruby 和  Smalltalk  语言的优秀特性，语法非常简练和优美，开发效率也非常高（编程语言的开发效率和性能是相互矛盾的，越高级的编程语言性能越差，因为意味着更多底层的封装，不过开发效率会更高，需结合使用场景做取舍）

**Tips: PipeLine在Jenkins中的发展历史**

-   1.Jenkins 1.x 支持 Pipeline ，只不过是通过页面手动配置流水线。
    
-   2.Jenkins 2.x 开始支持 pipeline as code ,可以通过代码来配置流水线了。
    

**Q: 为什么要使用Pipeline?**

> 1.Pipeline是Jenkins2.X的最核心的特性，帮助Jenkins实现从`CI到CD`与`AutoDevOps`的转变;  
> 2.Pipeline是一组插件它可以让Jenkins可以实现持续交付 Pipeline的落地和实施。  
> 3.Pipeline提供了一组可扩展的工具，通过`Pipeline Domain Specific Language(DSL) syntax`可以达到 Pipeline as Code（Jenkinsfile存储在项目的源代码库）的目的。

Tips: 流水线的内容包括执行编译、打包、测试、输出测试报告等步骤。  
Tips: 持续交付`Pipeline (CD Pipeline)`是将软件从版本控制阶段到交付给用户或客户的完整过程的自动化表现, 软件的每一次更改（提交到源代码管理系统）都要经过一个复杂的过程才能被发布。

**Pipeline五大特性（优点）**

-   代码: Pipeline以代码的形式实现，通常被检入源代码控制，使团队能够编辑、审查和迭代其CD流程。
    
-   可持续性：Jenklins重启或者中断后都不会影响Pipeline Job。
    
-   停顿：Pipeline可以选择停止并等待构建人员的输入或批准，然后再继续Pipeline运行。
    
-   多功能：Pipeline支持现实世界的复杂CD要求，包括fork/join子进程，循环和并行执行工作的能力
    
-   可扩展：Pipeline插件支持其`DSL`的自定义扩展以及与其他插件集成的多个选项。
    

Tips: 它实现持续集成与部署、节省产品发布时间、优化部署策略、节省人力成本、以及自动化脚本复用等等;

**Q: 怎样安装Pipeline插件?**

> 答: 熟话说工欲善其事必先利其器，第一步当然需要安装Jenkins使用Pipeline所需的插件;

Jenkins pipeline 相关插件安装: 打开 Jenkins 找到 `【系统管理】->【插件管理】->【可选插件】` 然后在搜索框输入 `Pipeline`

```
Pipeline 命令行接口 杂项 代理启动器和控制器 构建触发器  2.6  2 年 3 月 ago
# 说明：一套插件，让您编排自动化，简单或复杂。更多细节请参阅Jenkins的 Pipeline代码。
```

![WeiyiGeek.Pipeline-Plug](https://i0.hdslb.com/bfs/article/843e91320c21ddb5048adbd27b35980b1b808f1a.png@942w_1235h_progressive.webp)

## Pipeline 基础知识

**基础说明:**

> -   Pipeline 脚本是由 Groovy 语言结合 DSL 语言实现的。
>     
> -   Pipeline 支持两种语法：`Declarative Pipeline (声明式 - 2.5 引入)` 和 `Scripted Pipeline (脚本式)`语法
>     
> -   Pipeline 也有两种创建方法：
>     
> 
> -   方式1、在 Jenkins 的 Web UI 界面中输入脚本；
>     
> -   方式2、通过创建一个 Jenkinsfile 脚本文件(`Groovy 语言结合 DSL 开发`)放入项目源码库中 (`推荐在 Jenkins 中直接从源代码控制(SCMD) 中直接载入 Jenkinsfile Pipeline`)
>     

**语法差异:**  
描述:  最初创建 Jenkins Pipeline 时 Groovy  语言被选为基础。Jenkins长期以来一直提供嵌入式Groovy引擎，以为管理员和用户提供高级脚本功能。另外Jenkins  Pipeline的实现者发现Groovy是构建现在称为"脚本 Pipelin" DSL的坚实基础。

由于它是功能齐全的编程环境，因此脚本化  Pipeline为Jenkins用户提供了极大的灵活性和可扩展性。Groovy学习曲线通常不是给定团队的所有成员所希望的，因此创建了声明式  Pipeline，以为编写Jenkins Pipeline提供更简单，更自以为是的语法。

两者基本上是下面的相同 Pipeline子系统。它们都是“ Pipeline作为代码”的持久实现。他们都可以使用内置在Pipeline中或由插件提供的步骤。两者都可以利用 共享库

但是它们的区别在于语法和灵活性。声明性限制使用更严格和预定义的结构为用户提供的功能，使其成为更简单的连续交付  Pipeline的理想选择。脚本化脚本提供的限制非常少，以至于对结构和语法的唯一限制往往是由Groovy本身定义的，而不是由任何特定于  Pipeline的系统定义的，因此，它成为高级用户和要求更复杂的用户的理想选择。顾名思义，声明性流水线鼓励使用声明性编程模型，而脚本  Pipeline 遵循更强制性的编程模型。

**Q: 选择`Declarative Pipeline`还是`Scripted Pipeline`?**

> 答: 最开始`Pipeline plugin`仅支持`Scripted Pipeline`一种脚本类型,而 `Declarative Pipeline` 为Pipeline plugin在2.5版本之后新增的一种脚本类型与原先的`Scripted Pipeline`一样都可以用来编写脚本。
> 
> 由于在我们使用BlueOcean流水线UI插件后，Declarative Pipeline 与 BlueOcean 脚本编辑器是可以兼容使用，`并且在eclarative Pipeline中，也是可以内嵌Scripted Pipeline代码的`，所以通常建议使用 `Declarative Pipeline` (`英 /dɪˈklærətɪv/`)的方式进行编写

**Declarative pipeline和Scripted pipeline的比较?**

-   1.共同点:
    

-   声明式和脚本式流水线都是 DSL 语言，用来描述软件交付流水线的一部分。
    
-   两者都能够使用pipeline内置的插件或者插件提供的step步骤部分。
    
-   两者都可以利用共享库扩展。
    

-   2.区别: 两者不同之处在于语法和灵活性;
    

-   Declarative pipeline 语法更严格 (`例如必须以 pipeline 关键词打头`)，有固定的组织结构但更容易生成代码段，所以它成为用户更理想的选择。
    
-   Scripted pipeline 语法更加灵活，因为Groovy本身只能对结构和语法进行限制，对于更复杂的pipeline来说，用户可以根据自己的业务进行灵活的实现和扩展。
    

Tips: 脚本式语法的确灵活、可扩展，但是也意味着更复杂。而声明式语法供更简单、更结构化（more opinionated）的语法 (`模块化的感觉`)。

## Pipeline 扩展共享库

描述: 由于流水线被组织中越来越多的项目所采用，常见的模式很可能会出现在多个项目之间共享流水线, `共享流水线有助于减少冗余并保持代码 “DRY（Don’t Repeat Yourself）”`。

**Q: 如何定义共享库?**

> 答: 我们将一些通用的代码或者代码包，封装定义为底层代码库，方便流水线创建。

特定的目录结构:

```
(root)
+- src                     # Groovy source files
|   +- org
|     +- foo
|       +- Bar.groovy      # for org.foo.Bar class
+- vars
|   +- foo.groovy          # for global 'foo' variable
|   +- foo.txt             # help for 'foo' variable
+- resources               # resource files (external libraries only)
|   +- org
|       +- foo
|           +- bar.json    # static helper data for org.foo.Bar
```

**目录结构解析:**

-   1、src 目录应该看起来像标准的 Java 源目录结构。当执行流水线时，该目录被添加到类路径下。
    
-   2、vars  目录定义可从流水线访问的全局变量的脚本。 每个 \*.groovy 文件的基名应该是一个 Groovy (~ Java) 标识符, 通常是  驼峰命名法（camelCased）。 匹配 \*.txt, 如果存在可以包含文档, 通过系统的配置标记格式化从处理 (所以可能是 HTML,  Markdown 等，虽然 txt 扩展是必需的)。这些目录中的 Groovy 源文件 在脚本化流水线中的 “CPS  transformation” 一样。
    
-   3、resources 目录允许从外部库中使用 libraryResource 步骤来加载有关的非 Groovy 文件。 目前，内部库不支持该特性。
    
-   4、根目录下的其他目录被保留下来以便于将来的增强。
    

**Q: 如何将将共享库设置为全局共享库？**

> 描述: 在Jenkins 管理页面中的 “Configure System” 页面中的 “Global Pipeline Libraries” 中设置全局共享库。

**Q: 如何使用封装的代码库**

> 答: Jenkinsfile 文件中需要使用 @Library 注解，指定库的名字。另外关于代码库的动态加载、版本管理和检索方式等，请见官网。

**Q: 如何编写自己的 Jenkins 共享库，共享库中的变量作用域?**

> 答: 其他关于写库的访问步骤、定义全局变量 请见官网。

## BlueOcean 介绍

**Q: 什么是BlueOcean?**

> A: BlueOcean 重新考虑了 Jenkins 的用户体验而重新设置UI界面,从而更加直观的展现Pipeline各流程执行情况;  
> BlueOcean由Jenkins Pipeline设计，但仍然兼容自由式工作，减少了团队成员的混乱，增加了清晰度。

**Q: 为啥要使用BlueOcean?**

> 连续交付（CD）Pipeline的复杂可视化，允许快速和直观地了解Pipeline的状态。  
> Pipeline编辑器通过引导用户直观和可视化的过程创建Pipeline，使创建Pipeline平易近人。  
> 个性化，以适应团队每个成员的角色需求。  
> 需要干预和/或出现问题时确定精度。BlueOcean显示了Pipeline需要注意的地方，便于异常处理和提高生产率。  
> 用于分支和拉取请求的本地集成可以在GitHub和Bitbucket中与其他人进行代码协作时最大限度提高开发人员的生产力。

**Q: 如何安装BlueOcean?**

> A: 同样是在插件中搜索 ”Blue Ocean“ 下载安装即可  
> Blue Ocean - 外部工具集成 用户界面 - BlueOcean Aggregator - 1.24.3 2 月 5 天 ago

## 0x01 Pipeline Syntax

## (0) Groovy Basic Syntax

描述: 我们前面说过不管是声明式还是脚本式都是基于Groovy语言,所以学习 Groovy 基础知识是必须的。

-   1.虽然Groovy同时支持静态类型和动态类型，但是在定义变量时，在Groovy中我们习惯使用def关键字
    

```
def x="abc"
def y=1
```

-   2.不像 Java语法语句,Groovy语句最后的分号不是必需的。
    
-   3.Groovy中的方法调用可以省略括号，比如System.out.println "Hello world"。
    

```
System.out.println x
println t
```

-   4.支持单引号、双引号。双引号支持插值(变量)，单引号不支持。
    
-   5.支持三引号。三引号分为三单引号和三双引号。它们都支持换行，区别在于只有三双引号支持插值(变量)。
    

```
def var = """
This is Variable!
<br>
Test defiend
"""
```

-   6.支持函数。
    

```
def getSecure(String Ticket_Token) {
  def token = "Ticket Token is " + Ticket_Token
  return token
}

println getSecure("weiyigeek")  //Ticket Token is weiyigeek
```

-   7.支持闭包。
    

```
// # 闭包的定义方法：
def codeBlock = {print "hello world!"}
// codeBlock() //

// # 闭包的另类用法:
// 定义一个stage函数
def stage(String name, closue) {
  println name
  def closue() {
    println "闭包调用的 closue function!"
  }
}
stage("stage name",{println "closue"})
```

-   8.支持类定义和实例化。
    

```
class Greet {
  def name
  Greet(who) { name = who[0].toUpperCase() + who[1..-1] }
  def salute() { println "Hello " + name + "!" }
}

g = new Greet('world')  // create object
g.salute()   // Hello World!
```

## (1) Scripted Pipeline Syntax

描述: Scripted Pipeline 是基于 groovy 的一种 `DSL` 语言相比于 Declarative pipeline，它为jenkins用户提供了更巨大的灵活性和可扩展性。

**Scripted Pipeline 基础结构说明:**

-   Node：节点，一个 Node 就是一个 Jenkins 节点，Master 或者 Agent，是执行 Step 的具体运行环境，比如我们之前动态运行的 Jenkins Slave 就是一个 Node 节点
    
-   Stage：阶段，一个 Pipeline 可以划分为若干个 Stage，每个 Stage 代表一组操作，比如：Build、Test、Deploy，Stage 是一个逻辑分组的概念，可以跨多个 Node
    
-   Step：步骤，Step 是最基本的操作单元，可以是打印一句话，也可以是构建一个 Docker 镜像，由各类 Jenkins 插件提供，·`比如命令：sh 'make'，就相当于我们平时 shell 终端中执行 make 命令一样。` (注意:此处Step不是cripted Pipeline关键字而是代表一条执行语句)
    

**Scripted Pipeline 语法示例:**

```
// Jenkinsfile (Scripted Pipeline)
// #结构1
node {
    // @变量定义
    def mvnHome
    // #结构2
    stage('Preparation') {
      // # 结构3 - 它就是 Step 基本操作单元
      echo "Scripted Pipeline"
    }
}
```

Tips : 注释（Comments）和Java一样，`支持单行（使用//）、多行（/* */）和文档注释（使用/** */）`。

## Hello-World 实践

Step 1.在Jenkins的WEB UI -> 新建任务 -> simple-pipeline-demo 任务名称 -> 选择流水线 -> 确定

Step 2.在 Dashboard -> simple-pipeline-demo -> 流水线 -> 可以选择pipeline script（或者直接从scm拉取Jenkinsfile）此处为了演示只是简单的了解 -> 应用保存

```
# Scripted Pipeline 脚本式
node {
  stage('Clone') {
    echo "1.Clone Stage"
  }
  stage('Test') {
    echo "2.Test Stage"
  }
  stage('Build') {
    echo "3.Build Stage"
  }
  stage('Deploy') {
    echo "4. Deploy Stage"
  }
}
```

![WeiyiGeek.Create PipeLine 流水线项目](https://i0.hdslb.com/bfs/article/807e0845f74bb2921cb3c6c04aec3a32aed14f88.png@942w_329h_progressive.webp)

Step 3.立即构建 -> 查看阶段视图 (或者利用blue-Ocean插件)进行更加直观的查看 -> 观察构建的日志信息

```
Started by user admin
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/simple-pipeline-demo
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Clone)
[Pipeline] echo
1.Clone Stage
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] echo
2.Test Stage
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] echo
3.Build Stage
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy)
[Pipeline] echo
4. Deploy Stage
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS

WeiyiGeek.jenkins流水线执行结果与blue-Ocean
```

![WeiyiGeek.jenkins流水线执行结果与blue-Ocean](https://i0.hdslb.com/bfs/article/764a812a539f1563ddec41bb4ca5516473dc6217.png@942w_363h_progressive.webp)

PS : 你可以选择使用BlueOcean或者jenkins原生的流水控制台展示两则并不冲突，但是需要注意Scripted pipeline不完全兼容`BlueOcean`;

## 变量名-Identifiers

描述: 标识符（Identifiers）也称变量名, `以字母、美元符号$或下划线_开始`，不能以数字开始。  
例如以下是可用的标识符：

```
def name
def item3
def with_underscore
def $dollarStart
```

以下是不可用的标识符：

```
def 3tier // 不能以数字开始
def a+b // "+"号是非法字符
def a#b // #号也不是可用的字符
```

Tips : 在点号后是可以使用关键字作为标识符时产生`org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed:`错误;

```
foo.as
foo.assert
foo.break
foo.case
foo.catch
```

## 字符串-String

描述: 在Groovy中字符串有两种类型，一种是Java原生的`java.lang.String`；另一种是`groovy.lang.GString`，又叫插值字符串(interpolated strings)。

-   (1) 单引号字符串（Single quoted string）  
    在Groovy中，使用单引号括住的字符串就是java.lang.String，不支持插值：
    

```
def name = 'yjiyjgie'
println name.class // class java.lang.String
```

-   (2) 三单引号字符串（Triple single quoted string）  
    使用三单引号括住字符串支持多行，也是java.lang.String实例，在第一个’‘’起始处加一个反斜杠\\可以在新一行开始文本：
    

```
def strippedFirstNewline = '''line one
line two
line three
'''

// 可以写成下面这种形式，可读性更好

def strippedFirstNewline = '''\
line one
line two
line three
'''
```

-   (3) 双引号字符串（Double quoted string）  
    如果双引号括住的字符串中没有插值表达式（interpolated expression），那它就是java.lang.String；如是有插值表达式，那它就是groovy.lang.GString：
    

```
def normalStr = "yjiyjige" // 这是一个java.lang.String

def interpolatedStr = "my name is ${normalStr}" // 这是一个groovy.lang.GString
```

-   (4) 字符串插值（String interpolation）  
    在Groovy所有的字符串字面量表示中，除了单引号字符串和三单引号字符串，其他形式都支持字符串插值。字符串插值也即将占位表达式中的结果最终替换到字符串相应的位置中：
    

```
def name = 'Guillaume'          // a plain string
def greeting = "Hello ${name}" // name变量的值会被替换进去
assert greeting.toString()  == 'Hello Guillaume'

//当使用点号表达式时，可以只用$代替${}：
def person = [name: 'Guillaume', age: 36]
println "$person.name is $person.age years old"
```

补充说明:

```
// 插值占位符中还支持闭包，而闭包的一个好处是惰性求值（lazy evaluation）：
def number = 1
def eagerGString = "value == ${number}" // 普通形式
def lazyGString = "value == ${-> number}" // 这是一个闭包

println eagerGString // == "value == 1"
println lazyGString  // == "value == 1"

number = 2
println eagerGString // == "value == 1" // eagerGString已经被固定下来了
println lazyGString  // == "value == 2" // lazyGString的值会被重新计算
```

Tips: GString与String的hashCode是不一样的即使他们最终结果一样。所以在Map中不应该用GString去做元素的Key，而又使用普通的String去取值;

```
// 当一个方法的需要一个java.lang.String变量，而我们传递的是一个groovy.lang.GString实例时，GString的toString方法会被自动调用，看起来像我们可以直接将一个GString赋值给一个String变量一样。

def key = "a"
def m = ["${key}": "letter ${key}"] // key类型是一个GString
assert m["a"] // == null // 用一个普通String类型的key去取值取代的值为null 而并非 letter a
```

Tips : 对于输出对象带有指定方法时如有需要拼接其它字符串需要以`${对象.方法}`进行包含;

```
def number = 3.14
println "$number.toString()" // 这里会报异常,因为相当于"${number.toString}()"
println "${number.toString()} -- Other String、" // 这样就正常了
```

## 数字 - Numbers

描述: 当使用def指明整数字面量时，变量的类型会根据数字的大小自动调整：

```
// 如果要强制指明一个数字的字面量类型，可以给字面量加上类型后缀：
BigInteger 使用G或g
Long 使用L或l
Integer 使用I或i
BigDecimal 使用G或g
Double 使用D或d
Float 使用F或f

def a = 1
assert a instanceof Integer

// Integer.MAX_VALUE
def b = 2147483647
assert b instanceof Integer


// Integer.MAX_VALUE + 1
def c = 2147483648
assert c instanceof Long


// Long.MAX_VALUE
def d = 9223372036854775807
assert d instanceof Long

// Long.MAX_VALUE + 1
def e = 9223372036854775808
assert e instanceof BigInteger
```

Tips : 为了精确地计算小数，在Groovy中使用def声明的小数是BigDecimal类型的：

```
def decimal = 123.456
println decimal.getClass() // class java.math.BigDecimal
```

## 列表-List

描述：默认情况下Groovy的列表使用的是`java.util.ArrayList`，用中括号\[\]括住，使用逗号分隔：

```
def numbers = [1, 2, 3]
println numbers.getClass() # // class java.util.ArrayList

# 如果要使用其它类型的列表（如：LinkedList）可以使用as操作符或显式分配给一个指定类型的变量：

def arrayList = [1, 2, 3]  # // 默认类型
# assert arrayList instanceof java.util.ArrayList

def linkedList = [2, 3, 4] as LinkedList # // 使用as操作符
# assert linkedList instanceof java.util.LinkedList

LinkedList otherLinked = [3, 4, 5]         # // 显式指明类型
# assert otherLinked instanceof java.util.LinkedList
```

Groovy重载了列表的\[\]和<<操作符，可以通过List\[index\]访问指定位置元素，也可以通过`List << element`往列表末尾添加元素：

```
def letters = ['a', 'b', 'c', 'd']
assert letters[0]  //  == 'a'
assert letters[1]  // == 'b'
assert letters[-1] // == 'd' 从后面访问 倒数第一个
assert letters[-2] // == 'c' // 从后面访问 倒数第二个


// 元素修改
letters[2] = 'C'
assert letters[2]   //  == 'C'

letters << 'e' // 往最后面添加元素
assert letters[4] == 'e'
assert letters[-1] == 'e'
assert letters[1, 3] == ['b', 'd']      // 提取指定元素
assert letters[2..4] == ['C', 'd', 'e'] // 支持范围（ranges）操作


// 二维列表
def multi = [[0, 1], [2, 3]]
assert multi[1][0] == 2
```

## 字典 - Maps

描述: Groovy使用中括号\[\]来定义字典，元素需要包含key和value使用冒号分隔，元素与元素之间用逗号分隔：

```
// key部分其实是字符串
def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']
assert colors['red'] == '#FF0000' // 使用中括号访问
assert colors.green == '#00FF00' // 使用点表达式访问

// 两种方式
colors['pink'] = '#FF00FF'
colors.yellow = '#FFFF00'

assert colors.pink      // == '#FF00FF'
assert colors['yellow'] // == '#FFFF00'
assert colors instanceof java.util.LinkedHashMap // 默认使用LinkedHashMap类型
```

在上边的例子中，虽然没有明确的使用字符串`’red‘、’green‘`，但Groovy会自动把那些key转化为字符串。并且在默认情况下，初始化字典时key也不会去使用已经存在的变量：

```
def keyVal = 'name'
def persons = [keyVal: 'Guillaume'] // 非常注意: 此处的key是字符串keyVal而不是name

assert !persons.containsKey('name')
assert persons.containsKey('keyVal')

// 如果要使用一个变量作为key，需要用括号括住：
def keyVal = 'name'
def persons = [(keyVal): 'Guillaume'] // 相当于[ 'name' : 'Guillaume' ]

assert persons.containsKey('name')
assert !persons.containsKey('keyVal')
```

## 条件语句 - Condition

**if 语句**  
第一个决策语句是 if 语句。这种说法的一般形式是

```
// # syntax
if(condition) {
  statement #1
  statement #2
  ...
}

# 示例
//@ 流程控制Groovy表达式如if/else条件语句
if (env.BRANCH_NAME == 'master') {
  echo 'I only execute on the master branch'
} else {
  echo 'I execute elsewhere'
}// # syntax if(condition) {   statement #1   statement #2   ... } # 示例 //@ 流程控制Groovy表达式如if/else条件语句 if (env.BRANCH_NAME == 'master') {   echo 'I only execute on the master branch' } else {   echo 'I execute elsewhere' }
```

**For循环**  
for 语句用于遍历一组值。for 语句通常以以下方式使用。

```
for(variable declaration;expression;Increment) {
  statement #1
  statement #2
  …
}

# 例子：
for(int i = 0;i<5;i++) {
  println(i);
}
```

**Switch 语句**  
Swict 语句用于条件判断, 基础语法如下

```
switch () {
  case '1':
    echo 1
    break
  case '2':
    echo 2
    break
  default:
}
```

## 异常 - Exception

描述:流程控制是Groovy的异常处理机制,在实际过程中建议同时使用try...catch..finally进行捕获异常;

```
try {
      helloWorld()  // == Scripted Pipeline - Hello Wrold - STARTED - 1024!
      dir("place") {
          sh 'id'  // == uid=112(jenkins) gid=117(jenkins) groups=117(jenkins)
      }
  } catch (e) {
      // If there was an exception thrown, the build failed
      currentBuild.result = "FAILED"
      throw e
  } finally {
      println "success or failure, always send notifications"   // == Success or failure, always send notifications
  }
```

## 函数 - Functions

描述：Groovy中的方法是使用返回类型或使用def关键字定义的, 方法可以接收任意数量的参数并定义参数时不必显式定义类型，可以添加修饰符如`public，private和protected`。

Tips : 注意事项

-   默认情况下如果未提供可见性修饰符则该方法为public。
    
-   注意: 函数定义不能被包含在`node{}块`之中, 而函数调用是在 `node { stage() { 函数名称} }` 之中的;
    
-   注意: 函数参数有定义默认值
    

简单示例:

```
node {
  stage('函数调用') {
    // 函数名称
    methodName
  }
}

def methodName(String param = 'default1',int number = 1024) {
  //Method code
  println "Hello Wrold function - ${param} - ${age}!"
}
```

## 语法总结

描述: 此次对 `Scripted Pipeline` 语法的使用进行一个简单的总结, 或许在后面的Declarative Pipeline中可以进行使用;

```
node {
  // 变量定义
  def foo
  // 字符串
  def project="HelloWorld"
  // 单引号字符串（Single quoted string）不能解析变量
  def name='weiyigeek - ${project}'
  // 三单引号字符串（Triple single quoted string）不能解析变量
  def line='''\
  Line one
  Line ${project}
  '''
  // 定义字典Maps
  def person = [name: 'WeiyiGeek', age: 96]


  stage('Scripted Pipeline Syntax') {
    // (1) 变量声明
    foo="Identifiers"
    // (2) 变量调用
    echo "${foo} -- ${project}" // == Identifiers -- HelloWorld
    // (3) 字符串输出
    echo "my name is ${name}" // == my name is weiyigeek - ${project}
    // (4) 多行字符串输出
    echo "${line}" /* ==
    Line one
    Line ${project}
    */

    // (5) 变量对象属性输出
    echo "$person.name is $person.age years old" // == WeiyiGeek is 96 years old

    name = 'WeiyiGeek'
    project = "my name is ${name}"
    def greeting = "Hello ${name}" // name变量的值会被替换进去

    // (6) 字符串格式化声明
    assert greeting.toString()
    println "${project.toString()}" //调用方法后直接输出 ==  my name is  WeiyiGeek

    // 闭包
    def number = 1
    def eagerGString = "value == ${number}"   // 普通形式
    def lazyGString = "value == ${-> number}" // 这是一个闭包
    println eagerGString // == value == 1
    println lazyGString  // 此处输出为空 (value == 1)
    number=1024
    echo "-----------------"
    println eagerGString // == value == 1
    println lazyGString  // 此处输出为空 (value == 1024)


    // (7) 变量类型输出
    println "变量name类型："+name.class+", \n 变量project类型: "+project.class+", \n 变量greeting类型: "+greeting.class
    /* == 变量name类型：class java.lang.String,
          变量project类型: class org.codehaus.groovy.runtime.GStringImpl,
          变量greeting类型: class org.codehaus.groovy.runtime.GStringImpl */


    // (8) 数值类型与类型强转
    def decimal = 123.456
    println "${decimal.getClass()} -- ${decimal} " // ==  class java.math.BigDecimal -- 123.456
    //assert decimal instanceof BigDecimal

    // (9) 数组列表
    def arrayList = [1,2,3,"number"]
    println arrayList[0]   // == 1
    arrayList << '5'
    echo "assert ${arrayList[0]} ---  ${arrayList[-2]} --- ${arrayList[-1]}"   // == assert 1 ---  number --- 5

    def multi = [[0, 1], [2, 3]]  // 二维数组
    echo "assert ${multi[1][0]}"  // == 2

    // (10) 字典Map
    def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']
    colors['pink'] = '#FF00FF'
    colors.yellow = '#FFFF00'
    println "colors 类型 : "+ colors.pink.class + ", assert colors.pink : " + colors.pink + ",assert colors['yellow'] : " + colors['yellow']
    /*       colors 类型 : class java.lang.String, assert colors.pink : #FF00FF,assert colors['yellow'] : #FFFF00     */

    // 如果要使用一个变量作为key（缺省为字符串），需要用括号括住：
    def keyVal = 'name'
    def persons = [(keyVal): 'WeiyiGeek'] // 相当于[ 'name' : 'WeiyiGeek' ]
    println "key is name: " + persons.containsKey('name') + " , Key not is KeyVal : " + !persons.containsKey('keyVal')
    /* key is name: true , Key is  KeyVal : true */

    // (11) 条件语句 - if - for - switch
    if ( keyVal == 'name' ) {
      echo "keyVal value is name"   // == keyVal value is name
    } else {
      echo "keyVal value is not name"
    }

    if (env.BRANCH_NAME == 'master') {
      echo 'I only execute on the master branch'
    } else {
      echo 'I execute elsewhere'
    }

    for (int i=0; i < 5; i++) {
      println arrayList[i]  // 如果越界显示则 `null` // == 1,2,3,number,5
    }

    def option = "deploy"
    switch ("${option}") {
      case 'deploy':
          echo "deploy"   // == deploy
          break
      case 'rollback':
          echo "rollback"
          break
      default:
          echo "default"
    }

    // (12) 异常捕获
    try {
        helloWorld()  // == Scripted Pipeline - Hello Wrold - STARTED - 1024!
        dir("place") {
            sh 'id'  // == uid=112(jenkins) gid=117(jenkins) groups=117(jenkins)
        }
    } catch (e) {
        // If there was an exception thrown, the build failed
        currentBuild.result = "FAILED"
        throw e
    } finally {
        println "success or failure, always send notifications"   // == Success or failure, always send notifications
    }

    // （13）函数调用
    helloWorld("weiyigeek")  /* Scripted Pipeline - Hello Wrold - weiyigeek - 1024! */
  }
}

// (13) 函数声明定义时 不能包括在node中
def helloWorld (String username = 'STARTED',int age = 1024) {
    println "Scripted Pipeline - Hello Wrold - ${username} - ${age}!"
}
```

![](https://i0.hdslb.com/bfs/article/718e52fcf4bc126bacacddae2f61b9942d132694.png@942w_858h_progressive.webp)

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

> 如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！     
> 
>               欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】  

![](https://i0.hdslb.com/bfs/article/34186b2c202f80eafb4e63c78454e4739c78b2a1.png@942w_483h_progressive.webp)