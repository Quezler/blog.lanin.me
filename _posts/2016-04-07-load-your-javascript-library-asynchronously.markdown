---
layout: post
title: "Load Your JavaScript Library Asynchronously"
date: 2016-04-07T17:06:44+03:00
author: Maxim Lanin
permalink: /load-your-javascript-library-asynchronously/
categories:
  - JavaScript
  - HTML5
tags:
  - tutorial
  - javascript
  - library
  - html
  - html5
  - async
format: standard
---
Lets imagine, we have a common page:

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="app.js"></script>
    <script>
      App.init('greeting', 'Hello, John Doe!');
    </script>
  </head>
  <body>
    <h1 id="greeting"></h1>
  </body>
</html>
```

With javascript library included:

```javascript
// app.js
var App = {
  /**
   * Handle already registered actions.
   *
   * @param  {String} id   Element ID
   * @param  {String} name User's name
   */
  init: function (id, name) {
    document.getElementById(id).text(name);
  }
}
```
<!-- more -->

When your browser loads this page, first of all it will load and execute your included script, fire `init` method and then show you the hello message. As you can see, script loading blocks your page, but we want to show it as quickly as possible!

I think I didn't tell you anything new and you already know, that it is always better to place such script tags to the very bottom of the body tag.

But what if I tell you that today it is not the only option?

## Async way

HTML5 brings us a new `async` attribute for the script tag. If browser sees this attribute, it will try to load your js file in async way.

In our example we can do it like this:

```html
<script src="app.js" async="async"></script>
```
Now browser will show page and load our script at the same time. Even if we will add more scripts with async attribute, they will be loaded together asynchronously.

The main point in this paragraph is that page shows instantly. So if there is any additional scripts that uses your `app.js`, they may throw an error. Because they can be loaded before `app.js` itself. That's why `App.init()` method in the first example will throw an error that it cannot be found.

## Solution
But what if I tell you that there is a solution that allows us not only to load our script asynchronously but call any actions in it without worrying if our script is already loaded.

> Besides, I want to mention that practically all counters and analytic scripts nowadays are made using this approach.

Approach is very easy. We need to check if our app is already loaded, then use it. If not, save all reference history to handle it after script is finally received by browser.

First lets update our script block that inits the App:

```javascript
var App = App || [];
App.push(['init', 'greeting', 'Hello, John Doe!']);
```

First line here checks if there is any `App` already registered. If not, assign new empty array to it.
Then push new array, where #0 element in it is a method name, others are its arguments we want to pass.

Now we need to update `app.js` to handle array's push method and handle history.

```javascript
if (typeof App !== 'undefined') {
  var registeredActions = App
}

var App = {
  /**
   * Handle already registered actions.
   *
   * @param  {String} id   Element ID
   * @param  {String} text Text to show
   */
  init: function (id, text) {
    document.getElementById(id).innerHTML = text;
  },

  /**
   * Implement array's push method to handle push calls.
   *
   * @param  {Array|Function} item Method call. [method_name, args...]
   */
  push: function (item) {
    if (typeof item === 'function') {
      item()
    } else {
      var functionName = item.shift()
      App[functionName].apply(null, item)
    }
  },

  /**
   * Handle already registered actions.
   *
   * @param  {Array} array History of calls
   */
  _processPushes: function (array) {
    for (var i = 0; i < array.length; i++) {
      App.push(array[i])
    }
  }
}

if (typeof registeredActions !== 'undefined') {
  App._processPushes(registeredActions)
}
```

First we check if there was already `App` var registered. If it was, then saves them to `registeredActions` var.

Then there is `App` itself with 2 extra methods:

- `push` implements array's push calls;
- `_processPushes` handles calls history.

At last we check if there is saved history process it.

## Conclusion
You can successfully use this technique to speed up loading of your applications.

Also it helps you to provide non-blocking js scripts, to include on your customers sites (if you made one more counter, for example ;))

---

All examples can be found on [github.com](https://github.com/mlanin-tutorials/js-async).
