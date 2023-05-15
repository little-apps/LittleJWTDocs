---
title: Parsing
description: 
published: true
date: 2022-02-21T05:44:50.088Z
tags: jwt, token, parsing, parsetoken
editor: markdown
dateCreated: 2022-02-05T07:00:43.011Z
---

A token can be parsed from a string and transformed into a ``LittleApps\LittleJWT\JWT\JWT`` instance. If the token cannot be parsed, ``parseToken()`` will return ``null``. This means the token could not be parsed, not that it couldn't be validated. More information the JWT instance can be found in [The JWT](/the-jwt) documentation.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$token = "...";

$jwt = LittleJWT::parseToken($token);

if (! is_null($jwt)) {
    // The JWT was parsed.
    // This does NOT mean the JWT is valid.
} else {
    // The JWT wasn't parsed.
}
```