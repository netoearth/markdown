

## 一、客户端生成 [公钥](https://so.csdn.net/so/search?q=%E5%85%AC%E9%92%A5&spm=1001.2101.3001.7020)、私钥，并添加到服务器端

## 1\. 客户端生成[密钥](https://so.csdn.net/so/search?q=%E5%AF%86%E9%92%A5&spm=1001.2101.3001.7020)

```
ssh-keygen -t rsa -P ''
```

\-t表示key的类型，[rsa](https://so.csdn.net/so/search?q=rsa&spm=1001.2101.3001.7020)表示key类型 -P表示密码，-P ” 就表示空密码，也可以不用-P参数，这样就要三车回车，用-P就一次回车。运行完之后在~/.ssh目录下生成私钥id\_rsa和公钥id\_rsa.pub

## 2\. 将客户端公钥id\_rsa.pub复制到服务端

```
scp ~/.ssh/id_rsa.pub user@192.168.1.140:~
```

```
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
```

如果没有这个 authorzied\_keys 文件，可以创建一个。

## 二、设置文件权限

```
sudo chmod 755 ~/.ssh
sudo chmod 600 ~/.ssh/authorized_keys 
```

## 三、设置用户以证书登录后，可选择 sudo 操作

```
sudo visudo

%sudo ALL=(ALL:ALL) ALL
#替换这一行如下：
%sudo ALL=(ALL) NOPASSWD:ALL 
```

## 四、客户端以私钥 id\_rsa 登录

## 1\. ssh证书登录命令

```
ssh -i ~/.ssh/id_rsa username@<ssh_server_ip>
```

## 2\. scp远程拷贝命令

```
scp -i ~/.ssh/id_rsa filename username@<ssh_server_ip>:/username
```