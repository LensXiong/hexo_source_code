---
layout: post
title: ﻿【Laravel 源码分析】一次请求的生命周期
date: 2019-06-15 23:19:24.000000000 +09:00
categories:
- 技术
tags:
- Laravel
toc: true
---

**
摘要：学会使用一个工具是最基本的前提，如果你能理解工具背后的原理，站在一个更高的层面去深度剖析和思考，这才是你应该做的事情。写`Laravel`源码分析系列文章，除了让自身能力不断提升，更重要的是希望完成一个个原始积累，最终让自己的思维方式从量变能够得到质变。本篇文章主要通过对源码进行分析，重点阐述`Laravel`框架一次请求的完整生命周期。整个生命周期包括以下几个部分：第一，载入 `Composer` 生成的自动加载设置；第二，创建服务容器实例并完成内核的绑定（包含`HTTP`内核和`Console`内核）；第三，从服务容器中解析处理 `HTTP`请求的 `Kernel` 实例；第四，调用`Kernel` 实例中的`handle() `方法接收一个 `HTTP` 请求，并最终生成一个 `HTTP` 响应；第五，将响应值发送给客户端；第六，终止程序，做一些善后及清理工作。结合本篇文章及源码，你将逐步明白`Laravel`中一次`HTTP`请求的完整生命周期。
**
<!-- more -->
<The rest of contents | 余下全文>
﻿
# 前言
当`Web`服务器（`Apache/Nginx`）的请求被导向单入口文件 `public/index.php`时， `index.php`中极简的代码出色的完成整个`HTTP`的请求，逻辑组织既严谨又完美，以下便是其优雅的六行代码：
```
<?php
// 加载项目依赖
require __DIR__.'/../bootstrap/autoload.php';
// 创建服务容器实例并完成内核的绑定（包含`Http`内核和`Console`内核）和注册异常处理
$app = require_once __DIR__.'/../bootstrap/app.php';
//  服务容器解析内核实例
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
//  接收请求并返回响应
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
// 发送响应到客户端
$response->send();
// 终止程序，清理中间件
$kernel->terminate($request, $response);
```
上面这段单入口文件代码完成了一次`Http`请求的完整生命周期，具体过程如下：
> 
① 载入`composer`自动加载设置，以完成项目中所有依赖的加载。
② 创建一个 `Application` 实例，作为全局的服务容器，同时完成内核的绑定（包含`Http`内核和`Console`内核）及注册异常处理。
③ 将处理请求的核心类 `Kernel` 实现实例绑定到该全局的服务容器中，以便后续通过它处理 `HTTP` 请求。
④ 将捕获的请求传递给 `Kernel` 实例的`handle()`方法进行处理，处理结果最终返回一个响应的实例。
⑤ 将返回的响应实例发送给客户端。
⑥ 执行 `Kernel` 实例上的 `terminate()` 终止程序，做一些善后及清理工作，最后退出脚本。

# 加载项目依赖
`Composer`是`PHP`中用来管理依赖（`dependency`）关系的工具，你可以在自己的项目中声明所依赖的外部工具库（`libraries`），`Composer`会帮你安装这些依赖的库文件。
在`Laravel`所有组件的加载工作，入口文件中仅需一行代码即可完成：
```
require __DIR__.'/../bootstrap/autoload.php';
```

# 服务容器实例创建
```
$app = require_once __DIR__.'/../bootstrap/app.php';
```
单入口文件的第二段代码加载`/bootstrap/app.php`，主要完成了服务实例的创建、完成内核的绑定（包含`Http`内核和`Console`内核）及注册异常处理，以下我们通过源码来看看究竟做了哪些工作？
```
// 创建服务容器实例
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);
// 将 App\Http\Kernel 实例绑定到 Illuminate\Contracts\Http\Kernel 接口
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);
// 将 App\Console\Kernel 实例绑定到 Illuminate\Contracts\Console\Kernel 接口
$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);
// 将 App\Exceptions\Handler 实例绑定到 IIlluminate\Contracts\Debug\ExceptionHandler 接口
$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);
return $app;
```
以绑定`Http`实例为例：
`singleton()` 方法会以单例方式在服务容器中将 `App\Http\Kernel `实例绑定到 `Illuminate\Contracts\Http\Kernel `接口，后续我们要获取 `App\Http\Kernel `实例，就可以通过 `Illuminate\Contracts\Http\Kernel` 接口从服务容器中获取，获取方法是 `$app->make()`：
```
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
```
# 解析内核实例
通过以上服务容器实例的创建，并完成相关内核的绑定后，就会通过服务容器的`make() `方法将内核解析出来，而解析的过程实质就是内核实例化的过程。那么内核是如何进行实例化？
```
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
```
上面创建实例的时候 `Illuminate\Contracts\Http\Kernel `接口被绑定的实例是 `App\Http\Kernel `，而 `App\Http\Kernel `继承
`HttpKernel`类，所以我们最终定位到`vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php`中：
```
// 中间件的优先顺序列表
protected $middlewarePriority = [
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Auth\Middleware\Authenticate::class,
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];
// 创建 HTTP 内核实例
public function __construct(Application $app, Router $router)
    {
        $this->app = $app;
        $this->router = $router;
        $router->middlewarePriority = $this->middlewarePriority;
        foreach ($this->middlewareGroups as $key => $middleware) {
            $router->middlewareGroup($key, $middleware);
        }
        foreach ($this->routeMiddleware as $key => $middleware) {
            $router->aliasMiddleware($key, $middleware);
        }
    }
// 注册中间件组
public function middlewareGroup($name, array $middleware)
    {
        $this->middlewareGroups[$name] = $middleware;
        return $this;
    }
//  为中间件注册别名
public function aliasMiddleware($name, $class)
    {
        $this->middleware[$name] = $class;
        return $this;
    }
```
通过以上`_construct()`构造函数我们可以看出，解析内核实例时传入两个参数为`$app`和`$router`两个实例，接下来主要完成以下里两个步骤：
① 将内核自定义的一些中间件优先注册到路由器中，目的是在实际处理 `Http`请求前调用这些中间件实现过滤请求的目的。
② 为路由中间件设置别名。


# 处理HTTP请求
```
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);]
```
创建完内核实例之后，下面我们就正式进入 `$kernel->handle() `方法内部看看` Http`请求是被如何处理的。
打开 `Illuminate\Foundation\Http\Kernel `（`App\Http\Kernel` 的父类），查看` handle ()` 方法，可以看到核心处理逻辑通过 `sendRequestThroughRouter ()`方法实现：
```
// 处理 HTTP 请求
public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();
            $response = $this->sendRequestThroughRouter($request);
        } catch (Exception $e) {
            $this->reportException($e);
            $response = $this->renderException($request, $e);
        } catch (Throwable $e) {
            $this->reportException($e = new FatalThrowableError($e));
            $response = $this->renderException($request, $e);
        }
        $this->app['events']->dispatch(
            new Events\RequestHandled($request, $response)
        );
        return $response;
    }

// 发送请求到路由（发送请求前，先调用 bootstrap() 方法运用应用的启动类）
  protected function sendRequestThroughRouter($request)
    {
        // 将 $request 实例注册到 APP 容器 供后续使用
        $this->app->instance('request', $request);
        // 清除之前 $request 实例缓存
        Facade::clearResolvedInstance('request');
        // 启动引导程序
        $this->bootstrap();
        // 发送请求至路由
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
// 启动引导程序类
public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }
// 获取应用bootstrap类
protected function bootstrappers()
    {
        return $this->bootstrappers;
    }
protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];
```
通过上面的源代码我们发现在发送请求到路由之前，先调用`bootstrap()`方法启动引导程序类，相当于对整个应用进行初始化。相关的启动有加载环境变量、全局配置、异常处理、注册门面、注册服务提供者、启动服务。具体内容可选择其中一个类的启动，查看下源码究竟是如何进行的，这里暂时就不做过多说明。

当以上启动类完成时，才真正进入`Http`的请求处理，我们重点来看一下这段代码：
```
 return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
```
发送请求至路由的代码写的非常优雅，从中我们可以看出主要完成了：
① 首先实例化管道；
② 将 `$request` 传入管道；
③ 对 `$request `执行中间件处理；
④ 中间件处理成功后，才会将请求分发到路由器进行处理，并返回响应实例。

>  注：路由器会将请求 `URL` 路径与应用注册的所有路由进行匹配，如果有匹配的路由，则先收集该路由所分配的所有路由中间件，通过这些路由中间件对请求进行过滤，所有路由中间件校验通过才会运行对应的匿名函数或控制器方法，执行相应的请求处理逻辑，最后准备好待发送给客户端的响应。

# 发送响应

发送响应由`Symfony\Component\HttpFoundation\Response`中的`send()` 方法完成，具体代码如下：
```
  public function send()
    {
        $this->sendHeaders();
        $this->sendContent();
        if (function_exists('fastcgi_finish_request')) {
            fastcgi_finish_request();
        } elseif ('cli' !== PHP_SAPI) {
            static::closeOutputBuffers(0, true);
        }
        return $this;
    }
```

# 终止程序
将响应实例发送给客户端后，程序继续运行 `$kernel->terminate() `做一些善后清理工作，并最终退出脚本。
这些善后清理工作主要包括运行终止中间件，以及注册到服务容器的一些终止回调：
```
// 终止中间件和服务容器的回调
public function terminate($request, $response)
    {
        $this->terminateMiddleware($request, $response);
        $this->app->terminate();
    }
// 终止中间件
protected function terminateMiddleware($request, $response)
    {
        $middlewares = $this->app->shouldSkipMiddleware() ? [] : array_merge(
            $this->gatherRouteMiddleware($request),
            $this->middleware
        );
        foreach ($middlewares as $middleware) {
            if (! is_string($middleware)) {
                continue;
            }
            list($name) = $this->parseMiddleware($middleware);
            $instance = $this->app->make($name);
            if (method_exists($instance, 'terminate')) {
                $instance->terminate($request, $response);
            }
        }
    }
// 终止注册到服务容器的回调
public function terminate()
    {
        foreach ($this->terminatingCallbacks as $terminating) {
            $this->call($terminating);
        }
    }
```
至此，我们终于走完一次请求的完整生命周期。最后，我们试着做一下总结：
① 首先，所有的请求都会被`web`服务器导向单入口文件`public/index.php`；
② 第一步注册`composer`自动加载，以完成项目中所需依赖的加载；
③ 第二步在创建服务容器实例时，不仅包括项目基础服务、项目服务提供者别名、目录路径等在内的一系列注册工作，还会绑定 `HTTP` 内核及 `Console` 内核到 `APP `服务容器，绑定异常处理到服务容器；
④ 第三步通过服务容器去解析内核实例， 将`HTTP` 内核定义的中间件组注册到路由器，注册完后就可以在实际处理 `HTTP` 请求前调用这些中间件实现过滤请求的目的；
⑤ 第四步将`HTTP`请求实例注册到服务容器，并且通过启动引导程序来加载环境变量、全局配置、异常处理、注册门面、注册服务提供者、启动服务等，随后请求被分发到匹配的路由，在路由中执行中间件以过滤不满足校验规则的请求，只有通过中间件处理的请求才最终处理实际的控制器或匿名函数生成响应结果；
⑥ 第五步获取到响应结果后发送给请求的客户端；
⑦ 第六步运行程序` $kernel->terminate() `做一些善后清理工作，并最终退出脚本。

# 参考资料
[深度挖掘 Laravel 生命周期](https://learnku.com/articles/10421/depth-mining-of-laravel-life-cycle#11bfd9)
[底层原理 —— 一次 Laravel 请求的生命周期](https://laravelacademy.org/post/8597.html)
