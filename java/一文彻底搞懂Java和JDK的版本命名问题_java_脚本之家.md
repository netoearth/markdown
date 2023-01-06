Java是面向对象的编程语言，在我们开发Java应用的程序员的专业术语里，Java这个单词其实指的是Java开发工具，也就是JDK(Java Development Kit)。所以我们常常在CSDN等各大程序员论坛讨论到安装Java8或者JDK8或者JDK1.8或J2SE8或J2SE1.8或J2SE8或J2SE1.8，其实这3个专业词汇的概念是一样的。

**告诉庆哥，你对Java的版本号以及JDK的命名真正清楚嘛？比如：**

1.  **Java8  
    **
2.  **Java SE 8.0  
    **
3.  **JDK1.8  
    **
4.  **……**

知道这些是怎么回事嘛？知道还有个Java 2的说法嘛？知道还有以下说法嘛？

-   J2SE1.3
-   J2SE1.4
-   ……

**现在已经6月份了，到了9月份，一个新的长期支持版本，Java17就要发布了，啥？Java版本都到17了？不不不，我一直在用JDK1.8啊，咦，JDK1.8？Java17？**

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162825103.jpg)

这是怎么回事呢？**别着急，今天庆哥带你彻底搞懂这些蜜汁操作！**

## Java版本和JDK版本

要搞懂这些令人疑惑的人命名，那理解的一个关键就是Java版本和JDK版本了，首先啊，咱们常说Java有三个版本，对吧，分别是：

-   JavaSE（Java Platform，Standard Edition）Java标准版
-   JavaME（Java Platform，Micro Edition）Java微型版
-   JavaEE（Java Platform，Enterprise Edition) JAVA企业版

其实啊，**你只要关注JavaSE就行**，这个是Java的标准版本，像ME忽视就行，至于JavaEE是在JavaSE的基础上升级而来的一套规范，我们平常做Java开发，你想下，是不是就是需要个JDK，这个JDK是与JavaSE相对应的。

**完了，我知道你们又懵了……**

## JavaEE到底是个啥

那我就再详细点给大家说说这个JavaEE，其实我们平常听到最多的，用到最多的就是JavaSE，因为人家是Java的标准版本，但是这个JavaSE提供的是Java的核心功能，一般是用来开发桌面应用的，但是企业级开发，我们做的项目啥的就不简简单单是个桌面级应用了，一般是web应用，动态网站这些！

那么问题来了，**面对企业级的比较大的项目开发，JavaSE提供的一些核心基础功能用倒是可以用，但是用起来太费劲了**，很多东西都得自己从头造轮子，一步步的用代码从最基础的开始写，费劲啊。

于是乎，**在JavaSE的基础上整理出一套规范**，其目的就是用来解决企业级开发中遇到的一些问题，这些问题就是单独用JavaSE去整比较费劲的东西！

那啥又是规范呢？**说白了，就是规定你该怎样怎样去做，比如面对常见的web请求处理，我们知道有servlet，那JavaEE就对servlet做了规范**，也就是说你如果要用servlet去处理一个web请求，首先嘞，你必须得实现一个HttpServlet类，这还没完，你这个类还得继承Servlet接口，而且你还得实现它的接口方法，哪些呢？就是doGet和doPost这些，咋样，熟悉吧，再比如你这个doGet方法还必须得接收两个参数……

你看，JavaEE就给你规定了Servlet得按照这个规定去使用，于时一些其他厂商就围绕JavaEE的这些规范去做一些具体的实现，比如我们常见的tomcat，它被成为servlet容器，其中的servlet实现就是给予JavaEE对servlet做出的规范！

**说的再简单点，JavaEE规定了servlet如何去处理web请求，然后具体的厂商根据这个规定去做具体的实现和增强，然后就搞出了tomcat……**

那Spring是啥呢？最开始的Spring就是为了解决JavaEE在使用中遇到的一些问题，比如JavaEE中规定的servlet，那spring中的spring-mvc就是对这个servlet的进一步封装，从而让其变得更加好用！

实际上，**spring中大量使用了或者实现了JavaEE的一些规范标准**！说的再直白点，你JavaEE不是一组规范嘛，规定了啥啥啥该怎么用，那我spring就这样做，你规范中确实比较好用的我就直接拿来用，不好用的我就在加工处理封装成更好用的，可以简单的理解成spring就是JavaEE的升级版，或者超强实现版！

**随着时间的发展，JavaEE的更新太慢了，而Spring就非常迅速，而且人家超级好用，因此，慢慢的JavaEE早就落后十万八千里了。**

你像我们平常做开发，就下载配置个JDK，其实就是对应的JavaSE，然后我们使用的一般就是以Spring为主的框架了，那JavaEE体现在哪里，Spring框架中大量使用和实现了JavaEE规范，而JavaEE又是在JavaSE基础上升级而来的一组规范，那可不就是一个JDK就行了！

所以，现在你看一些关于JavaEE开发框架的书，其实都是在介绍SSM这些框架的，说白了，JavaEE慢慢也就成了各种给予其规范实现的一些框架了，为首的就是老大哥Spring了！

## Java版本的蜜汁操作

以上花了较多篇幅去介绍到底啥是JavaEE以及和Spring的一些关系，你就记住：

> 用Spring就对了

那我们再来看Java版本号的这些神奇操作，之前也说了，理解的关键就是Java版本和JDK版本，重点理解如下：

> 我们无论说Java版本还是JDK版本都是对于JavaSE这个标准版本而言，最终的则是要知道，每个Java版本其实是对应一个具体的JDK版本，也就是说Java是语言，JDK是Java这门语言的开发工具包，所以Java的版本可以说是抽象上的宏观上的一个概念，有其自己的版本名称，对应的具体的实实在在存在的则是JDK了

**记住啦，一个Java版本对应着一个JDK版本！**

我这里花了一个图，大家一起来看下：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162825104.jpg)

也就是最开始啊，Java的早期版本是在1995年发布的，那个时候是叫做Oak，但是这个商标被注册了，于时在1996年的时候更改为Java，那这个时候Java的第一个正式版本Java1.0就发布了，于此同时对应的开发工具包jdk的版本就是JDK1.0了。

[![自媒体培训](https://img.jbzj.com/image/ape820.png)](https://www.apeclass.com/?did=15)

## J2SE是个啥

那随着时间的发展，Java的版本不是一直叫做Java1.X这种形式，在到了1998年的时候，**Java的平台更名为J2SE**，所以从那个时候，Java的版本命名就成了J2SE 1.2这种形式，也就是这里：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162825105.jpg)

那与之对应的JDK的版本就是JDK1.2和JDK1.3这种形式了！

## JDK1.5的重大变化

那到了2004年的时候，Java版本变化比较大，此时对应的JDK1.5升级比较大，那为了表明该版本的重要性，于时将Java版本从原来的J2SE 1.5更名为Java SE 5.0（内部版本号1.5.0），**于是后续的Java版本号都是Java SE X的这种形式**，也就是这样：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162825106.jpg)

这里需要注意的是，直到2017年JavaSE 9的发布，此时对应的JDK版本都是JDK1.X这种形式，但是到了2018年JavaSE 10的发布就变了！

## JDK命名的变化

到了2018年发布JavaSE 10的时候，此时对应的JDK版本不再是JDK1.10这种形式，而是变成了JDK10这种形式，其实这个也是比较好理解的！

那后续的版本就是这个样子了：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162826107.jpg)

直到现在一直就稳定这样的命名了，比如最新的JavaSE 16对应的JDK16，那到了这里，**又有个蜜汁操作了**，我们看下Oracle的官网上的JDK变化：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162826108.jpg)

看到没，这里有JDK7，JDK8还有JDK9，按照我们之前说的不应该是这样的嘛：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162826109.jpg)

所以说啊，关于Java的版本和JDK版本命名真的挺乱的，在2018年更新JavaSE 10之后，对应的JDK版本叫做JDK10，后续为了方便统一，之前的JDK1.8也可以叫做JDK8了！

**不过到了现在，Java的版本号比较稳定了，也就是Java SE XX这种形式，比如即将发布的Java SE 17，这是一个长期受支持的版本！那对应的JDK版本就是JDK17了。**

## 查看JDK版本的更新内容

**作为一个Java程序员，你要随时关注着Java的版本更新**，以及JDK的升级带来了哪些新特性，那该如何关注这些呢？

其实就是这个：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162826108.jpg)

比如我们点击最新的JDK16：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162826110.jpg)  
![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162826111.jpg)

不知道的赶紧收藏这个地址吧：[https://docs.oracle.com/en/java/javase/index.html](https://docs.oracle.com/en/java/javase/index.html)

## 查看Javav版本变化

另外这里再提供一个随时查看Java版本变化的地址，可以看到一个比较直观的图片，就是这样的：

![在这里插入图片描述](https://img.jbzj.com/file_images/article/202106/20210604162826112.jpg)

是不是很直观呢？

赶紧收藏地址吧：https://zh.wikipedia.org/wiki/Java%E7%89%88%E6%9C%AC%E6%AD%B7%E5%8F%B2

## 小结

以上内容是我根据自己的经验以及查找相关资料所得，当然，其中的内容也有可能存在一些错误，比如关于JavaEE那块的理解，我理解的可能不够准确，因此，**如果你在阅读本文中发现描述不够准确或有误的地方，还请给予我一定的反馈，非常感谢！**

到此这篇关于一文彻底搞懂Java和JDK的版本命名问题的文章就介绍到这了,更多相关Java和JDK的版本命名内容请搜索脚本之家以前的文章或继续浏览下面的相关文章希望大家以后多多支持脚本之家！