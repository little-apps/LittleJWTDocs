---
title: Validating
description: 
published: true
date: 2023-05-16T04:02:27.261Z
tags: anonymous function, callback, checks, jwt, rules, token, validatable, validating
editor: markdown
dateCreated: 2022-02-05T07:04:17.541Z
---

Validating allows for a JWT to be sent through a series of checks (such as that it's not expired and the signature matches) before the JWT is used for other things. The ``LittleJWT`` [facade](https://laravel.com/docs/8.x/facades) provides methods to validate tokens. 

When validating a [JWT](/the-jwt) instance, use the ``validate`` method which returns an object with the JWT and the result of the validation:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

$result = LittleJWT::validate($jwt, function (Validator $validator) {
  // ...
});

$passes = $result->passes();
```

If it's a token, it will need to be transformed to a JWT instance first. You can transform it with the ``parse`` method first, and then send it to the ``validate`` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

$token = 'ey...';

$jwt = LittleJWT::parse($token);

if (! is_null($jwt)) {
  $result = LittleJWT::validate($jwt, function (Validator $validator) {
    // ...
  });
  
  if ($result->passes()) {
    // Token is valid.
  } else {
    // Token is invalid.
  }
} else {
  // Token could not be parsed.
}
```

Or, you can use the ``validateToken`` method to do the parsing and validating:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

$token = 'ey...';

// Unlike validate, the validateToken method returns a boolean.
$passes = LittleJWT::validateToken($token, function (Validator $validator) {
  // ...
});

if ($passes) {
  // Token was parsed and is valid.
} else {
  // Token wasn't parsed or is invalid.
}
```

# Callback Parameter

The second parameter accepted is a [callback function](https://www.php.net/callable). The callback receives a ``LittleApps\LittleJWT\Validation\Validator`` instance, which is used to specify what rules to check on the JWT or token. Further documentation on the validator as well as available rules is [located here](/the-validator).

The following demonstrates passing an anonymous function to validate a token:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

$result = LittleJWT::validate($jwt, function (Validator $validator) {
    $validator
        ->valid()                          // Checks the signature is valid.
        ->equals('abc', 'def', true, true) // Checks claim 'abc' in header equals 'def'
        ->equals('ghi', 'klm')             // Checks claim 'ghi' in payload equals 'klm'
        ->equals('nop', 'qrs');            // Checks claim 'nop' in header equals 'qrs'
});
```

 > See [Validatables](/validatables) for more information on utilizing validatables as an alternative to anonymous functions.

# Default Validatable

The third parameter is a ``boolean`` that indicates if the default validatable should also be used. If the second parameter is not specified, it defaults to ``true``. Validation checks specified in the callback parameter will be done after the default validatable checks.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

// Applies the default validatable
$result = LittleJWT::validate($jwt, function (Validator $validator) {
    $validator
        // All rules here are checked before the default validatable.
        ->contains(['foo'])
        ->equals('foo', 'bar');
}, true);

// Also applies the default validatable
$result = LittleJWT::validate($jwt, function (Validator $validator) {
    /* ... */
});

// Ignores the default validatable
$result = LittleJWT::validate($jwt, function (Validator $validator) {
    /* ... */
}, false);
```

 > See [Validatables](/validatables) for more information on checks done by the default validatable and changing the default validatable.
