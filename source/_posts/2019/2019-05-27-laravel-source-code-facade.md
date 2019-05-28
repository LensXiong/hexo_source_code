---
layout: post
title: ﻿【Laravel源码分析】Facade 的工作原理
date: 2019-05-27 08:09:24.000000000 +09:00
categories:
- 技术
tags:
- Laravel
toc: true
---

**
摘要：从开始接触`Laravel`框架的时候，就被框架的一些思想吸引，不管`Laravel`框架是优雅还是过度设计，个人都只是想从中学习其优秀的思想，恰好赶上公司项目用`Laravel`开发，也趁着这个机会深入对源码进行分析，以便从中能够吸收到一些优秀的思想。本文带着实际项目开发中的一些疑问，刨根问底去追溯问题的答案，一步一步分析门面的执行原理和别名加载的流程，也希望通过自己不断的积累，能够站在更高的层面去认识`Laravel`框架。
![https://www.valuecoders.com/blog/wp-content/uploads/2018/05/laravel.jpg](https://www.valuecoders.com/blog/wp-content/uploads/2018/05/laravel.jpg)
**
<!-- more -->
<The rest of contents | 余下全文>

>当我们写出 `Route::get ()` 这样的语句时，到底发生了什么，为什么我们可以这么写？

本文以`Route` 为例，详细分析了门面运行的原理，其他的门面运行实质上也是相同的道理。
`Route` 门面类文件位置如下：
```
vendor/laravel/framework/src/Illuminate/Support/Facades/Route.php
```
首先，我们来看一下 `Route` 的门面类：
```
class Route extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor()
    {
        return 'router';
    }
}
```
>  注：`Facade` 基类文件位置为：
```
vendor/laravel/framework/src/Illuminate/Support/Facades/Facade.php
```
`Route` 门面类继承门面基类 `Facade`, 但是在基类 `Facade`中并没有`get`方法，但我们注意到一个方法`__callStatic`：
```
 public static function __callStatic($method, $args)
     {
         $instance = static::getFacadeRoot();

         if (! $instance) {
             throw new RuntimeException('A facade root has not been set.');
         }

         return $instance->$method(...$args);
     }
```

`__callStatic`的解释:当对一个类使用一个未定义的静态方法的时候，会自动调用类中预先定义的`__callStatic()`这个静态魔术方法。

静态魔术方法`__callStatic`做了两件事：
① 调用`getFacadeRoot()`获取门面实例对象，本实例中为`router`对象实例。
② 通过获得的门面实例对象调用相应的方法，本实例中调用`get()`方法。
接下来，我们看看是如何获得对象实例的：
```
    public static function getFacadeRoot()
 {
     return static::resolveFacadeInstance(static::getFacadeAccessor());
 }

 protected static function getFacadeAccessor()
    {
        throw new RuntimeException('Facade does not implement getFacadeAccessor method.');
    }

 protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        return static::$resolvedInstance[$name] = static::$app[$name];
    }

    protected static $app;

    protected static function getFacadeAccessor()
    {
        return 'router';
    }
```

`getFacadeRoot()`函数调用了`getFacadeAccessor()`函数，而又因为基类中的`getFacadeAccessor()`函数被重写，返回`router`，`resolveFacadeInstance()`函数中利用 `$app` ，也就是服务容器创建了 `router`，创建成功后放入 `$resolvedInstance` 作为缓存，以便后期快速加载。

总结：
所有的门面类，本例中为`Route` 门面类，都需要通过重写`getFacadeAccessor()`方法，获取组件的注册名称，最终调用`__callStatic()`静态方法
时会通过服务容器或缓存获得一个相应的对象实例，最后通过获得的该对象实例调用不同的方法。

写到这里，你有没有一个疑问，为什么我们可以直接在` laravel `中全局用使用 `Route`，而不需要使用命名空间`use Illuminate\Support\Facades\Route`？
> 实质上` laravel `在启动的过程中，一开始便加载`config/app.php`中的`aliases`数组，最后通过`AliasLoader.php`中`load`函数调用`class_alias`将别名映射到真正的门面类中去。

```
 /**
     * Load a class alias if it is registered.
     *
     * @param string $alias
     * @return bool|null
     */
    public function load($alias)
    {
        if (static::$facadeNamespace && strpos($alias, static::$facadeNamespace) === 0) {
            $this->loadFacade($alias);
            return true;
        }
        if (isset($this->aliases[$alias])) {
            return class_alias($this->aliases[$alias], $alias);
        }
    }
```
接下来，我们从` laravel `的`index.php`一步一步分析整个加载和执行的流程：

```
require __DIR__.'/../bootstrap/autoload.php';
$app = require_once __DIR__.'/../bootstrap/app.php';
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
...
```
第一步，注册`composer`自动加载，使我们可以非常方便的使用`composer`加载所需的类。第二步，实例化了`Application`；第三步，构造`HttpKernel`；第四步，注册加载相关服务，处理请求和响应。其中，门面类的别名加载和注册也是在第四步完成的，我们接着来具体分析是如何进行加载并解析别名的真正类名。
在开始之前我们先来看一下传递给`handle()`函数的参数 `$request`是如何获取的：
```
$request = Illuminate\Http\Request::capture()
```
> ` laravel `通过全局 `$_SERVER` 数组构造一个 `Http` 请求（具体就是调用 `Symfony` 框架的 `Request` 类的 `createFromGlobals` 方法，来创建一个 `SymfonyRequest `对象）。

```
public static function capture()
    {
        static::enableHttpMethodParameterOverride();
        return static::createFromBase(SymfonyRequest::createFromGlobals());
    }
public static function createFromGlobals()
    {
        $server = $_SERVER;
        if ('cli-server' === PHP_SAPI) {
            if (array_key_exists('HTTP_CONTENT_LENGTH', $_SERVER)) {
                $server['CONTENT_LENGTH'] = $_SERVER['HTTP_CONTENT_LENGTH'];
            }
            if (array_key_exists('HTTP_CONTENT_TYPE', $_SERVER)) {
                $server['CONTENT_TYPE'] = $_SERVER['HTTP_CONTENT_TYPE'];
            }
        }
        $request = self::createRequestFromFactory($_GET, $_POST, array(), $_COOKIE, $_FILES, $server);
        if (0 === strpos($request->headers->get('CONTENT_TYPE'), 'application/x-www-form-urlencoded')
            && in_array(strtoupper($request->server->get('REQUEST_METHOD', 'GET')), array('PUT', 'DELETE', 'PATCH'))
        ) {
            parse_str($request->getContent(), $data);
            $request->request = new ParameterBag($data);
        }
        return $request;
    }
public static function createFromBase(SymfonyRequest $request)
    {
        if ($request instanceof static) {
            return $request;
        }
        $content = $request->content;
        $request = (new static)->duplicate(
            $request->query->all(), $request->request->all(), $request->attributes->all(),
            $request->cookies->all(), $request->files->all(), $request->server->all()
        );
        $request->content = $content;
        $request->request = $request->getInputSource();
        return $request;
    }
```
`handle()` 方法位于 `vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php`中，是整个`HTTP`处理的核心内容。该方法内容如下：
```
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
```
第一个`enableHttpMethodParameterOverride()`方法使得在`POST`请求中可以添加`_method`参数来伪造`HTTP`方法（如`post`中添加`_method=DELETE`来构造`HTTP DELETE`请求）。
第二个`sendRequestThroughRouter()`方法是通过中间件或路由过滤获取的`$request`请求。具体内容如下：
```
 protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);
        Facade::clearResolvedInstance('request');
        $this->bootstrap();
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
```
第一步在 `laravel` 的 `Ioc` 容器设置 `$request `请求的对象实例，第二步是在`Facade` 中清除 `request` 的缓存实例。
第三步是通过`HTTP`内核引导相关的组件，其中包含门面的注册以及配置文件的加载，门面的别名也就是从这里载入的。
```
public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }
protected function bootstrappers()
    {
        return $this->bootstrappers;
    }
protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class, // 加载配置文件
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class, // 注册门面
        ...
    ];
```
接下来调用 `Ioc` 容器的`bootstrapWith() `函数来创建这些组件并利用组件进行启动服务。
```
 public function bootstrapWith(array $bootstrappers)
    {
        $this->hasBeenBootstrapped = true;
        foreach ($bootstrappers as $bootstrapper) {
            $this['events']->fire('bootstrapping: '.$bootstrapper, [$this]);
            $this->make($bootstrapper)->bootstrap($this);
            $this['events']->fire('bootstrapped: '.$bootstrapper, [$this]);
        }
    }
```
`RegisterFacades` 的 `bootstrap()` 函数：
``` 
class RegisterFacades
{
    public function bootstrap(Application $app)
    {
        Facade::clearResolvedInstances();
        Facade::setFacadeApplication($app);
        AliasLoader::getInstance(array_merge(
            $app->make('config')->get('app.aliases', []),
            $app->make(PackageManifest::class)->aliases()
        ))->register();
    }
}
```
第一步，清除了 `Facade` 中的缓存；第二步，设置 `Facade` 的 `Ioc` 容器；第三步，获得我们`app/confog`文件中`aliases `别名映射数组；第四步：使用 `aliases` 实例化初始化 `AliasLoader`；第五步，调用 `AliasLoader->register ()`。
```
public function register()
        {
            if (! $this->registered) {
                $this->prependToLoaderStack();
                $this->registered = true;
            }
        }
protected function prependToLoaderStack()
     {
       spl_autoload_register([$this, 'load'], true, true);
     }
public function load($alias)
    {
        if (static::$facadeNamespace && strpos($alias, static::$facadeNamespace) === 0) {
            $this->loadFacade($alias);
            return true;
        }
        if (isset($this->aliases[$alias])) {
            return class_alias($this->aliases[$alias], $alias);
        }
    }
```
分析到最后，实质上是利用`class_alias()`函数将别名映射到真正的门面类中。

以上就是门面的基本原理和门面别名服务的启动流程，其中更详细的内容需要更进一步对源码进行剖析。




