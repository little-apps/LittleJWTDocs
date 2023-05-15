---
title: Creating
description: 
published: true
date: 2023-05-15T05:42:22.004Z
tags: anonymous function, buildable, callback, claims, create, createjwt, createtoken, creating, generating, jwt, token
editor: markdown
dateCreated: 2022-02-05T06:58:00.402Z
---

# Creating a JWT

If you want to create a JWT instance, call the ``create`` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

$jwt = LittleJWT::create(function (Builder $builder) {
    // ...
});
```

 > More information on the JWT instance can be found under [The JWT](https://github.com/little-apps/LittleJWT/wiki/The-JWT) page.
 
## Creating a Token

If you want to create a token, simply cast the JWT instance to a string:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

$jwt = LittleJWT::create(function (Builder $builder) {
    // ...
});

$token = (string) $jwt;
// $token = "ey...";
```

# Parameters

## Callback

The first parameter accepted by either method is a [callback function](https://www.php.net/callable). The callback receives a ``LittleApps\LittleJWT\Build\Builder`` instance, which is used to specify what claims to include in the JWT. Further documentation on the ``Builder`` instance is [located here](https://github.com/little-apps/LittleJWT/wiki/The-Builder).

The following demonstrates passing an anonymous function to create a token:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

$jwt = LittleJWT::create(function (Builder $builder) {
    $builder
        ->abc('def', true)
        ->ghi('klm')
        ->nop('qrs', false);
});
```

 > See [Buildables](/buildables) for more information on utilizing buildables as an alternative to anonymous functions.

## Default Buildable

The second parameter indicates if the default buildable should also be used to create the token.  If the second parameter is not specified, it defaults to ``true``. Claims specified in the callback parameter will be added after and possibly override the default claims.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

// Applies the default builder
$jwt = LittleJWT::create(function (Builder $builder) {
    /* ... */
}, true);

// Also applies the default builder
$jwt = LittleJWT::create(function (Builder $builder) {
    /* ... */
});

// Ignores the default builder
$jwt = LittleJWT::create(function (Builder $builder) {
    /* ... */
}, false);
```

 > See [Buildables](/buildables) for more information on claims added by the default buildable and changing the default buildable.


