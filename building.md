---
title: Building
description: 
published: true
date: 2022-02-21T05:26:28.099Z
tags: buildable, claims, jwt, build, building, buildjwt
editor: markdown
dateCreated: 2022-02-05T06:53:18.364Z
---

The `buildJWT` method in the LittleJWT facade allows for a JWT to be built using an instance of the `LittleApps\LittleJWT\Build\Build` class:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$build = LittleJWT::buildJWT();
```

Claims can be added similar to how claims are added to the `LittleApps\LittleJWT\Build\Builder` instance. The `Build` instance does not have any claims automatically added to it.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$build = LittleJWT::buildJWT();

// Method overloading
$build->abc('def');

// Property overloading
$build->abc = 'def';

// Defined methods
$build->addClaim('abc', 'def');
```

Use the `build()` method create a JWT:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$build = LittleJWT::buildJWT();

// ...

$jwt = $build->build();
```

Use chaining to build a JWT in a single line of code:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$jwt =
    LittleJWT::buildJWT()
      ->abc('def', true)
      ->ghi('lmn')
        ->build();
```

A `LittleApps\LittleJWT\Contracts\Buildable` implementation can be applied to the `Build` instance using the `passBuilderThru()` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$buildable = /* ... */;

$jwt =
    LittleJWT::buildJWT()
      ->passBuilderThru($buildable)
        ->build();
```