---
id: 76
title: Form Request validation errors
date: 2015-07-16T12:25:36+00:00
author: Maxim Lanin
layout: post
permalink: /form-request-validation-errors/    
dsq_thread_id:
  - 4292898129
categories:
  - Laravel
  - PHP
tags:
  - FormRequest
  - laravel
  - php
  - request
  - validation
format: standard
---
For the last half a year since Laravel 5.0 was released I saw lots of questions on how to handle validation errors using new feature Form Requests. So I decided to cover all options.

<!-- more -->

The most common problem is that Form Requests validation processed internally by Laravel Router itself. It doesn't throw an exception that can be handled in your application via your default Exception Handler.

So first let’s cover how Form Validation works. And for this purpose lets create a dummy route, controller and a Form Request.

```php
<?php

Route::post('posts', 'PostsController@store');

class PostsController extends Controller {

    /**
     * Save new post.
     * @param StoreRequest $request
     */
    public function store(StoreRequest $request) {
        // store post
    }

}

class StoreRequest extends Request {

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required',
            'body'  => 'required',
        ];
    }
}
```

When router receives command to handle a `POST /posts` route it tries to invoke controller's `store(StoreRequest $request)` method, and asks Container to resolve and inject our `StoreRequest` defined in it. After it will be resolved it automatically fires authorize method and then validates the request using riles from `rules()` method.

If validation fails, by default FormRequest throws `HttpResponseException` with a prepared `RedirectResponse` that contains all validation errors. And here lies the **problem**: this exception will be caught and processed right in the Router, it will never pass to your application level.

After Router catches this `HttpResponseException`, it fetches it's `RedirectResponse` and forces it right in the output. Because of that exception can't be caught in your Exceptions Handler. You can't log this exceptions, change messages or anything else.

By default Form Request redirects you to the same page but luckily you can overwrite this behaviour. In the default `FormRequest` class you can find three protected properties:

  * `$redirect` - the URI to redirect to.
  * `$redirectRoute` - the route to redirect to.
  * `$redirectAction` - the controller action to redirect to.

Using one of this properties you can define the url to redirect to.

If you want to handle exceptions throwing by your own, you can overwrite `failedValidation(Validator $validator)` method.

```php
<?php
    /**
     * Handle a failed validation attempt.
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return mixed
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException($this->response(
            $this->formatErrors($validator)
        ));
    }
```

In it you can do whatever you want. Throw your own exception to catch it in your Handler, Log your errors, prepare your own response, etc.

If you expect the same behaviour for all your requests, you can do it right in your abstract Request class. Or you can overwrite this method right in particular Request.

If you want to overwrite `authorize` errors behaviour, you can do it in the same way, but using `protected function failedAuthorization()` method.
