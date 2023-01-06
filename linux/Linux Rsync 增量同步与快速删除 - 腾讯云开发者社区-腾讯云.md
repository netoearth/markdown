#### **增量同步**

`rsync [args] SRC [DEST]`情形：同时维护着两份不同的`data_center`，但以`old_data_center`为标准。因为权限的缘故没有开启rsync自动同步，只是每隔一段时间手动同步一下。`SRC`和`DEST`都是采用mount形式，如果每一次都完整地`copy`，耗时很长，这时候就想到采用增量同步的方法，因为两份`data_center`同时由不同人维护，所以内容略有不同，`data_center`同步的时候不光要完全同步`old_data_center`的所有内容，而且要删除自身多余的内容，保持完全一致。

http://static.cyblogs.com/1559267-20190509170938551-832776092.png

```
rsync -a 
--delete 
--progress /old_vip_data_center/test_envs/trainer/resource /vip_data_center/test_envs/trainer/resource/
```

\--delete: 删除`DEST`端存在但是`SRC`端不存在的文件，如果不使用此参数，则DEST端会同步SRC端的文件，但DEST端已有的文件不受影响。

#### **快速删除大量文件**

先建一个空目录，随便位置

用rsync删除目标目录

```
rsync --delete-before -avH --progress /local/empty_dir/ /local/trainer_test/
```

`trainer_test`清空之后可以再用`rm -rf trainer_test`删除

注意不要忘了文件夹最后的`/`

`rsync`提供了一些跟删除相关的参数

```
rsync --help | grep delete
--del an alias for --delete-during
--delete-before receiver deletes before transfer (default)
```

选项说明：-a 递归方式传输文件，并保持文件属性 --delete-before 接收者在传输之前进行删除操作 --progress 在传输时显示传输过程 -- 归档模式，表示以递归方式传输文件，并保持所有文件属性 -H 保持硬连接的文件 -v 详细输出模式 -stats 给出某些文件的传输状态

不过在使用上面的命令进行清理时，存在一个问题，清空后，目标目录的权限会和源目录的权限一样。如：`/tmp/empty`是`root：root`，而`maildrop`之前是`postfix：postdrop` ，执行之后也会`maildrop`目录的权限也会变成`root：root` 。由于-a权限是-rlptogD几个参数的集合，所以可以将og（owner:group）两个参数去掉。清空时自动保持之前的目录权限，如下：

```
rsync --delete -rlptD /tmp/empty/ /var/spool/postfix/maildrop/
```

#### **为什么rsync这么快呢？**

rm删除内容时，将目录的每一个条目逐个删除(unlink)，需要循环重复操作很多次；

rsync删除内容时，建立好新的空目录，替换掉老目录，基本没开销。

> If you want to conquer fear, don't sit home and think about it. Go out and get busy.

#### **实战**

今天因为用代码生成SQL脚本的时候，本来是说100W的数据生成一个的，结果因为一个运算符的问题导致了生成上百万的小文件。

```
while ((line = br.readLine()) != null) {
    if (count < skipHeadCount) {
        count++;
        continue;
    }
    // 每MAX_SIZE就会生成一个,MAX_SIZE=1000000
    int fileExtName = (count - skipHeadCount) / MAX_SIZE; // 当时种类count - skipHeadCount忘记打括号了
    if (fileExtName > currentFileExtName) {
        bw.flush();
        currentFileExtName = fileExtName;
        bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(new File(String.format(fileOutputPath, currentFileExtName)))));
    }
    String formatStr = genService.format(line);
    bw.write(formatStr);
    bw.newLine();
    log.info("count:{}", count);
    count++;
}
```

删除的时候会报错

实战后发现效率贵高的一种方式：

![](https://ask.qcloudimg.com/http-save/yehe-6497730/a3295db413d3fc82794bd42bef2d0c96.png?imageView2/2/w/1620)

http://static.cyblogs.com/WechatIMG461.png

#### **参考地址**

-   https://www.cnblogs.com/sayiqiu/p/10816572.html

如果大家喜欢我的文章，可以关注个人订阅号。欢迎随时留言、交流。如果想加入微信群的话一起讨论的话，请加管理员简栈文化-小助手（lastpass4u），他会拉你们进群。

文章分享自微信公众号：

![](https://open.weixin.qq.com/qr/code?username=gh_223e52b70cfa)

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

如有侵权，请联系

cloudcommunity@tencent.com

删除。