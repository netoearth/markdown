**本篇为Redis系列学习的最后一篇，本系列从入门到实践花费了许多精力才完成，以供读者阅读，其中经历不知道多少次摸爬滚打的痛苦（抓狂）。  
**

**本章目录:**

-   0x00 Redis 管理工具
    

-   AnotherRedisDesktopManage
    
-   PhpRedisAdmin
    

-   0x01 客户端脚本连接
    

-   (0) Redis Lua 脚本连接示例
    
-   (1) PHP 脚本连接使用 redis 示例
    
-   (2) C# 连接使用redis
    
-   (3) JAVA 连接操作 redis 示例
    
-   (4) Python 连接Redis Sentinel实例
    

-   0x0n 入坑出坑
    

-   (1) 安装编译
    
-   问题1.Redis进行源码编译时显示`zmalloc.h:50:10: fatal error: jemalloc/jemalloc.h: No such file or directory`错误
    
  
-   (2) 运行使用
    
-   问题0.使用命令行-a 参数时会报`Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.`警告
    
-   问题1.警告超委托内存设置为0后台保存可能在低内存条件下失败
    
-   问题2.当创建redis集群时显示`Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.`错误
    
-   问题3.Redis集群启动报`Node 172.16.24.214:6379 replied with error: ERR Unknown node`错误。
    
-   问题4.Redis集群进行卡槽迁移时报`The following slots are open: xxxx.`错误。
    
-   问题5.集群初始化时一直等待 `Waiting for the cluster to join`错误。
    
-   问题6.当报无法连接的时候，通过 telnet192.168.1.13:6379 是无法连接通，则说明配置的哪里有问题导致的
    
-   问题7.使用check进行检测集群运行状态时显示`【4】[ERR] Not all 16379 slots are covered by nodes`错误。
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: 一个更快更好更稳定的redis桌面管理器，兼容Linux、windows、mac。更重要的是，加载大量密钥时不会崩溃。  
项目地址: https://github.com/qishibo/AnotherRedisDesktopManager/releases

描述: phpRedisAdmin是一个简单的web界面，用于管理Redis数据库。它是根据知识共享署名3.0许可证发布的。此代码由Erik Duckerlboer开发和维护。  
项目地址: https://github.com/erikdubbelboer/phpRedisAdmin

例如，以docker方式进行部署PhpRedisAdmin

```
# 示例
  docker run --rm -it -e REDIS_1_HOST=myredis.host -e REDIS_1_NAME=MyRedis -p 80:80 erikdubbelboer/phpredisadmin

# Environment variables summary
  REDIS_1_HOST - define host of the Redis server
  REDIS_1_NAME - define name of the Redis server
  REDIS_1_PORT - define port of the Redis server
  REDIS_1_AUTH - define password of the Redis server
  ADMIN_USER - define username for user-facing Basic Auth
  ADMIN_PASS - define password for user-facing Basic Auth
```

Redis 脚本使用 Lua 解释器来执行脚本,通过内嵌支持 Lua 环境,执行脚本的常用命令为 EVAL。

1.EVAL script numkeys key \[key ...\] arg \[arg ...\] #执行 Lua 脚本。

-   script：Lua 脚本程序
    
-   numkeys：用于指定键名参数的个数。
    
-   key \[key ...\]：从 EVAL 的第三个参数开始算起,表示在脚本中所用到的那些 Redis 键(key),这些键名参数可以在 Lua 中通过全局变量 KEYS 数组,用 1 为基址的形式访问( KEYS\[1\] , KEYS\[2\] ,以此类推)。
    
-   arg \[arg ...\]： 附加参数
    

2.EVALSHA sha1 numkeys key \[key ...\] arg \[arg ...\] #根据给定的 sha1 校验码，执行缓存在服务器中的脚本

-   sha1:通过 SCRIPT LOAD 生成的 sha1 校验码。
    

3.SCRIPT EXISTS script \[script ...\] 查看指定的脚本是否已经被保存在缓存当中。

4.SCRIPT FLUSH 从脚本缓存中移除所有脚本。

5.SCRIPT KILL 杀死运行的Lua脚本,主要用于终止运行时间过长的脚本，比如一个因为 BUG 而发生无限循环的脚本

6.SCRIPT LOAD script 将脚本 script 添加到脚本缓存中,但并不立即执行这个脚本。

```
192.168.1.100:6379[7]>  EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"

> SCRIPT LOAD "return 'hello moto'" #将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本
"232fd51614574cf0867b83d384a5e898cfd24e5a"  #给定脚本的 SHA1 校验和

> SCRIPT EXISTS 232fd51614574cf0867b83d384a5e898cfd24e5a #校验指定的脚本是否已经被保存在缓存当
1) (integer) 1

> EVALSHA "232fd51614574cf0867b83d384a5e898cfd24e5a" 0
"hello moto"

> SCRIPT FLUSH     # 清空缓存OK
> SCRIPT Kill     # 杀死当前正在运行的 Lua 脚本
```

扩展官网：https://pecl.php.net/package/redis //下载与php对应得版本(非常注意操作系统得位数)  
环境：WINDOW7/XMAPP/PHP7.2  
安装扩展：

`#Step1.下载TS版本且32位系统 PHP 7.2 Thread Safe (TS) x86 #Step2.解压php_redis.dll到php根目录中ext目录中 #Step3.在php.ini配置启用redis扩展;extension = php_redis #Step4.执行php -m 查看是否有该模块 @Linux下安装redis(百度即可) PHP redis 驱动：下载地址为:https://github.com/phpredis/phpredis/releases pipize/php-config`

php连接使用redis案例：

```
<?php
/***
 * 功能：实现redis连接以及5种数据类型元素建立
 */

 //Step1.实例化redis对象设置连接参数
$redis = new \Redis();
try{
    $link = $redis->pconnect('127.0.0.1',6379,1,NULL,100); #设置超时时间并且重试（返回Boolean）
    if(!$link){
        throw new Exception("!Redis服务器连接失败"); //抛出异常
    }else{
        echo "Redis连接成功!<br>";
    }
}catch(Exception $e){
    print_r($e);
    exit(0);
}

//Step2.进行redis认证
if($redis->auth('weiyigeek'))
{
    echo "Redis AUTH 认证成功!<br>";
}else{
    echo "Redis AUTH 认证失败!";
}

//Step3.String 类型
$redis->set('name','Weiyigeek');        //设置String类型键值
$redis->mSet(['id'=>'Weiyi','age'=>22]);
$redis->setEx('key',120,'value');  //存活2分钟

$strLen = $redis->strlen('name');  //获取字符串长度
$type = $redis->type('age');     //返回其类型
$redis->incrBy('age',8);    //String类型进行计算
$getValue = $redis->mGet(['name','id','age']); //获取String

echo "name键的字符串长度：".$strLen."<br>";
echo "<pre>";
var_dump($type);
var_dump($getValue);
echo "</pre> <br>";

##### 执行结果 #######
// Redis连接成功!
// Redis AUTH 认证成功!
// name键的字符串长度：9

// int(1)
// array(3) {
//   [0]=>
//   string(9) "Weiyigeek"
//   [1]=>
//   string(5) "Weiyi"
//   [2]=>
//   string(2) "30"
// }




//Step2.List 类型
$redis->delete('list1');
$redis->lpush('list1','A','C'); //直接插入多个元素 从左边开始插入 ->
$redis->rpush('list1','B');

$redis->delete('key1');
$redis->rPush('key1', 'A','B','C','D'); // returns 3 从右边开始插<-

$lcount = $redis->lLen('key1'); //元素个数

echo "key1元素个数:".$lcount."<br>";
echo "<pre>";
var_dump($redis->lRange('list1', 0, -1));
var_dump($redis->lRange('key1', 0, -1));
echo "</pre> <br>";

##### 执行结果 #######
// key1元素个数:4

// array(3) {
//   [0]=>
//   string(1) "C"
//   [1]=>
//   string(1) "A"
//   [2]=>
//   string(1) "B"
// }
// array(4) {
//   [0]=>
//   string(1) "A"
//   [1]=>
//   string(1) "B"
//   [2]=>
//   string(1) "C"
//   [3]=>
//   string(1) "D"
// }




//Step3.Hash 类型
$redis->delete('driver1');
$redis->hSet('driver1','name','张伟');
$redis->hSet('driver1','age','22');
$redis->hSet('driver1','gender',0);
$redis->hSet('driver1','love','Play Computer!');

$hcount = $redis->hLen('driver1');  //散列个数
$hlen = $redis->hStrLen('driver1','love'); //散列种键值的长度
$var = $redis->hGet('driver1','name');  //获取HASH种的键值
$var1 = $redis->hMGet('driver1',array('age','gender','love'));

echo "散列driver1的长度为：".$hcount.", Love字段值的长度：".$hlen."<br><pre>";
var_dump($var);
var_dump($var1);
echo "</pre><br>";

##### 执行结果 #######
// 散列driver1的长度为：4, Love字段值的长度：14

// string(6) "张伟"
// array(3) {
//   ["age"]=>
//   string(2) "22"
//   ["gender"]=>
//   string(1) "0"
//   ["love"]=>
//   string(14) "Play Computer!"
// }



//Step4.set类型
$redis->sRem('set1','A','B','C','D');  //删除集合种的元素
$redis->sAdd('set1','B','A','C','D');  //添加集合元素
$redis->sAdd('set2','B','D','E','F');

$sCount = $redis->sCard('set1');  //获取集合种的值
$sinter = $redis->sInter('set1','set2'); //集合set1与set2的交集

$diffCount = $redis->sDiffStore('set3', 'set1', 'set2'); //将集合set1 - set2的差集存(A,C)入set3种

echo "集合set1的长度为：".$sCount.", set1与set2差集元素的个数为：".$diffCount."<br><pre>";
var_dump($sinter);  //交
var_dump($redis->sMembers('set3')); //差集
echo "</pre><br>";

##### 执行结果 #######
// 集合set1的长度为：4, set1与set2差集元素的个数为：2

// array(2) {
//   [0]=>
//   string(1) "B"
//   [1]=>
//   string(1) "D"
// }
// array(2) {
//   [0]=>
//   string(1) "C"
//   [1]=>
//   string(1) "A"
// }



//Step5.sort set 有序集合类型
$redis->delete('zset1');
$redis->zAdd('zset1',1.0,'zhanghai');
$redis->zAdd('zset1',3.0,'Weiyigeek');
$redis->zAdd('zset1',7.5,'Chenghuiming');
$redis->zAdd('zset1',8.6,'Kangkang');

$zCount = $redis->zCard('zset1'); //有序集合键里有多个元素
$zCount1 = $redis->zCount('zset1',0,5.0);  //score在0~5之间的个数
$desc = $redis->zRange('zset1',0,-1,true); //分数从小到大排列 （最后一个参数是显示分数）
$asc = $redis->zRevRange('zset1',0,-1,true); //分数从大到小排列

echo "有序集合zset1有 ".$zCount."个元素,且score分数在0~5之间的个数：".$zCount1."<br><pre>";
var_dump($desc);
var_dump($asc);
echo "</pre>";
```

执行结果 ：  

```
// 有序集合zset1有 4个元素,且score分数在0~5之间的个数：2
// array(4) {
//   ["zhanghai"]=>
//   float(1)
//   ["Weiyigeek"]=>
//   float(3)
//   ["Chenghuiming"]=>
//   float(7.5)
//   ["Kangkang"]=>
//   float(8.6)
// }
// array(4) {
//   ["Kangkang"]=>
//   float(8.6)
//   ["Chenghuiming"]=>
//   float(7.5)
//   ["Weiyigeek"]=>
//   float(3)
//   ["zhanghai"]=>
//   float(1)
// }
```

高性能Redis协议封装，支持.Net Core，经过一年多日均80亿调用量验证  
项目地址：http://git.newlifex.com/NewLife/NewLife.Redis

**1)基础 Redis**  
Redis实现标准协议以及基础字符串操作，完整实现由独立开源项目NewLife.Redis提供;采取连接池加同步阻塞架构，具有超低延迟（200~600us）以及超高吞吐量的特点。  
在物流行业大数据实时计算中广泛应有，经过日均100亿次调用量验证。

NewLife.Redis 一个完整的Redis协议的功能的实现，但是redis的核心功能并没有在这里面，Redis的核心功能的实现是在NewLife.Core里面,Redis的核心功能就是有这两个类实现：

-   有一个NewLife.Caching的命名空间，里面有一个Redis类里面实现了Redis的基本功能
    
-   另一个类是RedisClient是Redis的客户端(代表对服务器的一个连接)
    

Redis的封装有两层:

-   一层是NewLife.Core里面的Redis以及RedisClient
    
-   另一层就是NewLife.Redis，这里面的FullRedis是对Redis的实现了Redis的所有的高级功能(是Redis的一个扩展)
    

```
// 实例化Redis，默认端口6379可以省略，密码有两种写法
//var rds = Redis.Create("127.0.0.1", 7);
var rds = Redis.Create("pass@", 7);
//var rds = Redis.Create("server=;password=pass", 7);
rds.Log = XTrace.Log; // 调试日志。正式使用时注释
```

**2) C#案例**

```
/**打开Program.css**/
using NewLife.Threading;
namespace Test
{
  class Program
  {
    static void Main(String[] args)
    {
      Xtrace.UseConsole(); //向控制台输出日志，方便调试使用查看结果
      FullRedis.Register(); //集合FULLREDIS，否则Redis.create会得到默认的redis对象
      Test1(); //调用静态方法
      Console.ReadKey()
    }
  }
}

static void Test1()
{
    var ic = Redis.Create("pass@", 7);//创建Redis实例，得到FullRedis对象
    //var ic = new FullRedis();//另一种实例化的方式
    //ic.Server = "";
    //ic.Db = 3;      //Redis中数据库
    ic.Log = XTrace.Log;//显示日志，进行Redis操作把日志输出，生产环境不用输出日志

    // 简单操作
    Console.WriteLine("共有缓存对象 {0} 个", ic.Count);//缓存对象数量

    ic.Set("name", "大石头");//Set K-V结构，Set第二个参数可以是任何类型
    Console.WriteLine(ic.Get<String>("name"));//Get泛型，指定获取的类型

    ic.Set("time", DateTime.Now, 1);//过期时间秒
    Console.WriteLine(ic.Get<DateTime>("time").ToFullString());
    Thread.Sleep(1100);
    Console.WriteLine(ic.Get<DateTime>("time").ToFullString());

    // 列表
    var list = ic.GetList<DateTime>("list");
    list.Add(DateTime.Now);
    list.Add(DateTime.Now.Date);
    list.RemoveAt(1);
    Console.WriteLine(list[list.Count - 1].ToFullString());

    // 字典
    var dic = ic.GetDictionary<DateTime>("dic");
    dic.Add("xxx", DateTime.Now);
    Console.WriteLine(dic["xxx"].ToFullString());

    // 队列
    var mq = ic.GetQueue<String>("queue");
    mq.Add(new[] { "abc", "g", "e", "m" });
    var arr = mq.Take(3);
    Console.WriteLine(arr.Join(","));

    // 集合
    var set = ic.GetSet<String>("181110_1234");
    set.Add("xx1");
    set.Add("xx2");
    set.Add("xx3");
    Console.WriteLine(set.Count);
    Console.WriteLine(set.Contains("xx2"));

    Console.WriteLine("共有缓存对象 {0} 个", ic.Count);
}
```

![WeiyiGeek.C#案例](https://i0.hdslb.com/bfs/article/809915a58682cb71a6bb93011628e0770114a114.png@942w_603h_progressive.webp)

总结：

-   数据库中不合法的时间处理,习惯大于小于号不习惯用等于号，这样可以处理很多意外的数据
    
-   Set的时候最好指定过期时间防止有些需要删除的数据
    
-   Redis异步尽量不用，因为Redis延迟本身很小，大概在100us-200us，再一个就是Redis本身是单线程的，异步任务切换的耗时比网络耗时还要大
    

**3) Redis简单案例**  
网站搜索的热搜词,用.NET Core和StackExchange.Redis以及Jquery-ui 主要是用了里面的autocomplete;

搜索关键字的时候不存在则添加,存在则将利用有序集合中的zincrby进行分数排序;

```
#AutoController
using AutoCompleteDemo.Common;
using Microsoft.AspNetCore.Mvc;
using System;
using System.Collections.Generic;
using System.Linq;

namespace AutoCompleteDemo.Controllers
{
    public class AutoController : Controller
    {
        private readonly IRedis _redis;
        private readonly string _searchKey = "search";
        public AutoController(IRedis redis)
        {
            _redis = redis;
        }

        public IActionResult Index()
        {
            //keys
            IList<string> keys = new List<string>()
            {
                "kobe","johnson","jabbar","west","o'neal","baylor","mccann","worthy","gasol","chamberlain",
                "fisher","odom","bynum","horry","rambis","riley","clarkson","Williams","young","Russell",
                "ingram","randle","nance","brown","deng","yi","ariza","artest","walton","vujacic",
                "james","paul","curry","park","yao","kevin","wade","rose","popovich","leonard",
                "aldridge","ginobili","duncan","lavine","rubio","garnett","wiggins","westbrook","durant","ibaka",
                "nowitzki","pierce","crawford","love","smith","iguodala","barnes","green","thompson","harden",
                "lillard","mccollum","lin","jackson","nash","stoudemire","whiteside","dragic","Howard","batum"
            };

            //init
            Random random = new Random();
            var tran = _redis.GetTransaction();
            for (int i = 0; i < 2000000; i++)
            {
                tran.SortedSetIncrementAsync(_searchKey, keys[random.Next(0, 70)], 1);
            }
            tran.ExecuteAsync();

            return View();
        }

        public IActionResult GetHotKey(string key="")
        {
            if (string.IsNullOrEmpty(key))
            {//default
                var res = _redis.ZRevRange(_searchKey, 0, 9);   //将分数从高到低排序 desc
                var list = (from i in res select i.ToString());
                return Json(list);
            }
            else
            {//by user input
                var res = _redis.ZRevRange(_searchKey, 0, -1);
                var list = (from i in res select i.ToString()).Where(x => x.Contains(key)).Take(10).ToList();
                return Json(list);
            }
        }

        [HttpPost]
        public IActionResult SetHotKey(string key)
        {
            if (!string.IsNullOrWhiteSpace(key))
            {
                _redis.ZIncrby(_searchKey,key);
                //other
                //...
                return Json(new { code = "000", msg = "OK" });
            }
            else
            {
                return Json(new { code = "999", msg = "keyword can not be empty!" });
            }
        }
    }
}

#Index.cshtml
@{
    ViewData["Title"] = "Auto Complete";
}
<div class="row">
    <div class="col-md-6 col-md-offset-4" style="padding-top:50px;">
        <input id="key" name="key" placeholder="search" class="form-control col-md-4">
        <button class="btn btn-primary" type="button" id="searchSubmit">Search</button>
        <div id="result"></div>
    </div>
</div>
@section scripts{
    <script type="text/javascript">
        $(function () {
            //show hot keyword
            $("#key").autocomplete({
                source: function (request, response) {
                    $.ajax({
                        url: "@Url.Action("GetHotKey", "Auto")",  //关联上面的方法
                        dataType: "json",
                        data: {
                            key: request.term
                        },
                        success: function (data) {
                            response(data);
                        }
                    });
                },
            });

            //search
            $("#searchSubmit").click(function () {
                $.ajax({
                    url: "@Url.Action("SetHotKey", "Auto")",
                    dataType: "json",
                    type: "POST",
                    data: { key: $("#key").val() },
                    success: function (data) {
                        if (data.code == "000") {
                            $("<p>search successful!</p>").appendTo("#result");
                        } else {
                            $("<p>"+data.msg+"</p>").appendTo("#result");
                        }
                    }
                });
            });
        });
    </script>
}
```

![WeiyiGeek.搜索热词](https://i0.hdslb.com/bfs/article/2c3d5b109cdb318e2b26a7d1b8fe00e6a89db24f.png@942w_468h_progressive.webp)

描述：采用Jedis jar包进行利用java操作Redis，所以必须将其导入工程的lib中，下面简单演示一下使用;

config.properties

```
# Connection Redis Configure
RedisUrl=10.20.10.248:6379
RedisAuth=weiyigeek.top
```

基础实例:

```
import redis.clients.jedis.Jedis;
/**
 * 采用 Java 进行 Redis 连接使用测试
 * @author WeiyiGeek
 */
public class RedisDemo1 {
    public static void main(String[] args) {
        String[] RedisUrl=null;
        String RedisAuth=null;
        try {
            //1.创建一个属性配置对象并打开配置文件
            Properties prop = new Properties();
            InputStream ins = RedisDemo1.class.getClassLoader().getResourceAsStream("config.properties");
            prop.load(ins);
            RedisUrl=prop.getProperty("RedisUrl").split(":");
            RedisAuth=prop.getProperty("RedisAuth");
            System.out.println("RedisServer:"+ prop.getProperty("RedisUrl") + "\nPassword:" + RedisAuth );

            //2.连接到 redis 服务
            Jedis jedis = new Jedis(RedisUrl[0],Integer.parseInt(RedisUrl[1]));
            System.out.println("正在Redis认证连接...");
            jedis.auth(RedisAuth);

            //3.查看链接是否成功并且服务是否运行
            System.out.println("服务正在运行: "+jedis.ping()+"\n");

            //4.选择一号库
            jedis.select(1);

            //5.设置 redis 字符串数据与 获取存储的数据并输出
            String key = "WeiyiGeek";
            jedis.set(key, "www.weiyigeek.top");
            jedis.set("count", "1");
            jedis.setex("Key", 60, "60 Sec");
            System.out.println("当前数据库总键数:"+jedis.dbSize());
            if(jedis.exists(key)) {
                System.out.println("Redis中WeyiGeek键存储的字符串为:"+ jedis.get(key));
                System.out.println("其类型为 : " + jedis.type(key));
            }
            //value + 1 并返回其其值
            System.out.println("incr key = " + jedis.incr("count"));
            System.out.println("incrby key 5 = " + jedis.incrBy("count", 5));

            //6.采用迭代器进行遍历所有键值
            Set<String> keys = jedis.keys("*");
            for (Iterator iterator = keys.iterator(); iterator.hasNext();) {
                String k = (String) iterator.next();
                if (jedis.type(k).equalsIgnoreCase("string")) {
                    System.out.println(k + " - " + jedis.get(k));
                }

            }
            System.out.println("");
            //清空所有的key
            jedis.flushAll();

            //关闭释放jedis连接资源
            jedis.disconnect();

        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```

执行结果:

```
RedisServer:10.20.10.248:6379
Password:weiyigeek.top
正在Redis认证连接...
服务正在运行: PONG

当前数据库总键数:3
Redis中WeyiGeek键存储的字符串为:www.weiyigeek.top
其类型为 : string
incr key = 2
incrby key 5 = 7
count - 7
WeiyiGeek - www.weiyigeek.top
Key - 60 Sec
```

描述: 在客户端使用哨兵时，需要先先连接一个Sentinel实例，然后再使用 `SENTINEL get-master-addr-by-name master-name` 获取Redis地址信息。

通过连接返回的Redis地址信息，通过ROLE命令查询是否是Master。如果是连接进入正常的服务环节。否则应该断开重新查询。

当Sentinel发起failover后，切换了新的Master，Sentinel会发送 CLIENT KILL TYPE normal命令给客户端，客户端需要主动断开对老的Master的链接，然后重新查询新的Master地址，再重复走上面的流程。这样的方式仍然相对不够实时，可以通过Sentinel提供的`Pub/Sub`来更快地监听到failover事件加快重连。

如果需要实现读写分离/读走Slave，那可以走`SENTINEL slaves`来查询Slave列表并连接。

由于Redis是异步复制，所以Sentinel其实无法达到强一致性，它承诺的是最终一致性：最后一次failover的Redis Master赢者通吃，其他Slave的数据将被丢弃，重新从新的Master复制数据, 此外还有前面提到的分区带来的一致性问题。

其次，Sentinel的选举算法依赖时间，因此要确保所有机器的时间同步，如果发现时间不一致，Sentinel实现了一个TITL模式来保护系统的可用性。

**脚本实例:**

```
from redis.sentinel import Sentinel
sentinel = Sentinel([('localhost', 26379)], socket_timeout=0.1) # 连接到sentinel服务实例
print(sentinel.discover_master('mymaster'))  # master 实例发现
print(sentinel.discover_slaves('mymaster'))  # slave 实例发现
master = sentinel.master_for('mymaster', socket_timeout=0.1)
master.set('foo', 'bar')
slave = sentinel.slave_for('mymaster', socket_timeout=0.1)
slave.get('foo')
'bar'
```

Tips : (可选) 客户端可以通过`SENTINEL sentinels`来更新自己的Sentinel实例列表。

**错误信息:**

```
/opt/soft/redis-6.2.5# make PREFIX=${REDIS_DIR} install
  # cd src && make install
  # make[1]: Entering directory '/opt/soft/redis-6.2.5/src'
  #     CC adlist.o
  # In file included from adlist.c:34:
  # zmalloc.h:50:10: fatal error: jemalloc/jemalloc.h: No such file or directory
  #   50 | #include <jemalloc/jemalloc.h>
  #       |          ^~~~~~~~~~~~~~~~~~~~~
  # compilation terminated.
  # make[1]: *** [Makefile:368: adlist.o] Error 1
  # make[1]: Leaving directory '/opt/soft/redis-6.2.5/src'
  # make: *** [Makefile:9: install] Error 2
```

**错误原因:** 错误的本质是我们在开始执行 make 时遇到了错误（大部分是由于gcc未安装），然后我们安装好了gcc 后，我们再执行make ,这时就出现了`jemalloc/jemalloc.h: No such file or directory`。因为上次的编译失败有残留的文件，所以需要清理下然后重新编译就可以了。  
**解决办法:** 清理上次编译残留文件重新编译

make distclean && make

Tips: 网上其它采用的方式是 `make MALLOC=libc` ，虽然该方法最后也是可以成功安装好 redis 但是有一些隐患的，首先我们要知道redis 需要使用内存分配器的， `make MALLOC=jemalloc` 就是指定内存分配器为 jemalloc ，`make MALLOC=libc` 就是指定内存分配器为 libc ，这个是有安全隐患的，jemalloc 内存分配器在实践中处理内存碎片是要比libc 好的，而且在README.md 文档也说明到了，jemalloc内存分配器也是包含在源码包里面的，可以在 deps 目录下看到 jemalloc 目录。

**解决办法:** redis-cli -h 172.16.24.214 -a weiyigeek -c --no-auth-warning

**报错信息:**

WARNING overcommit\_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit\_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit\_memory=1' for this to take effect.

**解决办法:**

```
#方式1:root权限、临时生效不用重启Redis服务
echo 1 > /proc/sys/vm/overcommit_memory

#方式2:永久生效但是需要重启redis服务
vim /etc/sysctl.conf
vm.overcommit_memory=1
```

**错误信息:**

\[ERR\] Node 172.16.243.94:6379 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.

**问题原因:** 由于前面初始化集群时卡住，导致部分节点的nodes.conf文件中更新了新节点数据，需要删除数据，存在旧的集群相关配置未进行清理以及数据卡槽不为空。  
**解决方法:** 重置集群节点。

```
# 方式1.删除生成的配置文件 `/etc/redis/cluster/run/nodes_8000.conf`;如果不行则说明现在创建的结点包括了旧集群的结点信息,需要删除redis的持久化文件后再重启redis，比如:appendonly.aof、dump.rdb
rm -rf appendonly.aof  dump.rdb  nodes.conf

# 方式2.在集群相关节点(登录每个redis节点重置)上执行如下redis子命令。
redis-cli -p 7001 -c
flushdb
cluster reset
```

**错误信息:**

Node 172.16.24.214:6379 replied with error: ERR Unknown node 436c6a1d7e4c5f782e1e0620b831211ebb0a41a4

**问题原因:** 节点地址与其在集群中ID不匹配。  
**解决办法:** 重新强制节点群集与制定节点握手使其加入到集群中。如`redis-cli -h 172.16.24.214 -a weiyigeek -c cluster meet 172.16.243.97 6379`命令

**错误信息:**

```
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
[WARNING] Node 192.168.1.100:7000 has slots in importing state (5798,11479).
[WARNING] Node 192.168.1.100:7001 has slots in importing state (1734,5798).
[WARNING] Node 192.168.1.101:7002 has slots in importing state (11479).
[WARNING] The following slots are open: 5798,11479,1734
>>> Check slots coverage...
[OK] All 16379 slots covered.
```

**解决办法:** 取消对指定卡槽 slot 进行导入（import）或者迁移（migrate）。如`redis-cli -h 172.16.243.97 -a weiyigeek -c cluster setslot 5798 stable`

**问题原因:** 最开始部署时从使用docker-comopose部署的方法套用过来，由于redis.conf配置文件中参数`cluster-announce-ip`选项配置了宿主机的ip，当初始化时集群节点之间需要通讯，但是再k8s中宿主机ip已经不适用，导致无法通讯一致卡住在那个位置。  
**解决办法:** 配置中禁用 `cluster-announce-ip` 选项，采用上面的update脚本进行解决。

错误信息: `【2】can't connect to node 192.168.1.1:6379`  
问题原因: redis的配置文件，发现，在配置文件中配置bind 127.0.0.1这个地址,修改指定的IP地址，可以同时指定127.0.0.1，这样本机和ip地址都可以访问

```
# 方式1.指定接口
bind 192.168.1.13 127.0.0.1

# 方式2.任意接口
bind 0.0.0.0
```

**错误原因:** 由于redis clster集群节点宕机（或节点的redis服务重启），导致了部分slot数据分片丢失；在在删除节点的时候一定要注意删除的是否是Master主节点。  
**解决办法:** 官方是推荐使用fix 来修复集群,修复完成后检测下是否正确（查看别的节点）

```
# 修复集群和槽的重复分配问题（将所有slots迁移到一处）
redis-cli -h 172.16.243.97 -a weiyigeek --cluster fix --cluster-fix-with-unreachable-masters 172.16.24.214:6379
```

**特别注意:** 在部分节点重启后重新回到集群中的过程期间，在check集群状态的时候会出现"\[ERR\] Not all 16379 slots are covered by nodes."这个报错，  
需要稍微等待一会，等重启节点完全回到集群中后，这个报错就会消失！

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>             如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！          
> 
> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

  
文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】