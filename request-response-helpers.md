---
title: Request & Response Helpers
description: 
published: true
date: 2022-09-22T06:29:47.679Z
tags: http, request, response, json, array, authorization header, helpers
editor: markdown
dateCreated: 2022-02-05T07:01:24.779Z
---

The following describes macros that allow JSON Web Tokens (JWTs) to be pulled from HTTP requests and pushed to HTTP responses.

# Request

```php
use Illuminate\Http\Request;
use Illuminate\Routing\Router as Route;

Route::get('/example', function (Request $request) {
    // Get the token
    $token = $request->getToken();

    // Get the JWT
    $jwt = $request->getJwt();
});
```

# Response

## JSON

Build a ``Illuminate\Http\JsonResponse`` that contains information about the JWT:

```php
use Illuminate\Support\Facades\Response;

$response = Response::withJwt($jwt);
```

When ``$response`` is returned, the client will receive a JSON object with information about the JWT:

```json
{
    "access_token": "ey...",
    "token_type": "bearer",
    "expires_in": 3600,
    "expires_at": "2022-02-04T03:17:07+0000"
}
```

 > The 'expires_at' date/time value is in ISO 8601 format.

## Array

Use the ``ResponseBuilder`` to create the array before it's encoded in JSON:

```php
use LittleApps\LittleJWT\Utils\ResponseBuilder;

$data = ResponseBuilder::buildFromJwt($jwt);
```

The method will return an array with information about the JWT:

```php
$data = [
    "access_token" => "ey...",
    "token_type" => "bearer",
    "expires_in" => 3600,
    "expires_at" => "2022-02-04T03:17:07+0000"
];
```

## Authorization Header

Set the JWT or token as the 'Authorization' response header:

```php
use Illuminate\Support\Facades\Response;

return response(['foo' => 'bar'])->attachJwt($jwt);
```

The client will receive a response with the 'Authorization' header set to ``'Bearer ey...'``

> Information on generating a response for the LittleJWT guard can be found the [Guard](/guard) page.
