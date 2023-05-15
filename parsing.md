---
title: Parsing
description: 
published: true
date: 2023-05-15T05:58:49.551Z
tags: jwt, parsetoken, parsing, token
editor: markdown
dateCreated: 2022-02-05T07:00:43.011Z
---

A token can be parsed from a string and transformed into a JsonWebToken instance. If the token cannot be parsed, the method will return ``null``. A parsed JWT does not mean that is a valid JWT. More information the JWT instance can be found in [The JWT](/the-jwt) documentation.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$token = "ey...";

$jwt = LittleJWT::parse($token);

if (! is_null($jwt)) {
    // The JWT was parsed.
    // This does NOT mean the JWT is valid.
} else {
    // The JWT wasn't parsed.
}
```