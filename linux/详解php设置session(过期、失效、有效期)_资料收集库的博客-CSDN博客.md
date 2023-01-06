![](https://csdnimg.cn/release/blogv2/dist/pc/img/reprint.png)

[资料收集库](https://blog.csdn.net/u010433704) ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCurrentTime2.png) 于 2017-06-20 09:26:19 发布 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes2.png) 61071 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect2.png) 收藏 3

在php中设置[session](https://so.csdn.net/so/search?q=session&spm=1001.2101.3001.7020)有很多方面包有给session设置值或直接设置过期、失效和有效期，下面小编来给大家给各位朋友介绍怎么使用。

我们先来看看在php.ini中session怎么设置，打开 php.ini，查找Session设置部分中以下一项，代码如下：

<table><tbody><tr><td><p>1</p><p>2</p></td><td><div><p><code>session.save_path = </code><code>"N;/path"</code></p><p><code>session.save_path = </code><code>"C:/Temp"</code> <code>　　#此处以你自己设定的路径为准</code></p></div></td></tr></tbody></table>

 这项设置提供给我们可以给session存放目录进行多级散列，其中“N”表示要设置的目录级数，后面的“/path”表示session文件存放的根目录路径，比如我们设置为下面的格式，代码如下：

<table><tbody><tr><td><p>1</p></td><td><div><p><code>session.save_path = </code><code>"2;C:/Temp"</code></p></div></td></tr></tbody></table>

上面的设置表示我们把php的session文件进行两级目录存储，每一级目录分别是0-9和a-z共36个字母数字为目录名，这样存放session的目录可以达到36\*36个，共1332个文件夹，相信作为单台服务器来说，这是完全够用了，如果说您的系统架构设计为多台服务器共享session数据，可以把目录级增加到3级或者更多。

**Session过期时间设定**

继续PHP中的Session话题，在PHP中主要通过设置session.gc\_maxlifetime来设定Session的生存周期,例如如下代码：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p></td><td><div><p><code>&lt;?php</code></p><p><code>ini_set</code> <code>(</code> <code>'session.gc_maxlifetime'</code> <code>, 3600);</code></p><p><code>ini_get</code> <code>(</code> <code>'session.gc_maxlifetime'</code> <code>);</code></p><p><code>?&gt;</code></p></div></td></tr></tbody></table>

下面提供一个别人封装好的函数,但是我没有测试过,仅供参考,代码如下:

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p></td><td><div><p><code>&lt;?php</code></p><p><code>function</code> <code>start_session(</code> <code>$expire</code> <code>= 0)</code></p><p><code>{</code></p><p><code>&nbsp;</code> <code>if</code> <code>(</code> <code>$expire</code> <code>== 0) {</code></p><p><code>&nbsp;</code> <code>$expire</code> <code>= </code><code>ini_get</code> <code>(</code> <code>'session.gc_maxlifetime'</code> <code>);</code></p><p><code>&nbsp;</code> <code>} </code><code>else</code> <code>{</code></p><p><code>&nbsp;</code> <code>ini_set</code> <code>(</code> <code>'session.gc_maxlifetime'</code> <code>, </code><code>$expire</code> <code>);</code></p><p><code>&nbsp;</code> <code>}</code></p><p><code>&nbsp;</code> <code>if</code> <code>(emptyempty(</code> <code>$_COOKIE</code> <code>[</code> <code>'PHPSESSID'</code> <code>])) {</code></p><p><code>&nbsp;</code> <code>session_set_cookie_params(</code> <code>$expire</code> <code>);</code></p><p><code>&nbsp;</code> <code>session_start();</code></p><p><code>&nbsp;</code> <code>} </code><code>else</code> <code>{</code></p><p><code>&nbsp;</code> <code>session_start();</code></p><p><code>&nbsp;</code> <code>setcookie(</code> <code>'PHPSESSID'</code> <code>, session_id(), time() + </code><code>$expire</code> <code>);</code></p><p><code>&nbsp;</code> <code>}</code></p><p><code>}</code></p><p><code>?&gt;</code></p></div></td></tr></tbody></table>

**使用方法：**

加入start\_session(600);//600秒以后过期。

**session永不过期的方法**

打开php.ini设置文件，修改三行如下：

1、session.use\_cookies

把这个的值设置为1，利用cookie来传递sessionid

2、session.cookie\_lifetime

这个代表SessionID在客户端Cookie储存的时间，默认是0，代表浏览器一关闭SessionID就作废……就是因为这个所以PHP的session不能永久使用！ 那么我们把它设置为一个我们认为很大的数字吧，999999999怎么样，可以的！就这样。

3、session.gc\_maxlifetime

这个是Session数据在服务器端储存的时间，如果超过这个时间，那么Session数据就自动删除！那么我们也把它设置为99999999。

就这样一切ok了，当然你不相信的话就测试一下看看——设置一个session值过个10天半个月的回来看看，如果你的电脑没有断电或者宕机，你仍然可以看见这个sessionid。

当然也可能你没有控制服务器的权限并不能像我一样幸运的可以修改php.ini设置，一切依靠我们自己也是有办法的，当然就必须利用到客户端存储cookie了，吧得到的sessionID存储到客户端的cookie里面，设置这个cookie的值，然后把这个值传递给session\_id()这个函数，具体做法如下:

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p></td><td><div><p><code>&lt;?php</code></p><p><code>session_start();</code></p><p><code>$_SESSION</code> <code>[</code> <code>'count'</code> <code>];</code></p><p><code>isset(</code> <code>$PHPSESSID</code> <code>)?session_id(</code> <code>$PHPSESSID</code> <code>):</code> <code>$PHPSESSID</code> <code>= session_id();</code></p><p><code>$_SESSION</code> <code>[</code> <code>'count'</code> <code>]++;</code></p><p><code>setcookie(</code> <code>'PHPSESSID'</code> <code>, </code><code>$PHPSESSID</code> <code>, time()+3156000);</code></p><p><code>echo</code> <code>$count</code> <code>;</code></p><p><code>?&gt;</code></p></div></td></tr></tbody></table>

以上就是php设置session的具体做法，内容涉及session设置值或直接设置过期、失效和有效期，希望对大家的学习有所帮助。