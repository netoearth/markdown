![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

### [centos7](https://so.csdn.net/so/search?q=centos7&spm=1001.2101.3001.7020)安装nodejs指定版本

-   -   [使用nvm安装nodejs指定版本](https://blog.csdn.net/aa390481978/article/details/106684313#nvmnodejs_20)
    -   -   -   [1\. 下载nvm版本](https://blog.csdn.net/aa390481978/article/details/106684313#1_nvm_21)
            -   [2\. 把nvm加入系统变量](https://blog.csdn.net/aa390481978/article/details/106684313#2_nvm_25)
            -   [3\. 安装指定版本](https://blog.csdn.net/aa390481978/article/details/106684313#3__32)
            -   [4\. nvm其他常用命令](https://blog.csdn.net/aa390481978/article/details/106684313#4_nvm_39)

有时候，你发现通过以下的方式安装nodejs，由于系统环境等的问题，最后结果并不是你要的nodejs版本

-   修改nodesource后，进行安装
-   使用`n` 安装nodejs

修改nodesource后，进行安装

```
curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -
sudo yum install –y nodejs
node –version
```

使用`n` 安装nodejs

```
sudo yum install –y nodejs
sudo npm install -g n
n 9.2.0
node –version
```

## 使用nvm安装nodejs指定版本

#### 1\. 下载nvm版本

```
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

#### 2\. 把nvm加入系统变量

使得在终端可以使用nvm命令

```
source   ~/.bashrc

nvm --version
```

#### 3\. 安装指定版本

```
nvm install 9.2.0

node -v
```

#### 4\. nvm其他常用命令

```
  nvm ls-remote  查看远程版本列表
  nvm install 8.0.0                     Install a specific version number
  nvm use 8.0                           Use the latest available 8.0.x release
  nvm run 6.10.3 app.js                 Run app.js using node 6.10.3
  nvm exec 4.8.3 node app.js            Run `node app.js` with the PATH pointing to node 4.8.3
  nvm alias default 8.1.0               Set default node version on a shell
  nvm alias default node                Always default to the latest available node version on a shell
```