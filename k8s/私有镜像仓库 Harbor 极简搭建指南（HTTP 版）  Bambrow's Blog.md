本文简要介绍私有镜像仓库 Harbor 的搭建指南（HTTP 版），以及使用方法。搭建部分主要参考[官网](https://goharbor.io/docs/2.0.0/install-config/)。本文基于以下版本：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>Docker: 20.10.12</span><br><span>Docker-compose: 1.29.2</span><br><span>Harbor: 2.4.1</span><br></pre></td></tr></tbody></table>

## 准备工作

### 安装 Docker 与 Docker Compose

请直接参考[官网](https://docs.docker.com/engine/install/)。这里仅给出在 CentOS 上的例子：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br></pre></td><td><pre><span>sudo yum install -y yum-utils</span><br><span>sudo yum-config-manager \</span><br><span>    --add-repo \</span><br><span>    https://download.docker.com/linux/centos/docker-ce.repo</span><br><span>sudo yum install docker-ce docker-ce-cli containerd.io</span><br><span>sudo systemctl start docker</span><br><span>sudo systemctl <span>enable</span> docker</span><br><span></span><br><span>sudo curl -L <span>"https://github.com/docker/compose/releases/download/1.29.2/docker-compose-<span>$(uname -s)</span>-<span>$(uname -m)</span>"</span> -o /usr/<span>local</span>/bin/docker-compose</span><br><span>sudo chmod +x /usr/<span>local</span>/bin/docker-compose</span><br></pre></td></tr></tbody></table>

### 安装 OpenSSL

这里仅给出 CentOS 的例子：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>yum install -y openssl</span><br></pre></td></tr></tbody></table>

官网给出了两种安装模式，在线安装包或离线安装包。其区别是离线安装包里面含有镜像，在线版本在安装时则去 Docker Hub 拉取镜像。我们这里使用离线安装包。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>wget https://github.com/goharbor/harbor/releases/download/v2.4.1/harbor-offline-installer-v2.4.1.tgz</span><br><span>tar zxvf harbor-offline-installer-v2.4.1.tgz</span><br><span><span>cd</span> harbor</span><br></pre></td></tr></tbody></table>

在 `harbor` 文件夹里可以看到有一份文件 `harbor.yml.tmpl`，这是 Harbor 的配置信息，我们复制一份并进行修改（以下仅显示修改部分）：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>cp harbor.yml.tmpl harbor.yml</span><br></pre></td></tr></tbody></table>

harbor.yml

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br></pre></td><td><pre><span># Configuration file of Harbor</span><br><span></span><br><span># The IP address or hostname to access admin UI and registry service.</span><br><span># DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.</span><br><span><span>- hostname: reg.mydomain.com</span></span><br><span><span>+ hostname: your.domain.com (自行指定)</span></span><br><span></span><br><span># http related config</span><br><span>http:</span><br><span>  # port for http, default is 80. If https enabled, this port will redirect to https port</span><br><span>  port: 80</span><br><span></span><br><span># https related config</span><br><span># 直接禁用 HTTPS</span><br><span><span>- https:</span></span><br><span><span>+ # https:</span></span><br><span>  # https port for harbor, default is 443</span><br><span><span>- port: 443</span></span><br><span><span>+ # port: 443</span></span><br><span>  # The path of cert and key files for nginx</span><br><span><span>- certificate: /your/certificate/path</span></span><br><span><span>- private_key: /your/private/key/path</span></span><br><span><span>+ # certificate: /your/certificate/path</span></span><br><span><span>+ # private_key: /your/private/key/path</span></span><br><span></span><br><span># # Uncomment following will enable tls communication between all harbor components</span><br><span># internal_tls:</span><br><span>#   # set enabled to true means internal tls is enabled</span><br><span>#   enabled: true</span><br><span>#   # put your cert and key files on dir</span><br><span>#   dir: /etc/harbor/tls/internal</span><br><span></span><br><span># Uncomment external_url if you want to enable external proxy</span><br><span># And when it enabled the hostname will no longer used</span><br><span># external_url: https://reg.mydomain.com:8433</span><br><span></span><br><span># The initial password of Harbor admin</span><br><span># It only works in first time to install harbor</span><br><span># Remember Change the admin password from UI after launching Harbor.</span><br><span><span>- harbor_admin_password: Harbor12345</span></span><br><span><span>+ harbor_admin_password: yourPassword (自行指定)</span></span><br></pre></td></tr></tbody></table>

修改完毕后，直接运行 `./install.sh`，并等待 Docker Compose 执行完毕。部署完毕后，你就可以使用这台机器的 80 端口看到 Harbor 界面了。如果需要启动 Helm Chart 的管理功能，请使用 `./install.sh --with-chartmuseum`。

## 登录与使用方法

在安装 Harbor 的本机，可以直接在 `/etc/hosts` 里配置，在 127.0.0.1 后面加上你在配置文件里定义的 `hostname`，随后可以使用如下命令直接登录：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>docker login -u admin -p yourPassword http://your.domain.com</span><br></pre></td></tr></tbody></table>

上面的密码与 `hostname` 需要自行替换。

如果在其他机器登录，首先还是需要配 `/etc/hosts`，将 `hostname` 指向安装 Harbor 的机器 IP。登录时，可能会遇到如下情况：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>$ docker login -u admin -p yourPassword http://your.domain.com</span><br><span></span><br><span>Error response from daemon: Get https://your.domain.com/v2/: dial tcp xxx.xxx.xxx.xxx:443: connect: connection refused</span><br></pre></td></tr></tbody></table>

这个原因是访问 HTTPS 被拒绝（我们只配置了 HTTP），需要关闭安全验证。修改 `/etc/docker/daemon.json` 并加入如下内容：

/etc/docker/daemon.json

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span><span>+ "insecure-registries": ["your.domain.com:port", "0.0.0.0"]</span></span><br></pre></td></tr></tbody></table>

如果是 80 端口号则可以忽略端口部分。修改完毕后，重启 Docker：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>sudo systemctl restart docker</span><br></pre></td></tr></tbody></table>

必要时，可以在安装 Harbor 的机器上重启 Harbor：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span><span>cd</span> harbor</span><br><span>docker-compose down -v</span><br><span>docker-compose up -d</span><br></pre></td></tr></tbody></table>

再次登录即可正常使用。需要注意，使用 Harbor 时，镜像需要遵循以下格式：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br></pre></td><td><pre><span></span><br><span>docker tag SOURCE_IMAGE[:TAG] your.domain.com/PROJECT_NAME/REPOSITORY[:TAG]</span><br><span>docker push your.domain.com/PROJECT_NAME/REPOSITORY[:TAG]</span><br><span>docker pull your.domain.com/PROJECT_NAME/REPOSITORY[:TAG]</span><br><span></span><br><span></span><br><span>helm repo add --username admin --password ADMIN_PASSWORD harbor http://your.domain.com/chartrepo/</span><br><span>helm plugin install https://github.com/chartmuseum/helm-push</span><br><span>helm cm-push CHART_PATH --version=<span>"CHART_VERSION"</span> harbor</span><br><span>helm repo update</span><br><span>helm search repo CHART_PATH</span><br><span>helm install RELEASE_NAME CHART_NAME</span><br></pre></td></tr></tbody></table>

其中 `PROJECT_NAME` 是你在 Harbor UI 新建的项目名，`CHART_PATH` 是存储 Helm Chart 的路径，`CHART_NAME` 是使用 `helm search` 后搜索到的 Chart 名称，`RELEASE_NAME` 是 Helm 部署后的自定义名称。