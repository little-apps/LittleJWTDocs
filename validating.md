---
title: Validating
description: 
published: true
date: 2022-03-02T08:03:09.043Z
tags: anonymous function, callback, checks, jwt, rules, token, validating, validatable
editor: markdown
dateCreated: 2022-02-05T07:04:17.541Z
---

Validating allows for a JWT to be sent through a series of checks (such as that it's not expired and the signature matches) before the JWT is used for other things. The ``LittleJWT`` [facade](https://laravel.com/docs/8.x/facades) provides methods to validate tokens. The method to call depends on if it's a token or JWT being validated.

# The validateToken Method

When validating a [token](/#token-vs-jwt), use the ``validateToken`` method. The token will be [parsed as a JWT](/parsing) before being validated.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$passes = LittleJWT::validateToken($token, /* ... */);
```

# The validateJWT Method

When validating a [JWT](/the-jwt), use the ``validateJWT`` method.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$passes = LittleJWT::validateJWT($jwt, /* ... */);
```

# Parameters

## 1. Token or JWT

The first parameter accepts either a token or JWT depending the method being called (as shown above).

## 2. Callback Parameter

The second parameter accepted by either method is a [callback function](https://www.php.net/callable). The callback receives a ``LittleApps\LittleJWT\Validation\Validator`` instance, which is used to specify what rules to check on the JWT or token. Further documentation on the validator as well as available rules is [located here](/the-validator).

The following demonstrates passing an anonymous function to validate a token:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

$passes = LittleJWT::validateToken($token, function (Validator $validator) {
    $validator
        ->valid()                          // Checks the signature is valid.
        ->equals('abc', 'def', true, true) // Checks claim 'abc' in header equals 'def'
        ->equals('ghi', 'klm')             // Checks claim 'ghi' in payload equals 'klm'
        ->equals('nop', 'qrs');            // Checks claim 'nop' in header equals 'qrs'
});
```

 > See [Validatables](/validatables) for more information on utilizing validatables as an alternative to anonymous functions.

## 3. Default Validatable

The third parameter is a ``boolean`` that indicates if the default validatable should also be used. If the second parameter is not specified, it defaults to ``true``. Validation checks specified in the callback parameter will be done after the default validatable checks.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

// Applies the default validatable
$passes = LittleJWT::validateToken($token, function (Validator $validator) {
    $validator
        // All rules here are checked before the default validatable.
        ->contains(['foo'])
        ->equals('foo', 'bar');
}, true);

// Also applies the default validatable
$passes = LittleJWT::validateToken($token, function (Validator $validator) {
    /* ... */
});

// Ignores the default validatable
$passes = LittleJWT::validateToken($token, function (Validator $validator) {
    /* ... */
}, false);
```

 > See [Validatables](/validatables) for more information on checks done by the default validatable and changing the default validatable.

# Return Value

Both methods return a boolean, with ``true`` meaning all the validation checks passed and ``false``  meaning one or more of the checks failed.
