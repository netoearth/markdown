-   A+

摘要

经过多方的考虑，决定使用方案C，使用nginx自带模块生成缩略图，模块：–with-http\_image\_filter\_module.  
如下是我的安装参数：

./configure –prefix=/usr/local/nginx-1.4.1 –with-http\_stub\_status\_module –with-http\_realip\_module –with-http\_image\_filter\_module –with-debug

修改nginx.conf配置文件,或者放到你相应的server块中.

为了手机端浏览到与手机分辨率相匹配的图片，提高app访问速度以及减少用户的手机流量，需要将图片生成缩略图，这边共有以下解决方案。  
A.    发布新闻生成多重缩略图 – 无法匹配到各种尺寸图片  
B.    当相应缩略图不存在，则使用[php](http://www.ttlsa.com/php/ "php")或者java等程序生成相应缩略图 – 需要程序员协助  
C.    使用[nginx](http://www.ttlsa.com/nginx/ "nginx")自带模块生成缩略图 – 运维即可完成  
D.    使用nginx＋[lua](http://www.ttlsa.com/monitor/lua/ "lua")生成缩略图

经过多方的考虑，决定使用方案C，使用nginx自带模块生成缩略图，模块：--with-http\_image\_filter\_module.  
如下是我的安装参数：

<table><tbody><tr><td data-settings="hide"></td><td><div><p><span>.</span><span>/</span><span>configure</span><span> </span><span>--</span><span>prefix</span><span>=</span><span>/</span><span>usr</span><span>/</span><span>local</span><span>/</span><span>nginx</span><span>-</span><span>1.4.1</span><span> </span><span>--</span><span>with</span><span>-</span><span>http_stub_status</span><span>_</span>module<span> </span><span>\</span></p><p><span>--</span><span>with</span><span>-</span><span>http_realip_module</span><span> </span><span>--</span><span>with</span><span>-</span><span>http_image_filter_module</span><span> </span><span>--</span><span>with</span><span>-</span><span>debug</span></p></div></td></tr></tbody></table>

修改nginx.conf配置文件,或者放到你相应的server块中.

<table><tbody><tr><td data-settings="hide"></td><td><div><p><span></span><span>location</span><span> </span><span>~</span><span>*</span><span> </span><span>/</span><span>(</span><span>\</span><span>d</span><span>+</span><span>)</span><span>\</span><span>.</span><span>(</span><span>jpg</span><span>)</span><span>$</span><span> </span><span>{</span></p><p><span>set</span><span> </span><span>$</span><span>h</span><span> </span><span>$</span><span>arg_h</span><span>;</span><span>&nbsp;&nbsp; </span><span># 获取参数ｈ的值</span></p><p><span>set</span><span> </span><span>$</span><span>w</span><span> </span><span>$</span><span>arg_w</span><span>;</span>　<span># 获取参数w的值</span></p><p><span>#image_filter crop $h $w;</span></p><p><span>image_filter </span><span>resize</span><span> </span><span>$</span><span>h</span><span> </span><span>$</span><span>w</span><span>;</span><span> </span><span>#　根据给定的长宽生成缩略图</span></p><p><span>}</span></p><p><span>location</span><span> </span><span>~</span><span>*</span><span> </span><span>/</span><span>(</span><span>\</span><span>d</span><span>+</span><span>)</span><span>_</span><span>(</span><span>\</span><span>d</span><span>+</span><span>)</span><span>x</span><span>(</span><span>\</span><span>d</span><span>+</span><span>)</span><span>\</span><span>.</span><span>(</span><span>jpg</span><span>)</span><span>$</span><span> </span><span>{</span></p><p><span>if</span><span> </span><span>(</span><span> </span><span>-</span><span>e</span><span> </span><span>$</span><span>document_root</span><span>/</span><span>$</span><span>1.</span><span>$</span><span>4</span><span> </span><span>)</span><span> </span><span>{</span>　<span># 判断原图是否存在</span></p><p><span>rewrite</span><span> </span><span>/</span><span>(</span><span>\</span><span>d</span><span>+</span><span>)</span><span>_</span><span>(</span><span>\</span><span>d</span><span>+</span><span>)</span><span>x</span><span>(</span><span>\</span><span>d</span><span>+</span><span>)</span><span>\</span><span>.</span><span>(</span><span>jpg</span><span>)</span><span>$</span><span> </span><span>/</span><span>$</span><span>1.</span><span>$</span><span>4</span><span>?</span><span>h</span><span>=</span><span>$</span><span>2</span><span>&amp;</span><span>w</span><span>=</span><span>$</span><span>3</span><span> </span><span>last</span><span>;</span></p><p><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span>}</span></p><p><span>return</span><span> </span><span>404</span><span>;</span></p><p><span>}</span></p></div></td></tr></tbody></table>

例如图片：  
http://test.ttlsa.com/123\_100x10.jpg  
1、    首先判断是否存在原图123.jpg,不存在直接返回404（如果原图都不存在，还生成缩略图干啥，对吧）  
2、    跳转到http://tset.ttlsa.com/123.jpg?h=100&w=10，将参数高h和宽10带到url中。  
3、    Image\_filter resize指令根据h和w参数生成相应缩略图。  
备注：长宽取小，例如原图是100\*10,你传入的是10\*2，那么他会给你生成10\*1的图片.

生成缩略图只是image\_filter功能中的一个，它一共支持4种参数：  
test:返回是否真的是图片  
size:返回图片长短尺寸,返回json格式数据  
corp:截取图片的一部分,从左上角开始截取,尺寸写小了,图片会被剪切  
resize:缩放图片,等比例缩放

nginx生成缩略图优缺点  
优点：  
1、    根据传入参数即可生成各种比例图片  
2、    不占用任何硬盘空间

缺点：  
1、消耗CPU,访问量大将会给服务器带来极大的负担.

几点建议：  
1、    生成缩略是个消耗cpu的操作，如果访问量比较大的站点，最好考虑使用程序生成缩略图到硬盘上，或者在前端加上cache或者使用CDN。

转载请注明出处：http://www.ttlsa.com/html/1612.html

![weinxin](http://www.ttlsa.com/wp-content/uploads/2017/05/qrcode_for_gh_2fad25f55e8b_258.jpg)

**微信公众号**

扫一扫关注运维生存时间公众号，获取最新技术文章~