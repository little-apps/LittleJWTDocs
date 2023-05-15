---
title: Building
description: 
published: true
date: 2023-05-15T05:56:49.861Z
tags: build, buildable, building, buildjwt, claims, jwt
editor: markdown
dateCreated: 2022-02-05T06:53:18.364Z
---

If you want more control over the JWT build process, use the the ``build`` method to get a `LittleApps\LittleJWT\Build\Build` instance:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$build = LittleJWT::build();
```

The `Build` instance does not have any buildable callbacks when it is first created. Buildable callbacks can be added to specify claims for the JWT:

```php
use LittleApps\LittleJWT\Build\Builder;

// ...

$build->passBuilderThru(function (Builder $builder) {
  // Method overloading
  $build->abc('def');

  // Property overloading
  $build->abc = 'def';

  // Defined methods
  $build->addClaim('abc', 'def');
});
```

Use the `build()` method create a JWT. This does **not** sign the JWT.

```php
// ...

$unsigned = $build->build();
```

Use the ``sign`` method to sign the JWT:

```php
// ...

$unsigned = $build->build();
$signed = $unsigned->sign();
```