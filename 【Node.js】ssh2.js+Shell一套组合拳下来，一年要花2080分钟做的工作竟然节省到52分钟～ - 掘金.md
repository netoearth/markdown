## 前言

这是[【Node.js实战】](https://juejin.cn/column/7125427604525416455 "https://juejin.cn/column/7125427604525416455")专栏内的第6篇文章，专栏是分享使用Node.js技术编写实用脚本技巧。

专栏现有文章：

1.  [仿jsDoc写一个最简单的文档生成](https://link.juejin.cn/?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7119105933661175838 "https://juejin.cn/post/7119105933661175838")
2.  [50+行代码搞定一行命令更新Npm包](https://juejin.cn/post/7068969844607189029 "https://juejin.cn/post/7068969844607189029")
3.  [玩转nodeJs文件模块](https://link.juejin.cn/?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7072972877628178440 "https://juejin.cn/post/7072972877628178440")
4.  [Node.js操作Dom ，轻松hold住简单爬虫](https://juejin.cn/post/7184260401139220536 "https://juejin.cn/post/7184260401139220536")
5.  [【Node.js】写一个数据自动整理成表格的脚本](https://link.juejin.cn/?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7188320646412107835 "https://juejin.cn/post/7188320646412107835")

欢迎读者关注[【Node.js实战】](https://juejin.cn/column/7125427604525416455 "https://juejin.cn/column/7125427604525416455")专栏。

进入了新的一年，团队被分配了新的工作内容——每周巡检。

巡检工作简单，但需要人工重复性地登陆远程服务器、输入重复的命令，然后将命令的结果记录下来。每做一次估计花`40`分钟，但要每周做，一年52周，一年下来就要花`40*52=2080`分钟，这仅仅是团队一个人一年要花的时间。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9cbece5601541958ad598809eb0adbe~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

不能这么玩呀，纯纯工具人，所以我一直在思考如何用程序帮我自动巡检掉。这篇文章的出现，说明我的想法方向是正确的，收益可观一年要花**2080分钟**，被我减到**52 分钟**。

如果再扩展程序帮助到团队，这个公式将从`40*52*团队人数`变成`1*52*团队人数`，时间等于金钱。

未自动巡检：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a14a84a7548406181296013b71ac1c0~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

手动连接登陆远程服务器，再输入相应的命令获取结果，然后人工依据结果判断是否异常，相当麻烦，而且我要执行的命令不止一条。

自动巡检：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcf6fc32b1f8476889802dcb1dbe05b8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

运行macOS笔记本创建好的快捷指令，它会自动巡检服务器，并且巡检完成后直接打开巡检结果表格。当然没有macOS依然可以，但就是没有快捷指令这步，需要自己执行程序。

完整源码：[blog/ssh](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FCatsAndMice%2Fblog%2Ftree%2Fmain%2Fssh "https://github.com/CatsAndMice/blog/tree/main/ssh")

## 实现

### 实现难点

自动化巡检思路简单，思路如下：

本地程序连接登陆远程服务器→本地shell命令远程执行→本地程序获取命令结果→结果数据整理成表格

实现过程中主要有以下两个难点：

-   Node.js本地运行程序如何连接登陆远程服务器
-   登陆远程服务器帐号权限不足，在使用`sudo`命令时，如何自动输入密码

### 实现细节

解决Node.js本地运行程序如何连接登陆远程服务器：

社区已有的方案[ssh2](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmscdex%2Fssh2 "https://github.com/mscdex/ssh2")，它是用纯JavaScript为Node.js编写的SSH2客户端和服务器模块。可以使用它连接到远程服务器，并且[ssh2](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmscdex%2Fssh2 "https://github.com/mscdex/ssh2")提供了方法可以执行shell命令。

[ssh2](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmscdex%2Fssh2 "https://github.com/mscdex/ssh2")官方案例：

```
//...
const { Client } = require('ssh2');
const conn = new Client();
conn.on('ready', () => {
  console.log('Client :: ready');
  //执行uptime
  conn.exec('uptime', (err, stream) => {
    if (err) throw err;
    stream.on('close', (code, signal) => {
      console.log('Stream :: close :: code: ' + code + ', signal: ' + signal);
      conn.end();
    }).on('data', (data) => {
      //监听数据
      console.log('STDOUT: ' + data);
    }).stderr.on('data', (data) => {
      console.log('STDERR: ' + data);
    });
  });
})
//...
复制代码
```

官方案例仅执行一条shell命令，当按照顺序依次执行一条以上的命令，官方的这个写法会非常麻烦。例如：首先执行`docker ps -a -q`获取所有docker容器`id`，然后再`docker logs --tail 200 id`

```
 //...
 // 获取docker所有容器ID
 conn.exec('docker ps -a -q', (err, stream) => {
    if (err) throw err;
    stream.on('close', (code, signal) => {
      /**
        docker ps -a -q命令执行完成
        再执行docker logs -f --tail 200 id
      */
      conn.exec(`docker logs  --tail 200 ${id}`,(err,stream)=>{
         if (err) throw err;
          stream.on('close', () => {
          //如果命令再复杂点，还需要继续这样写下去
          
          }).on('data', (data) => {
            console.log( data);
          }).stderr.on('data', (data) => {
            console.log(data);
          });
      })
      
    }).on('data', (data) => {
      console.log('STDOUT: ' + data);
    }).stderr.on('data', (data) => {
      console.log('STDERR: ' + data);
    });
  });
 //...
复制代码
```

要想写法整洁点，我们需要再给 `exec`方法用`Promise`包一层。

`execFn.js:`

```
module.exports = (c = conn) => {
    return (command) => {
        return new Promise((resolve, reject) => {
            c.exec(command, (err, stream) => {
                if (err) {
                    reject(err)
                    return
                }
                let result = ''
                stream.on('close', () => {
                    resolve(String(result))
                }).on('data', (data) => {
                    //data数据是Buffer类型，需要转化成字符串
                    result += data
                })
            })
        })
    }
}
复制代码
```

包一层后，再执行命令：

```
const execFn = require('./execFn.js')

module.exports = (config, conn) => {
    conn.on('ready',async ()=>{
      const exec = execFn(conn)
      const result = await exec('docker ps -a -q')
      //...
      exec(`docker logs --tail 200 ${id}`)
    })
    //...
}
复制代码
```

这样代码会显得更整洁点，使用也更方便。

解决登陆远程服务器帐号权限不足，在使用`sudo`命令时，如何自动输入密码，可行方案有两种：

-   简单粗暴，直接使用`root`帐号密码进行登陆，这样即可不用考虑如何跳过密码输入的交互
-   使用shell管道命令`echo '密码' | sudo -S 命令`

`root`帐号密码团队不能给到我，所以我采用了后者来解决。

shell实现自动输入密码方法不只有使用管道命令`echo '密码' | sudo -S 命令`，还有其他的方法，但它在自动巡检的场景中是最合适的，它不需要额外要求服务器下载其他工具包，像`expect`指令它就需要安装expect包。巡检不只巡检一台服务器，如果每台都安装expect包，这工作量也烦人。

未自动输入密码：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b28201ad07c4ddcbcb19053fb46bd98~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

自动输入密码：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8714f7b2f984072991e6fcbc5d03d86~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

至此，自动化巡检难点之处已解决，下面的工作就是以执行shell命令返回的结果判断服务器状态是否正常，如：团队巡检文档规定当执行 `docker info |grep -A 5 "WARNING"`时，如果有返回结果则为异常。

```
//...
const before = `echo "${config.password}" | sudo -S `
exec(before + 'docker info |grep -A 5 "WARNING"').then((content) => {
    if (content) {
        rol[2] = '异常'
    }
})
//...
复制代码
```

该部分逻辑以团队巡检文档内容为准，不过多赘述，该部分代码在[sshServer.js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FCatsAndMice%2Fblog%2Fblob%2Fmain%2Fssh%2FsshServer.js "https://github.com/CatsAndMice/blog/blob/main/ssh/sshServer.js")文件。

为了做到巡检多台服务器的目的，巡检相关的逻辑代码使用函数进行包裹并从`sshServer.js`文件中导出。

`sshServer.js:`

```
const execFn = require('./execFn.js')
//...
module.exports = (config, conn) => {
    return new Promise((resolve, reject) => {
        const exec = execFn(conn)
        conn.on('ready', async (err) => {
            if (err) reject(err)
            console.log('连接成功');
            //省略
            
        }).connect({
            ...config,
            readyTimeout: 5000
        });
    })

}
复制代码
```

所有的服务器帐号密码均放置在`config.json`文件中：

```
[
    {
        "host": "xx",
        "port": "xx",
        "username": "xx",
        "password": "xx"
    }
    //...
]
复制代码
```

在`config.json`文件涉及到服务器信息需要保密，`config.json`文件不会被提交至仓库。

目录结构如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12efd4b3a6dd4856ba28c0be6869a7e8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

最后，将巡检的结果数据整理成表格，如何将数据导出表格已有对应的文章实现说明[【Node.js】写一个数据自动整理成表格的脚本](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FCatsAndMice%2Fblog%2Fissues%2F41 "https://github.com/CatsAndMice/blog/issues/41")

思路是一样的。

`index.js`

```
const { Client } = require('ssh2');
const configs = require('./config.json')
const sshServer = require('./sshServer.js');
const fs = require('fs');
const path = require('path');
const nodeXlsx = require('node-xlsx')

const promises = []

//表格数据 二维数组
const tables = [
    ['服务器ip', 'docker是否正常运行', 'docker远程访问', 'Docker日志是否有报错信息']
]

configs.forEach((config) => {
    const conn = new Client();
    promises.push(sshServer(config, conn))
})

Promise.all(promises).then((data) => {
    data.forEach((d) => {
        if (Array.isArray(d)) {
            tables.push(d)
        }
    })
    //生成xlsx表格
    const buffer = nodeXlsx.build([{ name: '巡检', data: tables }])
    const file = path.join(__dirname, '/server.xlsx')
    fs.writeFileSync(file, buffer, 'utf-8')
})
复制代码
```

巡检结果统一暂存于`tables`数组中，以便导出。

## 实现快捷指令巡检

使用命令行巡检还是太累了。 最好是鼠标点下自动触发自动巡检。

我们可以借助Mac快捷指令自定义再简化下。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47ddc898b3044e548eb403c3f191e402~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

快捷指令可以运行Shell。这样只需要编写一个名字叫做【巡检服务器】的快捷指令。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f9bd3b37c9d414882e830084b905aac~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

运行Shell后，以WPS打开`server.xlsx`文件。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bc68d6f02e8486886aeae46859beb4a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?) ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8ad36c3e21745929717b42a77be7499~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

快捷指令添加至访达。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f5f8af3b99e4679a0e2299b36880d1e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

这样就可以轻松实现自动巡检服务器功能了。

## 总结

文章灵感来源于工作，通过使用Node.js+Shell+ssh2做到自动连接登陆远程服务器，运行相关Shell命令，检查服务器程序运行是否正常等情况。

对于程序员来说，懒，才是第一生产力！！！

如果我的文章对你有帮助，你的👍就是对我的最大支持^\_^。