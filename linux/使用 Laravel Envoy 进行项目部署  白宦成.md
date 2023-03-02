在大型企业当中，往往会有各种各样的 CI & CD 工作流来完成项目的部署，而对于我自己的项目而言，我也希望能够有这样顺畅的交付体验，因此，我也为自己的业务加上了类似的方案，以实现顺滑的开发体验。

不过，我没有做的那么重，没有上 Ansible，也没有封装 Docker，就是标准的 LNMP 环境。只不过，我在标准的环境上加入了 **[Laravel Envoy — 一个快速部署工具](https://laravel.com/docs/master/envoy)**。

Envoy 是 Laravel 团队开发的一个在远程服务器上执行某些命令的工具，因为他同样使用 Blade 语法，因此在 Laravel 项目中接入非常方便。

## 如何使用 Laravel Envoy

### 1\. 安装 Laravel Envoy

Envoy 有多种使用方式，一种是跟随项目的使用方式，即使用 `composer require laravel/envoy --dev` 来安装，并使用 `php vendor/bin/envoy` 的方式来运作，这样的好处是你无需污染顶级目录，只在项目层面使用 Envoy 来进行部署。

不过我因为有很多项目，所以我会更加倾向于直接在全局安装 envoy，执行 `composer global requre laravel/envoy` 实现在全局安装 Enovy。全局安装完成后，当你需要执行命令，只需要执行 `envoy run deploy`，就能执行当前目录下所定义的命令。

> 除了全局安装，你也可以考虑使用 Bash alias 来实现类似的效果，将 `envoy` alias 给 `php vendor/bin/envoy` 就可以实现即使没有全局安装，也可以实现不需要手动输入前缀的特性。

### 2\. 初始化配置文件

当你需要配置时，你要做的很简单，只需要执行 `envoy init 服务器地址/昵称`就可以获得一个 `Envoy.blade.php` 文件，你可以通过修改这个文件的内容，来实现定义不同的工作。同样的，你可以将这个文件纳入到版本控制系统当中，从而实现部署脚本的跟踪和定义，对于后续的持续维护比较有帮助。

因为 Envoy.blade.php 也是采用了 Blade 的后缀，所以也同样适用了 Blade 引擎，你可以通过一个语意更加友好的方式来定义你的脚本。

```
@servers(['web' => '127.0.0.1'])

@task('deploy')
    cd /path/to/site
    git pull origin master
@endtask
```

比如，上面的脚本就是一个最基础的脚本，首先定义了这个脚本的命令会在名为 web 的服务器上执行，而这个服务器的地址是 127.0.0.1；并定义了一个任务 deploy。

### 3\. 执行部署命令

当你完成代码的编写，并提交了 Git 之后。执行命令 `envoy run deploy` ，就会自动的在 web 这个服务器上执行在 deploy 当中执行的命令。

deploy 的命令只要是正常的 bash 命令即可。

## 高级用法

### 1\. 引入其他任务

因为 Envoy 使用 Blade 语法来编写，你也可以使用 Blade 语法中的命令来引入其他文件的中的任务。因此，你可以将自己常用的命令定义出来，并将其通过 Packagist 发布出去，并在实际应用过程中，引入对应的文件来使用。

```
@import('vendor/package/Envoy.blade.php')
```

### 2\. 处理多个服务器

Envoy 支持定义多个服务器以及并行执行，对于架构相对复杂的业务，你可以定义不同的 servers ，并在对应的 task 当中来设定需要在哪些 Server 上执行。定义任务时，如果你传入了 `parallel` 参数，则可以控制命令同时在多个服务器上并行（该属性为 false 时则是串行（

```
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])
 
@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

### 3\. 定义变量或引入其他功能函数

你可以在 Envoy.blade.php 中加入 @setup 来完成变量的定义，并在后续的代码中使用。

```
@setup
    $now = new DateTime;
@endsetup
```

也可以直接使用 @include 来引入文件

```
@include('vendor/autoload.php')
 
@task('restart-queues')
    # ...
@endtask
```

### 4\. 使用命令行参数

在启动时，如果你使用诸如 `--param=value`这样的参数来调用，则可以在你的代码中直接读取对应的变量，从而实现执行时带入不同的参数。

## 更多用法

Envoy 当中还有更多的用法，这里不再一一列举，如果你感兴趣，可以直接参考其[官方文档](https://laravel.com/docs/9.x/envoy)，了解如何使用 [Envoy](https://laravel.com/docs/9.x/envoy) 达成你的部署目标。

## 总结

在你不是很着急使用 Ansible 或 更重的部署工具的时候， [Envoy](https://laravel.com/docs/9.x/envoy) 是一个不错的选择。特别是你使用 Laravel 进行开发时，Envoy 就是一个有效的工具。