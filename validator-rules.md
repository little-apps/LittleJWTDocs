---
title: Validator Rules
description: 
published: true
date: 2022-03-01T06:32:29.769Z
tags: laravel, request, rules, validatable, validation, validator
editor: markdown
dateCreated: 2022-03-01T05:37:05.853Z
---

Little JWT includes [custom validation rules](https://laravel.com/docs/8.x/validation#custom-validation-rules) for validating tokens passed as form inputs. This allows the token to be validated in [the controller](https://laravel.com/docs/8.x/controllers), rather than somewhere else like [the middleware](https://laravel.com/docs/8.x/middleware). 

# The ValidToken Rule

A ``ValidToken`` rule instance allows for tokens to be validated similiar to how tokens are validated using [the ``validateToken`` method](/validating#the-validatetoken-method).

Create a ``ValidToken`` instance by passing a [callback function](/validating#callback-parameter) (and maybe a [``boolean`` parameter](/validating#default-validatable-parameter) indicating if the default validatable should be used) to the constructor:

```php
use LittleApps\LittleJWT\Laravel\Rules\ValidToken;
use LittleApps\LittleJWT\Validation\Validator;

$callback = function (Validator $validator) {
    /* ... */
};

$applyDefault = false;

$rule = new ValidToken($callback, $applyDefault);
```

A callback array referencing a [validatable](/validatables) can also be passed as the first parameter:

```php
use LittleApps\LittleJWT\Laravel\Rules\ValidToken;
use LittleApps\LittleJWT\Validation\Validator;

$validatable = new MyValidator();

$callback = [$validatable, 'validate'];

// $applyDefault defaults to true if not passed.
$rule = new ValidToken($callback);
```

Attach the validation rule instance to a validator along with any other Laravel validation rules:

```php
use Illuminate\Support\Facades\Validator;
use LittleApps\LittleJWT\Laravel\Rules\ValidToken;

$validator = Validator::make($request->all(), [
    'token' => [
        'required',
        new ValidToken($callback, $applyDefault)
    ]
]);
```

> See [Laravel's documentation on Validation](https://laravel.com/docs/8.x/validation) for more information on working with a Validator instance and different methods of validating inputs.

# Implicit Rule


Little JWT provides the implicit ``validtoken`` rule as an alternative to a ``ValidToken`` rule instance. It's important to note that it is an [implicit rule](https://laravel.com/docs/8.x/validation#implicit-rules), meaning it implies the token is required before it can be validated.

A comma seperated list can be included to specify which [validatables to use](/validatables):

```php
use Illuminate\Support\Facades\Validator;
use LittleApps\LittleJWT\Laravel\Rules\ValidToken;

$validator = Validator::make($request->all(), [
    'token' => 'validtoken:default,guard'
]);
```

If a comma seperated list is not included, the [default validatable is used](/validatables#default-validatable):

```php
use Illuminate\Support\Facades\Validator;
use LittleApps\LittleJWT\Laravel\Rules\ValidToken;

$validator = Validator::make($request->all(), [
    'token' => 'validtoken'
]);
```

> The [ValidateRuleTest.php file](https://github.com/little-apps/LittleJWT/blob/main/tests/Features/ValidateRuleTest.php) contains further examples on running the rules through Laravel's validator.