---
title: Middleware
description: 
published: true
date: 2022-03-01T18:38:11.820Z
tags: middleware, validtoken, laravel, route, guard, http, request, response
editor: markdown
dateCreated: 2022-02-13T23:02:51.465Z
---

Little JWT comes with [middleware](https://laravel.com/docs/8.x/middleware), which allows a token to be validated before the HTTP request is processed any further.

The fully qualified class name can be used:

```php
Route::get('/protected', function () {
    //
})->middleware(\LittleApps\LittleJWT\Laravel\Middleware\ValidToken::class);
```

The ``'validtoken'`` alias can also be used:

```php
Route::get('/protected', function () {
    //
})->middleware('validtoken');
```

 >  The Valid Token middleware is not responsible for authentication (i.e.: retrieving the user associated with the token). See [the guard page](/guard) for more information on authentication.

## Parameters

The validatables to validate tokens with can be specified. Use any validatable keys (specified in the [configuration file](/validatables#configuration)) as the parameter:

```php
Route::get('/protected', function () {
    // Reaches here after token is validated by both the 'default' and 'guard' validatable.
})->middleware('validtoken:default,guard');
```

Not specifying any parameters will cause the token to be validated using the default validatable (set in [the configuration file](/validatables#changing-the-default-validatable)):

```php
Route::get('/protected', function () {
    // Reaches here after token is validated the the default validatable.
})->middleware('validtoken');
```