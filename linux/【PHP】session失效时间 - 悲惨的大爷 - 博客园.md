最近用到php中session时，忽然发现php中的session有点让人头疼啊，要设置一个严格的特定时间内过期的session还真不太容易！  
后来在网上查询时，发现这个问题还真是有点普遍，网上也有关于这个问题的面试问题，如：如何严格限制session在30分钟后过期！这个问题的答案顺便也写在这里  
1.设置客户端cookie的lifetime为30分钟；  
2.设置session的最大存活周期也为30分钟；  
3.为每个session加入时间戳，然后在程序调用时进行判断；

至于为什么，我们首先来了解下php中session的基本原理：

PHP中的session有效期默认是1440秒（24分钟），也就是说，客户端超过24分钟没有刷新，当前session就会失效。当然如果用户关闭了浏览器，回话也就结束了，Session自然也不存在  
了！  
大家知道，Session储存在服务器端，根据客户端提供的SessionID来得到这个用户的文件，然后读取文件，取得变量的值，SessionID可以使用客户端的Cookie或者Http1.1协议的  
Query\_String（就是访问的URL的“?”后面的部分）来传送给服务器，然后服务器读取Session的目录……  
要控制Session的生命周期，首先我们需要了解一下php.ini关于Session的相关设置（打开php.ini文件，在“\[Session\]”部分）：  
1、session.use\_cookies：默认的值是“1”，代表SessionID使用Cookie来传递，反之就是使用Query\_String来传递；  
2、session.name：这个就是SessionID储存的变量名称，可能是Cookie，也可能是Query\_String来传递，默认值是“PHPSESSID”；  
3、session.cookie\_lifetime：这个代表SessionID在客户端Cookie储存的时间，默认是0，代表浏览器一关闭SessionID就作废……就是因为这个所以Session不能永久使用！  
4、session.gc\_maxlifetime：这个是Session数据在服务器端储存的时间，如果超过这个时间，那么Session数据就自动删除！  
还有很多的设置，不过和本文相关的就是这些了，下面开始讲如何设置Session的存活周期。  
前面说过，服务器通过SessionID来读取Session的数据，但是一般浏览器传送的SessionID在浏览器关闭后就没有了，那么我们只需要人为的设置SessionID并且保存下来，不就可以……  
如果你拥有服务器的操作权限，那么设置这个非常非常的简单，只是需要进行如下的步骤：  
1、把“session.use\_cookies”设置为1，使用Cookie来储存SessionID，不过默认就是1，一般不用修改；  
2、把“session.cookie\_lifetime”改为你需要设置的时间（比如一个小时，就可以设置为3600，以秒为单位）;  
3、把“session.gc\_maxlifetime”设置为和“session.cookie\_lifetime”一样的时间；  
在PHP的文档中明确指出，设定session有效期的参数是session.gc\_maxlifetime。可以在php.ini文件中，或者通过ini\_set()函数来修改这一参数。问题在于，经过多次测试，修改这个  
参数基本不起作用，session有效期仍然保持24分钟的默认值。  
由于PHP的工作机制，它并没有一个daemon线程，来定时地扫描session信息并判断其是否失效。当一个有效请求发生时，PHP会根据全局变量  
session.gc\_probability/session.gc\_divisor（同样可以通过php.ini或者ini\_set()函数来修改）的值，来决定是否启动一个GC（Garbage Collector）。  
默认情况下，session.gc\_probability ＝ 1，session.gc\_divisor ＝100，也就是说有1%的可能性会启动GC。GC的工作，就是扫描所有的session信息，用当前时间减去session的最后修  
改时间（modified date），同session.gc\_maxlifetime参数进行比较，如果生存时间已经超过gc\_maxlifetime，就把该session删除。  
到此为止，工作一切正常。那为什么会发生gc\_maxlifetime无效的情况呢？  
在默认情况下，session信息会以文本文件的形式，被保存在系统的临时文件目录中。在Linux下，这一路径通常为\\tmp，在 Windows下通常为C:\\Windows\\Temp。当服务器上有多个PHP应  
用时，它们会把自己的session文件都保存在同一个目录中。同样地，这些PHP应用也会按一定机率启动GC，扫描所有的session文件。

问题在于，GC在工作时，并不会区分不同站点的session。举例言之，站点A的gc\_maxlifetime设置为2小时，站点B的 gc\_maxlifetime设置为默认的24分钟。当站点B的GC启动时，它会扫  
描公用的临时文件目录，把所有超过24分钟的session文件全部删除掉，而不管它们来自于站点A或B。这样，站点A的gc\_maxlifetime设置就形同虚设了。  
找到问题所在，解决起来就很简单了。修改session.save\_path参数，或者使用session\_save\_path()函数，把保存session的目录指向一个专用的目录，gc\_maxlifetime参数工作正常了。

还有一个问题就是，gc\_maxlifetime只能保证session生存的最短时间，并不能够保存在超过这一时间之后session信息立即会得到删除。因为GC是按机率启动的，可能在某一个长时间内  
都没有被启动，那么大量的session在超过gc\_maxlifetime以后仍然会有效。  
解决这个问题的一个方法是，把session.gc\_probability/session.gc\_divisor的机率提高，如果提到100%，就会彻底解决这个问题，但显然会对性能造成严重的影响。另一个方法是自己  
在代码中判断当前session的生存时间，如果超出了 gc\_maxlifetime，就清空当前session。

另外在查看php手册时发现，手册本身提供了一个函数用来管理session，可以有效的解决session定时过期问题，

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
<?php
class FileSessionHandler
{
private $savePath;

function open($savePath, $sessionName)
{
$this->savePath = $savePath;
if (!is_dir($this->savePath)) {
mkdir($this->savePath, 0777);
}

return true;
}

function close()
{
return true;
}

function read($id)
{
return (string)@file_get_contents(“$this->savePath/sess_$id”);
}

function write($id, $data)
{
return file_put_contents(“$this->savePath/sess_$id”, $data) === false ? false : true;
}

function destroy($id)
{
$file = “$this->savePath/sess_$id”;
if (file_exists($file)) {
unlink($file);
}

return true;
}

function gc($maxlifetime)
{
foreach (glob(“$this->savePath/sess_*”) as $file) {
if (filemtime($file) + $maxlifetime < time() && file_exists($file)) {
unlink($file);
}
}

return true;
}
}

$handler = new FileSessionHandler();
session_set_save_handler(
array($handler, ‘open’),
array($handler, ‘close’),
array($handler, ‘read’),
array($handler, ‘write’),
array($handler, ‘destroy’),
array($handler, ‘gc’)
);

// the following prevents unexpected effects when using objects as save handlers
register_shutdown_function(‘session_write_close’);

session_start();
// proceed to set and retrieve values by key from $_SESSION
?>
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

PS：也需有的童鞋会有一些疑惑，为什么定义的时候read 、write等方法都有参数，但是用session\_set\_save\_handler函数调用的时候却没有加入任何参数，这是因为这些参数本身是不用我  
们手动传入的，而是由php自动传入，可以理解为php会从php.ini里读取对应的设置，然后将对应的值传入对应方法！所以我们不用担心这些参数，只需要按照这些函数的格式，和参数个数将  
方法写好就OK！

2.

大多数据情况下我们对于session过期时间使用的是默认设置的时间，而对于一些有特殊要求的情况下我们可以设置一下session过期时间。

对此，可以在PHP中,设置php.ini,找到session.gc\_maxlifetime = 1440 #(PHP5默认24分钟)  
这里你可以随便设置一下过期时间.但是有人说设置以后,好象不起作用!  
其实不是不起作用,而是因为系统默认:

<table><tbody><tr><td><code>1</code></td><td><code>session.gc_probability = 1</code></td></tr></tbody></table>

<table><tbody><tr><td><code>2</code></td><td><code>session.gc_divisor = 1000</code></td></tr></tbody></table>

garbage collection 有个概率的，1/1000就是session 1000次才有一次被回收。  
只要你的访问量大了,那就能达到回收的效果.  
或者你也可以设置一下session.gc\_divisor 的值,  
比如:**session.gc\_divisor = 1,这样就能明显的看到SESSION过期的效果了.**

我们最常用的是在php程序中设置，如下例程序所示：

<table><tbody><tr><td><code>2</code></td><td><code>if</code><code>(!isset(</code><code>$_SESSION</code><code>[</code><code>'last_access'</code><code>])||(time()-</code><code>$_SESSION</code><code>[</code><code>'last_access'</code><code>])&gt;60)</code></td></tr></tbody></table>

<table><tbody><tr><td><code>3</code></td><td><code>$_SESSION</code><code>[</code><code>'last_access'</code><code>] = time();</code></td></tr></tbody></table>

这样就搞定了,如果要设置已过期的话也可以在程序中实现：

<table><tbody><tr><td><code>2</code></td><td><code>unset(</code><code>$_SESSION</code><code>[</code><code>'last_access'</code><code>]);</code><code>// 或 $_SESSION['last_access']='';</code></td></tr></tbody></table>

**session有过期的机制：**

session.gc\_maxlifetime 原来session 过期是一个小概率的事件，分别使用session.gc\_probability和session.gc\_divisor 来确定运行session 中gc 的概率 session.gc\_probability和session.gc\_divisor的默认值分别为 1和100。分别为分子和分母 所以session中gc的概率运行机会为1% 。如果修改这两个值，则会降低php的效率。所以这种方法是不对的！！  
因此，修改php.ini文件中的gc\_maxlifetime变量就可以延长session的过期时间了：（例如，我们把过期时间修改为86400秒）  
session.gc\_maxlifetime = 86400  
然后，重启你的web服务（一般是apache）就可以了。

**session“回收”何时发生：**

默认情况下，每一次php请求，就会有1/100的概率发生回收，所以可能简单的理解为“每100次php请求就有一次回收发生”。这个概率是通过以下参数控制的  
#概率是gc\_probability/gc\_divisor

<table><tbody><tr><td><code>1</code></td><td><code>session.gc_probability = 1</code></td></tr></tbody></table>

<table><tbody><tr><td><code>2</code></td><td><code>session.gc_divisor = 100</code></td></tr></tbody></table>

注意1：假设这种情况gc\_maxlifetime=120，如果某个session文件最后修改时间是120秒之前，那么在下一次回收（1/100的概率）发生前，这个session仍然是有效的。

注意2：如果你的session使用session.save\_path中使用别的地方保存session，session回收机制有可能不会自动处理过期session文件。这时需要定时手动（或者crontab）的删除过期的session：

<table><tbody><tr><td><code>1</code></td><td><code>cd</code>&nbsp;<code>/path/to/sessions;&nbsp;</code><code>find</code>&nbsp;<code>-cmin +24 |&nbsp;</code><code>xargs</code>&nbsp;<code>rm</code></td></tr></tbody></table>

**PHP中的session永不过期**

不修改程序是最好的方法了，因为如果修改程序，测试部一定非常郁闷，那么只能修改系统环境配置，其实很 简单，打开php.ini设置文件，修改三行如下：

**1、session.use\_cookies**

把这个的值设置为1，利用cookie来传递sessionid

**2、session.cookie\_lifetime**

这个代表SessionID在客户端Cookie储存的时间，默认是0，代表浏览器一关闭SessionID就作废……就是因为这个所以PHP的 session不能永久使用！ 那么我们把它设置为一个我们认为很大的数字吧，999999999怎么样，可以的！就这样。

**3、session.gc\_maxlifetime**

这个是Session数据在服务器端储存的时间，如果超过这个时间，那么Session数据就自动删除！ 那么我们也把它设置为99999999。

就这样一切ok了，当然你不相信的话就测试一下看看——设置一个session值过个10天半个月的回来看看，如果你的电脑没有断电或者宕机，你仍然可以看见这个sessionid。

当然也可能你没有控制服务器的权限并不能像我一样幸运的可以修改php.ini设置，一切依靠我们自己也是有办法的，当然就必须利用到客户端存储 cookie了，把得到的sessionID存储到客户端的cookie里面，设置这个cookie的值，然后把这个值传递给session\_id()这 个函数

**session失效不传递**

我们先写个php文件：<?=phpinfo()?>， 传到服务器去看看服务器的参数配置。  
转到session部分，看到session.use\_trans\_sid参数被设为了零。  
这个参数指定了是否启用透明SID支持，即session是否随着URL传递。我个人的理解是，一旦这个参数被设为0，那么每个URL都会启一个session。这样后面页面就无法追踪得到前面一个页面的session，也就是我们所说的无法传递。两个页面在服务器端生成了两个session文件，且无关联。（此处精确原理有待确认）  
所以一个办法是在配置文件php.ini里把session.use\_trans\_sid的值改成1。

当然我们知道，不是谁都有权限去改php的配置的，那么还有什么间接的解决办法呢？  
下面就用两个实例来说明：  
文件1 test1.php

<table><tbody><tr><td><code>02</code></td><td><code>//表明是使用用户ID为标识的session</code></td></tr></tbody></table>

<table><tbody><tr><td><code>06</code></td><td><code>//将session的name赋值为Havi</code></td></tr></tbody></table>

<table><tbody><tr><td><code>07</code></td><td><code>$_SESSION</code><code>[</code><code>'name'</code><code>]=</code><code>"Havi"</code><code>;</code></td></tr></tbody></table>

<table><tbody><tr><td><code>08</code></td><td><code>//输出session，并设置超链接到第二页test2.php</code></td></tr></tbody></table>

<table><tbody><tr><td><code>09</code></td><td><code>echo</code>&nbsp;<code>"&lt;a href="</code><code>test2.php</code><code>" rel="</code><code>external nofollow</code><code>" &gt;"</code><code>.</code><code>$_SESSION</code><code>[</code><code>'name'</code><code>].</code><code>"&lt;/a&gt;"</code><code>;</code></td></tr></tbody></table>

文件2： test2.php

<table><tbody><tr><td><code>6</code></td><td><code>//输出test1.php中传递的session。</code></td></tr></tbody></table>

<table><tbody><tr><td><code>7</code></td><td><code>echo</code>&nbsp;<code>"This is "</code><code>.</code><code>$_SESSION</code><code>[</code><code>'name'</code><code>];</code></td></tr></tbody></table>

所以，重点是在session\_start();前加上session\_id(SID);，这样页面转换时，服务器使用的是用户保存在服务器session文件夹里的session，解决了传递的问题。  
不过有朋友会反映说，这样一来，多个用户的session写在一个SID里了，那Session的价值就发挥不出来了。所以还有一招来解决此问题，不用加session\_id(SID);前提是你对服务器的php.ini有配置的权限：  
output\_buffering改成ON，道理就不表了。  
第二个可能的原因是对服务器保存session的文件夹没有读取的权限，还是回到phpinfo.php中，查看session保存的地址：

<table><tbody><tr><td><code>1</code></td><td><code>session.save_path:&nbsp;</code><code>var</code><code>/tmp</code></td></tr></tbody></table>

所以就是检查下var/tmp文件夹是否可写。  
写一个文件：test3.php来测试一下：

<table><tbody><tr><td><code>2</code></td><td><code>echo</code>&nbsp;<code>var_dump(</code><code>is_writeable</code><code>(</code><code>ini_get</code><code>(</code><code>"session.save_path"</code><code>)));</code></td></tr></tbody></table>

如果返回bool(false)，证明文件夹写权限被限制了，那就换个文件夹咯，在你编写的网页里加入：

<table><tbody><tr><td><code>1</code></td><td><code>//设置当前目录下session子文件夹为session保存路径。</code></td></tr></tbody></table>

<table><tbody><tr><td><code>2</code></td><td><code>$sessSavePath</code>&nbsp;<code>= dirname(</code><code>__FILE__</code><code>).</code><code>'/session/'</code><code>;</code></td></tr></tbody></table>

<table><tbody><tr><td><code>3</code></td><td><code>//如果新路径可读可写（可通过FTP上变更文件夹属性为777实现），则让该路径生效。</code></td></tr></tbody></table>

<table><tbody><tr><td><code>4</code></td><td><code>if</code><code>(</code><code>is_writeable</code><code>(</code><code>$sessSavePath</code><code>) &amp;&amp;&nbsp;</code><code>is_readable</code><code>(</code><code>$sessSavePath</code><code>))</code></td></tr></tbody></table>

<table><tbody><tr><td><code>6</code></td><td><code>session_save_path(</code><code>$sessSavePath</code><code>);</code></td></tr></tbody></table>

3.

概述：每一次php请求，会有1/100的概率（默认值）触发“session回收”。如果“session回收”发生，那就会检查/tmp/sess\_\*的文件，如果最后的修改时间到现在超过了1440秒（gc\_maxlifetime的值），就将其删除，意味着这些session过期失效。

1\. session在server端（一般是Apache with PHP module）如何存在的？

默认的，php会将session保存在/tmp目录下，文件名为这个样子：sess\_01aab840166fd1dc253e3b4a3f0b8381。每一个文件对应了一个session（会话）。  
more /tmp/sess\_01aab840166fd1dc253e3b4a3f0b8381  
username|s:9:”jiangfeng”;admin|s:1:”0″;  
#变量名|类型:长度:值  
删除这里的session文件，就表示对应的session失效了。

2\. session在client端（一般是浏览器）如何存在的？  
session在浏览器端，只需要保存session ID（由server端生成的唯一ID）就可以了。有两种保存方式：在cookie中、在url里面。如果cookie中保存session ID，就可以看到浏览器的cookie中有一个PHPSESID变量。如果是URL传递的，就可以看到形如:  
index.php?PHPSESID=01aab840166fd1dc253e3b4a3f0b8381的URL。（在server端通过session.use\_cookies来控制使用哪一种方式）

3\. 在server端，php如何判断session文件是否过期？  
如果”最后的修改时间”到”现在”超过了gc\_maxlifetime（默认是1440）秒，这个session文件就被认为是过期了，在下一次session回收的时候，如果这个文件仍然没有被更改过，这个session文件就会被删除（session就过期了）。  
简单的说，如果我登录到某网站，如果在1440秒（默认值）内没有操作过，那么对应的session就认为是过期了。  
所以，修改php.ini文件中的gc\_maxlifetime变量就可以延长session的过期时间了：（例如，我们把过期时间修改为86400秒）  
session.gc\_maxlifetime = 86400  
然后，重启你的web服务（一般是apache）就可以了。

注意：php5里面session过期使用了回收机制。这里设置时间为86400秒，如果session在86400秒内没有被修改过，那么在下一次“回收”时才真的被删除。

3\. session“回收”何时发生？

默认情况下，每一次php请求，就会有1/100的概率发生回收，所以可能简单的理解为“每100次php请求就有一次回收发生”。这个概率是通过以下参数控制的

#概率是gc\_probability/gc\_divisor  
session.gc\_probability = 1  
session.gc\_divisor = 100  
注意1：假设这种情况gc\_maxlifetime=120，如果某个session文件最后修改时间是120秒之前，那么在下一次回收（1/100的概率）发生前，这个session仍然是有效的。

注意2：如果你的session使用session.save\_path中使用别的地方保存session，session回收机制有可能不会自动处理过期session文件。这时需要定时手动（或者crontab）的删除过期的session：cd /path/to/sessions; find -cmin +24 | xargs rm

4\. 一些特殊情况

因为回收机制会检查文件的“最后修改时间”，所以如果某个会话是活跃的，但是session的内容没有改变过，那么对应的session文件也就没有改变过，回收机制会认为这是一个长时间没有活跃的session而将其删除。这是我们不愿看到的，可以通过增加如下的简单代码解决这个问题：

<?php  
if(!isset($_SESSION['last_access'])||(time()-$\_SESSION\['last\_access'\])>60)  
        $\_SESSION\['last\_access'\] = time();  
?>代码会每隔60秒，尝试修改修改一次session。

总结：如果想修改session过期时间，修改变量gc\_maxlifetime就可以了。php5的session采用被动的回收机制（garbage collection）。过期的session文件不会自己消失，而是通过触发“回收”来处理过期的session。