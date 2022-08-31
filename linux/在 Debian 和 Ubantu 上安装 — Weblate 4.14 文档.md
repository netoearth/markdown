[Weblate](https://docs.weblate.org/zh_CN/latest/index.html)

## 硬件要求[](https://docs.weblate.org/zh_CN/latest/admin/install/venv-debian.html#hardware-requirements "此标题的永久链接")

Weblate 应该可以在任何现代硬件上正常运行，以下是在单个主机（Weblate、数据库和 Web 服务器）上运行 Weblate 所需的最低配置：

-   2 GB 的内存
    
-   2 个 CPU 核心
    
-   1 GB 的存储空间
    

内存越多越好——用于所有级别的缓存（文件系统，数据库和 Weblate ）。

许多并发用户会增加所需的 CPU 内核数量。对于数百个翻译部件，推荐至少有 4 GB 的内存。

典型的数据库存储用量大约为每 1 百万单词 300 MB。克隆仓库所需的存储空间会变化，但 Weblate 试图通过浅克隆将其大小最小化。

备注

安装 Weblate 的实际要求会因其中管理的翻译规模而大不相同。

## 安装[](https://docs.weblate.org/zh_CN/latest/admin/install/venv-debian.html#installation "此标题的永久链接")

### 系统要求[](https://docs.weblate.org/zh_CN/latest/admin/install/venv-debian.html#system-requirements "此标题的永久链接")

安装构建 Python 模块所需的依赖项（请参见 [软件要求](https://docs.weblate.org/zh_CN/latest/admin/install.html#requirements)）：

```
apt install -y \
   libxml2-dev libxslt-dev libfreetype6-dev libjpeg-dev libz-dev libyaml-dev \
   libffi-dev libcairo-dev gir1.2-pango-1.0 libgirepository1.0-dev \
   libacl1-dev libssl-dev libpq-dev libjpeg62-turbo-dev build-essential \
   python3-gdbm python3-dev python3-pip python3-virtualenv virtualenv git

```

根据您打算使用的功能来安装所需的可选依赖项（参见 [可选依赖项](https://docs.weblate.org/zh_CN/latest/admin/install.html#optional-deps)）：

```
apt install -y \
   tesseract-ocr libtesseract-dev libleptonica-dev \
   libldap2-dev libldap-common libsasl2-dev \
   libxmlsec1-dev

```

可选地安装生产服务器运行所需要的软件，参见 [运行服务器](https://docs.weblate.org/zh_CN/latest/admin/install.html#server) 、 [Weblate 的数据库设置](https://docs.weblate.org/zh_CN/latest/admin/install.html#database-setup) 、 [使用 Celery 的后台任务](https://docs.weblate.org/zh_CN/latest/admin/install.html#celery)。根据于您的安装所占的空间，您会想要在特定的服务器上运行这些部件。

本地安装的使用说明：

```
# Web server option 1: NGINX and uWSGI
apt install -y nginx uwsgi uwsgi-plugin-python3

# Web server option 2: Apache with ``mod_wsgi``
apt install -y apache2 libapache2-mod-wsgi-py3

# Caching backend: Redis
apt install -y redis-server

# Database server: PostgreSQL
apt install -y postgresql postgresql-contrib

# SMTP server
apt install -y exim4

```

### Python 模块[](https://docs.weblate.org/zh_CN/latest/admin/install/venv-debian.html#python-modules "此标题的永久链接")

提示

我们使用 virtualenv 将 Weblate 安装在一个与您的系统分离的环境中。如果您不熟悉它，查阅 virtualenv [User Guide](https://virtualenv.pypa.io/en/stable/user_guide.html "（在 virtualenv v20.16）")。

1.  为 Weblate 新建 virtualenv：
    
2.  为 Weblate 激活 virtualevn ：
    
    ```
    . ~/weblate-env/bin/activate
    
    ```
    
3.  安装包含所有可选依赖项的 Weblate：
    
    ```
    # Install Weblate with all optional dependencies
    pip install "Weblate[all]"
    
    ```
    
    请查看 [可选依赖项](https://docs.weblate.org/zh_CN/latest/admin/install.html#optional-deps) 以了解对可选依赖关系的微调情况。
    
    备注
    
    在某些 Linux 发行版上，运行 Weblate 会出现 libffi 错误：
    
    ```
    ffi_prep_closure(): bad user_data (it seems that the version of the libffi library seen at runtime is different from the 'ffi.h' file seen at compile-time)
    
    ```
    
    这是由于通过PyPI发布的二进制包与该发行版不兼容造成的。为了解决这个问题，你需要在你的系统上重建该软件包：
    
    ```
    pip install --force-reinstall --no-binary :all: cffi
    
    ```
    

### 配置 Weblate[](https://docs.weblate.org/zh_CN/latest/admin/install/venv-debian.html#configuring-weblate "此标题的永久链接")

备注

后面的步骤假定 Weblate 使用的 virtualevn 已经激活（可以通过 `. ~/weblate-env/bin/activate` 来实现）。如果不是这种情况，您必须指定 到 **weblate** 命令的完全路径为 `~/weblate-env/bin/weblate`。

1.  将文件 `~/weblate-env/lib/python3.9/site-packages/weblate/settings_example.py` 复制为 `~/weblate-env/lib/python3.9/site-packages/weblate/settings.py`。
    
2.  将新的 `settings.py` 文件中的值调整为您所需要的。您至少需要提供数据库凭据和Django密钥，但在生产安装中需要做更多的更改，参见： [调整配置](https://docs.weblate.org/zh_CN/latest/admin/install.html#configuration)。
    
3.  为 Weblate 新建数据库及其结构（例子中的设置使用 PostgreSQL，已经准备好的生产设置请查看 [Weblate 的数据库设置](https://docs.weblate.org/zh_CN/latest/admin/install.html#database-setup) ）：
    
4.  新建管理员用户账户，并将输出的密码复制到剪贴板，同时将它存储供以后使用：
    
5.  收集 Web 服务器用的静态文件 （请参见 [运行服务器](https://docs.weblate.org/zh_CN/latest/admin/install.html#server) 和 [为静态文件提供服务](https://docs.weblate.org/zh_CN/latest/admin/install.html#static-files) ）：
    
6.  压缩 JavaScript 和 CSS 文件（可选步骤，请参见 [压缩客户端资源](https://docs.weblate.org/zh_CN/latest/admin/install.html#production-compress) ）：
    
7.  启动 Celery workers。这对开发来说是不必要的，但强烈建议在其他场景这样做。更多信息请参见 [使用 Celery 的后台任务](https://docs.weblate.org/zh_CN/latest/admin/install.html#celery)：
    
    ```
    ~/weblate-env/lib/python3.9/site-packages/weblate/examples/celery start
    
    ```
    
8.  启动开发服务器（生产设置请参见 [运行服务器](https://docs.weblate.org/zh_CN/latest/admin/install.html#server) ）：
    

## 安装后[](https://docs.weblate.org/zh_CN/latest/admin/install/venv-debian.html#after-installation "此标题的永久链接")

配置，您的 Weblate 服务器现在运行了，您可以使用它来启动。

-   您可以在 `http://localhost:8000/` 访问 Weblate。
    
-   使用安装期间获得的管理凭据登录或注册新用户。
    
-   现在，您可以在 Weblate virtualenv 活动时使用 **weblate** 命令来运行 Weblate 命令。
    
-   您可以使用 Ctrl+C 来停止测试的服务器。
    
-   在 `/manage/performance/` URL （参见 [管理界面](https://docs.weblate.org/zh_CN/latest/admin/admin.html#management-interface)）上或使用 **weblate check --deploy** 来复查您安装的潜在问题，请参见 [生产设置](https://docs.weblate.org/zh_CN/latest/admin/install.html#production)。
    

### 添加翻译[](https://docs.weblate.org/zh_CN/latest/admin/install/venv-debian.html#adding-translation "此标题的永久链接")

1.  打开管理界面（ `http://localhost:8000/create/project/` ），并新建您想要翻译的项目。更多细节请参见 [项目配置](https://docs.weblate.org/zh_CN/latest/admin/projects.html#project)。
    
    这里所有需要您指定的只是项目名称及其网站。
    
2.  新建一个部件，该部件是要翻译的实际对象——它指向版本控制系统（VCS）仓库，并选择要翻译的文件。更多详细信息请参见 [部件配置](https://docs.weblate.org/zh_CN/latest/admin/projects.html#component)。
    
    此处的重要字段为：[部件名](https://docs.weblate.org/zh_CN/latest/admin/projects.html#component-name)、[源代码库](https://docs.weblate.org/zh_CN/latest/admin/projects.html#component-repo) 及用于寻找可翻译文件的 [文件掩码](https://docs.weblate.org/zh_CN/latest/admin/projects.html#component-filemask)。Weblate 支持多种格式，包括 [GNU gettext](https://docs.weblate.org/zh_CN/latest/formats.html#gettext)、[Android 字符串资源](https://docs.weblate.org/zh_CN/latest/formats.html#aresource)、[苹果 iOS 字符串](https://docs.weblate.org/zh_CN/latest/formats.html#apple)、[Java 属性](https://docs.weblate.org/zh_CN/latest/formats.html#javaprop)、[Stringsdict 格式](https://docs.weblate.org/zh_CN/latest/formats.html#stringsdict) 或 [Fluent 格式](https://docs.weblate.org/zh_CN/latest/formats.html#fluent)，更多细节见 [支持的文件格式](https://docs.weblate.org/zh_CN/latest/formats.html#formats)。
    
3.  一旦完成上面的工作（根据您 VCS 仓库的大小，以及需要翻译的信息数量，这可能是个漫长的过程），您就可以开始翻译了。