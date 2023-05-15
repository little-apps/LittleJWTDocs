---
title: JSON Web Keys (JWKs)
description: 
published: true
date: 2022-12-01T05:41:42.769Z
tags: jwk, json web key, default jwk, custom jwk, littlejwt instance
editor: markdown
dateCreated: 2022-02-05T06:59:37.761Z
---

Signing and verifying JSON Web Tokens (JWTs) is done using a JSON Web Key (JWK). Each ``LittleApps\LittleJWT\LittleJWT`` instance has a different JWK.

# Default JWK

## Key Types

JSON Web Keys can be built using different key types. The settings for building the default JWK are specified in the ``config/littlejwt.php`` file.

The following Artisan commands are available to simplify generating types of keys:

```bash
# Generates a secure phrase
php artisan littlejwt:phrase

# Generates a private key file
php artisan littlejwt:pem jwk.pem

# Generates a PKCS12 key file
php artisan littlejwt:p12 jwk.p12
```

> Use ``php artisan help ...`` to find out about arguments and options for the commands.

### OpenSSL

Some JWKs use PHP's built-in OpenSSL extension and you may need to specify the location of the ``openssl.cnf`` configuration file in order for it to work correctly. The location can be specified in the ``.env`` file:

```env
LITTLEJWT_OPENSSL_CNF=/usr/lib/ssl/openssl.cnf
```

Additional configuration parameters to use with ``openssl_*`` functions can also be set in the ``config/littlejwt.php`` file:

```php
return [
    /**
     * Configuration options to use with OpenSSL.
     * @see https://www.php.net/manual/en/function.openssl-csr-new.php The config parameter contains possible config options.
     */
    'openssl' => [
        'config' => env('LITTLEJWT_OPENSSL_CNF', '')
    ],
];
```

## Algorithms

Various signature algorithms are provided by the JWT framework. The available algorithms and the needed Composer package for each is available at https://web-token.spomky-labs.com/the-components/signed-tokens-jws/signature-algorithms.

## Utilizing

To build or validate a JWT with the default JWK, use either the Facade or the Application container.

### Facade:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$token = LittleJWT::createToken(/* ... */);
$passes = LittleJWT::validateJWT(/* ... */);
```

### Application Container:

```php
use Illuminate\Support\Facades\App;
use LittleApps\LittleJWT\LittleJWT;

// Using the app() function:

$token = app('littlejwt')->createToken(/* ... */);
$passes = app(LittleJWT::class)->validateJWT(/* ... */);

// Using the App facade:
$token = App::make('littlejwt')->createToken(/* ... */);
$passes = App::make(LittleJWT::class)->validateJWT(/* ... */);
```

# Custom JWK

To build or validate a JWT using a different key, first you must create a JWK instance. Little JWT is built off of the PHP JWT Framework, which has [documentation on creating JWK objects](https://web-token.spomky-labs.com/the-components/key-jwk-and-key-set-jwkset/key-management).

```php
use Jose\Component\KeyManagement\JWKFactory;

$jwk = JWKFactory::createFromSecret(
    'changethis',          // The shared secret
    [                      // Optional additional members
        'alg' => 'HS256',
        'use' => 'sig'
    ]
);
```

Then, create a new ``LittleJWT`` instance for building and validating JWTs with that JWK:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$littleJwt = LittleJWT::withJwk($jwk);

$token = $littleJwt->createToken(/* ... */);
$passes = $littleJwt->validateJWT(/* ... */);
```