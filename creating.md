---
title: Creating
description: 
published: true
date: 2022-09-22T06:47:01.701Z
tags: claims, creating, token, jwt, callback, create, createjwt, createtoken, anonymous function, buildable, generating
editor: markdown
dateCreated: 2022-02-05T06:58:00.402Z
---

# Creating a Token

If you want to create a token, use the ``createToken`` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

$token = LittleJWT::createToken(function (Builder $builder) {
    // ...
});
```

# Creating a JWT

If you want to create a JWT instance, use the ``createJWT`` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

$jwt = LittleJWT::createJWT(function (Builder $builder) {
    // ...
});
```

 > More information on the JWT instance can be found under [The JWT](https://github.com/little-apps/LittleJWT/wiki/The-JWT) page.

# Parameters

## Callback

The first parameter accepted by either method is a [callback function](https://www.php.net/callable). The callback receives a ``LittleApps\LittleJWT\Build\Builder`` instance, which is used to specify what claims to include in the JWT. Further documentation on the ``Builder`` instance is [located here](https://github.com/little-apps/LittleJWT/wiki/The-Builder).

The following demonstrates passing an anonymous function to create a token:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

$token = LittleJWT::createToken(function (Builder $builder) {
    $builder
        ->abc('def', true)
        ->ghi('klm')
        ->nop('qrs', false);
});

// $token = "ey...";
```

 > See [Buildables](/buildables) for more information on utilizing buildables as an alternative to anonymous functions.

## Default Buildable

The second parameter indicates if the default buildable should also be used to create the token.  If the second parameter is not specified, it defaults to ``true``. Claims specified in the callback parameter will be added after and possibly override the default claims.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;

// Applies the default builder
$defaultToken = LittleJWT::createToken(function (Builder $builder) {
    /* ... */
}, true);

// Also applies the default builder
$defaultToken = LittleJWT::createToken(function (Builder $builder) {
    /* ... */
});

// Ignores the default builder
$customToken = LittleJWT::createToken(function (Builder $builder) {
    /* ... */
}, false);
```

 > See [Buildables](/buildables) for more information on claims added by the default buildable and changing the default buildable.


