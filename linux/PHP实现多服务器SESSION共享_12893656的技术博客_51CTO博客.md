©著作权归作者所有：来自51CTO博客作者wx59129d39de499的原创作品，请联系作者获取转载授权，否则将追究法律责任

## PHP实现多服务器SESSION共享

## 为什么要session共享

现在稍微大一点的网站基本上都有好几个子域名，比如www.feiniu.com, search.feiniu.com, member.feiniu.com，这些网站如果需要共用用户登录信息，那么就需要做到session共享，当然前提是有相同的主域。

## PHP的session原理

客户端访问php页面，执行session\_start，生成session\_id，一般我们是把session\_id存储到cookie上，session内容保存在服务端，客户端访问访问不同的页面都会把session\_id传到服务端，通过session\_id来获取session内容。

流程是这样，可是不同的服务器会对同一个客户端产生不同的session\_id，这样的话不同服务器就不能得到相同的session内容了。而且PHP 默认的 SESSION 数据都是分别保存在本服务器的文件系统中。

所以我们要解决session共享，就必须解决两个问题：

1\. 多台服务器用同一个session\_id

> 这个比较容易解决，只要在php中设置存session\_id的cookie域名为网站主域就可以了  
> 打开PHP.ini， 设置session.cookie\_domain = .feiniu.com,   
> 当然也可以在php代码当中设置ini\_set("session.cookie\_domain","feiniu.com");

2\. 多台服务器用同一个session\_id访问到相同的session内容

> 要实现这点，就必须把session内容存储到让所有服务器都能访问到的地方，php的session内容是默认存储到本服务器的文件中的，一般的解决方案是存入数据库，
> 
> memcache或者redis这种缓存服务器，当然用默认的文件存储方式也可以，用NFS统一存储。
> 
> 如何修改session存储引擎，参考这篇文章：

3\. 如何选择存储引擎

-   默认文件存储：这种方式的session销毁依托于php垃圾收集器，在高并发或销毁时间较长的情况下，在SESSION目录下产生大量文件，当然可以设置分级目录进行 SESSION 文件的保存。这会导致两个问题：第一、查找文件慢；第二，每个目录下可容纳的文件数是有限的，可能会导致新SESSION储存失败。
-   数据库存储：把Session存储在数据库里可以防止Session数据被垃圾收集器删除，可以固化存储session数据。但是用数据库来同步session，会加大数据库的IO，增加数据库的负担。而且数据库读写速度较慢，不利于session的适时同步。
-   memcache存储：

-   以这种方式来同步session，不会加大数据库的负担，并且安全性比较高，把session放到内存里面，比从文件中读取要快很多。
-   但是memcache把内存分成很多种规格的存储块，有块就有大小，这种方式也就决定了，memcache不能完全利用内存，会产生内存碎片，如果存储块不足，还会产生内存溢出
-   那些不需要“分布”的，不需要共享的，或者干脆规模小到只有一台服务器的应用，memcached不会带来任何好处，相反还会拖慢系统效率，因为网络连接同样需要资源。

-   redis存储：与memcache相比，redis访问稍稍慢一点点，好处是：

-   redis支持的数据结构较多，可以存储数组或对象，而memcache只能存储字符串
-   在session机器重启的情况下，memcache所有用户都必须重新获得 session，而redis不会
-   在突然涌来大量用户产生了很多数据把存储 session 的机器内存占满了的情况下，memcache 会罢工，所有 key 都没过期的话就不停的覆盖最后写入的数据，而 redis 只是会变慢 ，不会影响程序的逻辑

-   **赞**
-   **收藏**
-   **评论**
-   **举报**

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持JPEG/PNG/JPG，图片不超过1.9M

已经收到您得举报信息，我们会尽快审核

**相关文章**