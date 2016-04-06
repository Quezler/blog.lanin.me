---
id: 100
title: Laravel + Hashids
date: 2015-10-05T12:15:53+00:00
author: Maxim Lanin
layout: post
permalink: /laravel-hashids/
dsq_thread_id:
  - 4270631551
categories:
  - Laravel
  - PHP
tags:
  - extending
  - hashids
  - laravel
  - php
  - router
format: standard
---
Everyone nowadays used youtube or any short link creator. If you look to the links they use (eg. `https://youtu.be/qatmJtIJAPw`) you will notice that they use unique string hashes instead of common auto-increment ids. They do it because anyone can just iterate ids and get access to all their content. And of course they don't want it.

If you want to protect your content too, you have to replace your ids with hashes. There are lots of tools that can convert integers to unique hash ids. They work practically identically, and I will show how to integrate one of them [Hashids](http://hashids.org/php/) with Laravel.

There are lots of packages that already integrate Hashids into Laravel, but they only add it's facade and give some syntactic sugar. But we need complex integration with easy model binding and all logic under the hood! So let's do the magic!

<!-- more -->

## Installation

First of all you have to install package `lanin/laravel-hashids` via composer.

```bash
$ composer require lanin/laravel-hashids
```

Then register a ServiceProvider that will extend default Router and provide you with all needed functionality.

```php
<?php

Lanin\Laravel\Hashids\HashidsServiceProvider::class,
```

Last thing add `\Lanin\Laravel\Hashids\UseHashidsRouter` trait to your `App\Http\Kernel`. This will force Laravel to use package router to dispatch the route.

```php
<?php

namespace App\Http;

class Kernel extends HttpKernel
{
    use \Lanin\Laravel\Hashids\UseHashidsRouter;

    ...
}
```

## Usage

### Binding

After the installation, Router's method `model` that binds your placeholders ids to the models will be updated to automatically support hashids and convert them to the internal integer ids.

Thats it! Binding part is done.

### Routing

If you will pass hash id to the url it will be easily resolved into an associated model. But how replace ids in your html output? There are two ways. Everything depends on how you form your urls.

If you prefer form them by hands, package gives you the blade helper method `@hashids($id)` that will convert your id to the hash string.

```html
<a href="/posts/@hashids($post->id)">{{$post->title}}</a>
```

But this method is tedious. And I prefer using awesome Laravel feature that automatically extracts ids from models and inserts them into your urls. Example:

```php
<?php

route('page.show', $page);
url('page', ['id' => $page]);
```

This methods were updated to handle Hashids too. They will automatically replace integer ids from your models to Hashids.

> If for some reason you don't want to convert them, just implement `\Lanin\Laravel\Hashids\DoNotUseHashids` interface in your model.

## Facade

This package is also provides Hashids Facade. It can proxy all Hashids API.

To enable it, add facade in your `config/app.php`:

```php
<?php

'Hashids' => Lanin\Laravel\Hashids\HashidsFacade::class,
```

## Configuration

By default package will use your `APP_KEY` as a salt and 4 symbols length for the hash. If you want to overwrite it, publish hashids configs and edit `config/hahshids.php`

```bash
$ php artisan vendor:publish
```

## Contributing

Please feel free to fork this package and contribute by submitting a pull request to enhance the functionalities.

<a class="github-button" href="https://github.com/mlanin/laravel-hashids/fork" data-icon="octicon-repo-forked" data-count-href="/mlanin/laravel-hashids/network" data-count-api="/repos/mlanin/laravel-hashids#forks_count" data-count-aria-label="# forks on GitHub" aria-label="Fork mlanin/laravel-hashids on GitHub">Fork</a> <a class="github-button" href="https://github.com/mlanin/laravel-hashids" data-icon="octicon-star" data-count-href="/mlanin/laravel-hashids/stargazers" data-count-api="/repos/mlanin/laravel-hashids#stargazers_count" data-count-aria-label="# stargazers on GitHub" aria-label="Star mlanin/laravel-hashids on GitHub">Star</a>
