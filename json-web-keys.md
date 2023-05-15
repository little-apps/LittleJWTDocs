---
title: JSON Web Keys (JWKs)
description: 
published: true
date: 2023-05-15T06:48:05.547Z
tags: custom jwk, default jwk, json web key, jwk, littlejwt instance
editor: markdown
dateCreated: 2022-02-05T06:59:37.761Z
---

Signing and verifying JSON Web Tokens (JWTs) is done using a JSON Web Key (JWK). Each ``LittleApps\LittleJWT\LittleJWT`` handler can have a different JWK.

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

$jwt = LittleJWT::create(/* ... */);
$passes = LittleJWT::validate(/* ... */);
```

### Application Container:

```php
use Illuminate\Support\Facades\App;
use LittleApps\LittleJWT\LittleJWT;

// Using the app() function:

$jwt = app('littlejwt')->create(/* ... */);
$passes = app(LittleJWT::class)->validate(/* ... */);

// Using the App facade:

$jwt = App::make('littlejwt')->create(/* ... */);
$passes = App::make(LittleJWT::class)->validate(/* ... */);
```

# Custom JWKs

To build or validate a JWT using a different key, first you must create a JsonWebKey instance. Little JWT is built off of the PHP JWT Framework (which has [documentation on creating JWK objects](https://web-token.spomky-labs.com/the-components/key-jwk-and-key-set-jwkset/key-management)), however, it's recommended to use the provided ``Keyable`` factory:

```php
use LittleApps\LittleJWT\Contracts\Keyable;

$keyable = App::make(Keyable::class);

// 1. Creates a random one-time JWK:

$jwk = $keyable->generateRandomJwk();

// 2. Create a JWK from a secret phrase:

$jwk = $keyable->buildFromSecret(['phrase' => 'littlejwtisgreat']);

// Passing an empty secret phrase is NOT recommended and generates a warning:
$jwk = $keyable->buildFromSecret(['phrase' => '']);

// Not passing a secret phrase will cause the MissingKeyException exception to be thrown:
$jwk = $keyable->buildFromSecret([]);

// 3. Create a JWK based on a certificate file:

$jwk = $keyable->buildFromFile(['type' => 'crt', 'path' => '/path/to/cert.crt']);

// 4. Create a JWK based on a PKCS #12 certificate file:

$jwk = $keyable->buildFromFile(['type' => 'p12', 'path' => '/path/to/cert.p12', 'secret' => 'abcd']);

// 5. Create a JWK based on a PEM key file:

$jwk = $keyable->buildFromFile(['type' => 'pem', 'path' => '/path/to/cert.pem', 'secret' => 'abcd']);

```

If you opt to use the PHP JWT Framework's factory, make sure the 'alg' (algorithm) is specified and it is transformed into a ``JsonWebKey`` instance:

```php
use Jose\Component\KeyManagement\JWKFactory;
use LittleApps\LittleJWT\JWK\JsonWebKey;

$baseJwk = JWKFactory::createFromSecret(
    'changethis',          // The shared secret
    [                      
        'alg' => 'HS256',  // Needs to be specified.
        'use' => 'sig'
    ]
);

$jwk = JsonWebKey::createFromBase($baseJwk);
```

Then, create a new ``LittleJWT`` instance for building and validating JWTs with that JWK:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$handler = LittleJWT::withJwk($jwk);

$token = $handler->create(/* ... */);
$passes = $handler->validate(/* ... */);
```