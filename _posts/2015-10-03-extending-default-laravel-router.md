---
id: 103
title: Extending default Laravel 5 Router
date: 2015-10-03T22:13:01+00:00
author: Maxim Lanin
layout: post
permalink: /extending-default-laravel-router/
dsq_thread_id:
  - 4270771158
categories:
  - Laravel
  - PHP
tags:
  - extending
  - laravel
  - php
  - router
format: standard
---
While working with Laravel framework sometimes it becomes necessary to extend it's default Router. It Laravel 4 this process was pretty simple and was described in the [official manual](http://laravel.com/docs/4.2/extending#request-extension). But in Laravel 5 this process became more complicated and has some pitfalls.

<!-- more -->

## Router

First part is rather easy: we have to implement our router class and a Service. First of all lets create router:

```php
<?php namespace App\Services;

class Router extends \Illuminate\Routing\Router
{
  /**
   * Dummy method.
   *
   * @param $uri
   */
  public function hello($uri)
  {
    $this->get($uri, function()
    {
      return 'Hello!';
    });
  }
}
```

Here we created our custom router that extends the basic `\Illuminate\Routing\Router` and has a dummy method hello that just prints &#8216;Hello!' whatever route we want.

## Service Provider

Second we have to register our router to switch it with the default one. For this purpose we don't have to create new service provider, we already have already created `RouterServiceProvider`. So lets use it add create a register method.

```php
<?php
/**
 * Register the router instance.
 *
 * @return void
 */
protected function register()
{
  $this->app['router'] = $this->app->share(function($app)
  {
    return new \App\Services\Router($app['events'], $app);
  });
}
```

And this is it! Now you can use `Route::hello('/hello');` method in your `routes.php` file and it will work like a charm.

## Pitfalls

Everything looks good but if you want to modify route handling somehow, you will notice that this code is not enough.

Problem is that Laravel registers default router practically in the beginning of it's workflow and loads it to the Kernel before loading custom service providers. So to overwrite it we have to make a little hack.

First we need to overwrite `protected function dispatchToRouter()` from `\Illuminate\Foundation\Http\Kernel` in `\App\Http\Kernel` class.

This method fires when Laravel handles the route itself. But it does it after all service providers were loaded. So we can just update default Kernel router with our custom one.

```php
<?php
/**
 * Get the route dispatcher callback.
 *
 * @return \Closure
 */
protected function dispatchToRouter()
{
  $this->router = $this->app['router'];

  return parent::dispatchToRouter();
}
```

But still it is not enough. If you'll try to launch this code you will probably get an error linked to the middleware.

It happens because Kernel iterates route middlewares and saves them into the router right in it's constructor, so when we replace it with our custom one, Laravel misses all middleware info.

So let's update `dispatchToRouter` method and iterate middlewares manually.

```php
/**
 * Get the route dispatcher callback.
 *
 * @return \Closure
 */
protected function dispatchToRouter()
{
  $this->router = $this->app['router'];

  foreach ($this->routeMiddleware as $key => $middleware)
  {
    $this->router->middleware($key, $middleware);
  }

  return parent::dispatchToRouter();
}
```

And now everything will work perfectly.

> This method I successfully used while developing [lanin/laravel-hashids](https://github.com/mlanin/laravel-hashids) composer package, that integrates Hashids lib into Laravel. You can read about it [here](/2015/10/05/laravel-hashids/).
