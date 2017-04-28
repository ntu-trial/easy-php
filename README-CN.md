<p align="center"><img width="60%" src="logo.png"><p>

<p align="center"> 从0开始搭建一个属于你自己的PHP框架 <p>

### 如何构建一个自己的PHP框架

为什么我们要去构建一个自己的PHP框架？可能绝大多数的人都会说“市面上已经那么多的框架了，还造什么轮子？”。我的观点“造轮子不是目的，造轮子的过程中汲取到知识才是目的”。

那怎样才能构建一个自己的PHP框架呢？大致流程如下：

```
　　　　
入口文件 ----> 注册自加载函数
        ----> 注册错误(和异常)处理函数
        ----> 加载配置文件
        ----> 请求对象
        ----> 路由　
        ---->（控制器 <----> 数据模型）
        ----> 响应对象
        ----> json
        ----> 视图渲染数据
```

除此之外我们还需要单元测试、nosql支持、接口文档支持、一些规范辅助脚本等。大致的框架目录如下：

###  框架目录一览

```
app                             [应用目录]
├── demo                        [模块目录]
│   ├── controllers             [控制器目录]
│   ├── Index.php               [默认控制器文件，输出json数据]
│   ├── logics                  [逻辑层，主要写业务逻辑的地方]
│   │   └── HttpException.php   [例如http异常类]
│   └── models                  [数据模型目录]
│       └── Index.php           [默认模式文件，定义一一对应的数据模型]
├── config                      [配置目录]
│    └── demo                   [模块配置目录]
│        ├── common.php         [通用配置]
│        └── database.php       [数据库配置]
doc                             [接口文档目录]
framework                       [Easy PHP核心框架目录]
├── config                      [默认配置文件目录]
│      ├── common.php           [默认通用配置文件]
│      └── database.php         [默认数据库配置文件]
├── exceptions                  [异常目录]
│      ├── CoreHttpException.php[核心http异常]
├── handles                     [框架运行时挂载处理机制类目录]
│      ├── Handle.php           [处理机制接口]
│      ├── ErrorHandle.php      [错误处理机制类]
│      ├── ExceptionHandle.php  [未捕获异常处理机制类]
│      ├── ConfigHandle.php     [配置文件处理机制类]
│      ├── NosqlHandle.php      [nosql处理机制类]
│      └── RouterHandle.php     [路由处理机制类]
├── orm                         [对象关系模型]
│      ├── Interpreter.php      [sql解析器]
│      ├── DB.php               [数据库操作类]
│      └── db                   [数据库类目录]
│          └── Mysql.php        [mysql操作类]
├── nosql                       [nosql类目录]
│    └── Redis.php              [redis类文件]
├── App.php                     [框架类]
├── Container.php               [服务容器]
├── Load.php                    [自加载类]
├── Request.php                 [请求类]
├── Response.php                [响应类]
├── run.php                     [框架应用启用脚本]
public                          [公共资源目录，暴露到万维网]
├── frontend                    [application frontend directory]
│    ├── src                    [frontend resource directory]
│    │   ├── components         [frontend component directory]
│    │   ├── views              [frontend view directory]
│    │   ├── images             [frontend image directory]
│    └── dist                   [frontend build destination]
├── index.php                   [入口文件]
runtime                         [临时目录]
tests                           [单元测试目录]
├── demo                        [模块名称]
│      └── DemoTest.php         [测试演示]
├── TestCase.php                [测试用例]
vendor                          [composer目录]
.git-hooks                      [git钩子目录]
├── pre-commit                  [git pre-commit预commit钩子示例文件]
└── commit-msg                  [git commit-msg示例文件]
.env                            [环境变量文件]
.gitignore                      [git忽略文件配置]
cli                             [框架cli模式运行脚本]
composer.json                   [composer配置文件]
composer.lock                   [composer lock文件]
phpunit.xml                     [phpunit配置文件]
README.md                       [readme文件]

```

# 框架模块说明：

###  入口文件

定义一个统一的入口文件，对外提供统一的访问文件。对外隐藏了内部的复杂性，类似企业服务总线的思想。

[file: public/index.php]()

###  自加载模块

使用spl_autoload_register函数注册自加载函数到__autoload队列中，配合使用命名空间，当使用一个类的时候可以自动载入(require)类文件。注册完成自加载逻辑后，我们就可以使用use和配合命名空间申明对某个类文件的依赖。

[file: framework/Load.php]

###  错误和异常模块

脚本运行期间：

- 错误:

通过函数set_error_handler注册用户自定义错误处理方法，但是set_error_handler不能处理以下级别错误，E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、 E_COMPILE_ERROR、 E_COMPILE_WARNING，和在 调用 set_error_handler() 函数所在文件中产生的大多数 E_STRICT。所以我们需要使用register_shutdown_function配合error_get_last获取脚本终止执行的最后错误，目的是对于不同错误级别和致命错误进行自定义处理，例如返回友好的提示的错误信息。

- 异常:

通过函数set_exception_handler注册未捕获异常处理方法，目的捕获未捕获的异常，例如返回友好的提示和异常信息。

[file: framework/hanles/ErrorHandle.php]

###  配置文件模块

加载框架自定义和用户自定义的配置文件

[file: framework/hanles/ConfigHandle.php]

###  路由模块

通过用户访问的url信息，通过路由规则执行目标控制器类的的成员方法。

[file: framework/hanles/RouterHandle.php]

###  输入和输出

- 定义请求对象：包含所有的请求信息
- 定义响应对象：申明响应相关信息

[file: framework/Request.php]
[file: framework/Response.php]

###  传统的MVC模式提倡为MCL模式

传统的MVC模式包含model-view-controller层，绝大多时候我们会把业务逻辑写到controller层或model层，但是慢慢的我们会发现代码难以阅读、维护、扩展，所以我在这里强制增加了一个logics层。至于，逻辑层里怎么写代码怎么，完全由你自己定义，你可以在里面实现一个工具类，你也可以在里面再新建子文件夹并在里面构建你的业务逻辑代码，你甚至可以实现一个基于责任连模式的网关(我会提供具体的示例)。这样看来，我们的最终结构是这样的:

- M: models, 职责只涉及数据模型相关操作
- C: controllers, 职责对外暴露资源，前后端分离架构下controllers其实就相当于json格式的视图
- L: logics, 职责灵活实现所有业务逻辑的地方

视图View去哪了？由于选择了前后端分离和SPA(单页应用), 所以传统的视图层也因此去掉了，详细的介绍看下面。

[file: app/*]

###  使用Vue作为视图

完全的前后端分离，数据双向绑定，模块化等等的大势所趋。

[file: ]

###  数据库关系对象模型

[file: framework/orm/*]

###  服务容器

什么是服务容器？

服务容器听起来很浮，按我的理解简单来说就是提供一个第三方的实体，我们把业务逻辑需要使用的类或实例注入到这个第三方实体类中，当需要获取类的实例时我们直接通过这个第三方实体类获取。

服务容器的意义？

用设计模式来讲：其实不管设计模式还是实际编程的经验中，我们都是强调“高内聚，松耦合”，我们做到高内聚的结果就是每个实体的作用都是极度专一，所以就产生了各个作用不同的实体类。在组织一个逻辑功能时，这些细化的实体之间就会不同程度的产生依赖关系，对于这些依赖我们通常的做法如下：

```
class Demo
{
    public function __construct()
    {
        // 类demo直接依赖RelyClassName
        $instance = new RelyClassName();
    }
}
```

这样的写法没有什么逻辑上的问题，但是不符合设计模式的“最少知道原则”，因为之间产生了直接依赖，整个代码结构不够灵活是紧耦合的。所以我们就提供了一个第三方的实体，把直接依赖转变为依赖于第三方，我们获取依赖的实例直接通过第三方去完成以达到松耦合的目的，这里这个第三方充当的角色就类似系统架构中的“中间件”，都是协调依赖关系和去耦合的角色。最后，这里的第三方就是所谓的服务容器。

在实现了一个服务容器之后，我把Rquest,Config等实例都以单例的方式注入到了服务容器中，当我们需要使用的时候从容器中获取即可，十分方便。使用如下：

```
// 注入单例
App::$container->setSingle('别名，方便获取', '对象/闭包/类名');

// 例，注入Request实例
App::$container->setSingle('request', $request);
// 获取Request对象
App::$container->getSingle('request');
```

###  Nosql的支持

提供对nosql的支持，提供全局单例对象，借助我们的服务容器我们在框架启动的时候，通过配置文件的配置把需要的nosql实例注入到服务容器中。目前我们支持redis/memcahed/mongodb。

如何使用？如下，

```
// 获取redis对象
App::$container->getSingle('redis');
// 获取memcahed对象
App::$container->getSingle('memcahed');
// 获取mongodb对象
App::$container->getSingle('mongodb');
```

[file: framework/nosql/*]

###  接口文档生成和接口模拟

通常我们写完一个接口后，接口文档是一个问题，我们这里使用Api Blueprint协议完成对接口文档的书写和mock，同时我们配合使用Swagger通过接口文档实现对接口的实时访问。

Api Blueprint接口描述协议选取的工具是snowboard,具体使用说明如下：

**接口文档生成说明**

```
cd docs/apib

./snowboard html -i demo.apib -o demo.html -s

open the website, http://localhost:8088/
```

**接口mock使用说明**

```
cd docs/apib

./snowboard mock -i demo.apib

open the website, http://localhost:8087/demo/index/hello
```

[file: docs/*]

###  单元测试

基于phpunit的单元测试，写单元测试是个好的习惯。

如何使用？

tests目录下编写测试文件，具体参考tests/demo目录下的DemoTest文件,然后运行：

```
 vendor/bin/phpunit
```

测试断言示例：

```
/**
 *　演示测试
 */
public function testDemo()
{
    $this->assertEquals(
        'Hello Easy PHP',
        // 执行demo模块index控制器hello操作，断言结果是不是等于Hello Easy PHP　
        App::$app->get('demo/index/hello')
    );
}
```

[phpunit断言文档语法参考](https://phpunit.de/manual/current/zh_cn/appendixes.assertions.html)

[file: tests/*]

###  Git钩子配置

目的规范化我们的项目代码和commit记录。

- 代码规范：配合使用php_codesniffer，在代码提交前对代码的编码格式进行强制验证。
- commit-msg规范：采用ruanyifeng的commit msg规范，对commit msg进行格式验证，增强git log可读性和便于后期查错和统计log等, 这里使用了[Treri](https://github.com/Treri)的commit-msg脚本，Thx~。

[file: ./git-hooks/*]

# 如何使用?

```
网站服务模式:

步骤 1: cd public
步骤 2: php -S localhost:666
步骤 3: 打开网址 'http://localhost:666'

例如, http://localhost:666/Demo/Index/hello

--------------------------------------------

客户端脚本模式:

php cli --method=<module.controller.action> --<arguments>=<value> ...

例如, php cli --method=demo.index.get --username=easy-php

--------------------------------------------

获取帮助:

使用命令 php cli 或者 php cli --help
```