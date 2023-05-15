---
title: The JWT
description: 
published: true
date: 2022-08-09T06:34:46.699Z
tags: claims, header, jwt, payload, token, signature, base64
editor: markdown
dateCreated: 2022-02-05T07:02:43.450Z
---

An instance of the [``LittleApps\LittleJWT\JWT\JWT`` class](https://github.com/little-apps/LittleJWT/blob/main/src/JWT/JWT.php) provides immutable access to the header claims, payload claims, and signature.

# Create Token

A token can be created by casting the JWT to a string:

```php
$token = (string) $jwt;
// $token = 'ey...';
```

# Header & Payload Claims

The header and payload claims can be accessed using the appropriate method:

```php
$headers = $jwt->getHeaders();

$payload = $jwt->getPayload();
```

## Claim Manager

Both the ``getHeaders()`` and ``getPayload()`` methods return an instance of [the ``LittleApps\LittleJWT\JWT\ClaimManager`` class](https://github.com/little-apps/LittleJWT/blob/main/src/JWT/ClaimManager.php), which is an immutable collection of the claims. 

```php
$claims = $jwt->getPayload(); // or $jwt->getHeaders();

// Check if claim key 'abc' exists:
$exists = $claims->has('abc');
$exists = isset($claims->abc);
$exists = isset($claims['abc']);

// Get the claim value for key 'abc'
$value = $claims->get('abc'); // Returns null if key 'abc' doesn't exist.
$value = $claims->get('abc', false); // You can pass the default value that will be returned if the key doesn't exist.
$value = $claims->abc;
$value = $claims['abc'];

// Get all of the claim keys and values as a Collection.
$all = $claims->get();

// Get the number of claims in the header or payload.
$count = $claims->count();
```

> All claim values are deserialized using the appropriate mutator before being added to the collection. See [Mutating Claims](/mutating-claims) for more information.


# Signature

The signature can be retrieved from the JWT instance as raw bytes (decoded from base64):

```php
$signature = $jwt->getSignature();
```

## Encoding

The signature can be encoded back to base64 using the provided ``LittleApps\LittleJWT\Utils\Base64Encoder`` method:

```php
use LittleApps\LittleJWT\Utils\Base64Encoder;

$signature = $jwt->getSignature();
$encoded = Base64Encoder::encode($signature);
```