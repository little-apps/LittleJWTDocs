---
title: Guard
description: 
published: true
date: 2023-05-15T06:14:37.191Z
tags: adapters, auth.php, authentication, config, configuration, custom adapter, fingerprint, generic, guard, laravel, user provider
editor: markdown
dateCreated: 2022-02-05T06:58:51.801Z
---

Little JWT provides a [guard for Laravel](https://laravel.com/docs/8.x/authentication#introduction) that authenticates users using a provided JWT.

# Getting Started

The first step to enabling the Little JWT guard is adding the configuration options to the ``config/auth.php`` file:

```php
return [
    'guards' => [
        'jwt' => [
            'driver' => 'littlejwt',
            'adapter' => 'fingerprint',
            'provider' => 'users',
            'input_key' => 'token'
        ],
    ],
];
```

You can set ``'jwt'`` as the default guard in the ``config/auth.php`` file:

```php
return [
    'defaults' => [
        'guard' => 'jwt',
    ],
];
```

# Authentication

To authenticate a user from a JWT, assign ``'auth'`` as the middleware for a route:

```php
Route::get('/user', function () {
    //
})->middleware('auth');
```

You can also specify ``'jwt'`` as the guard to specifically use for authentication (if there's another default guard set):

```php
Route::get('/user', function () {
    //
})->middleware('auth:jwt');
```

Authentication is done using the adapter set in the ``config/auth.php`` file (see 'Adapters' below on how to change the adapter and available adapters).


After the validation passes, the user is retrieved (also by the adapter) and can be retrieved using the ``Auth`` facade:

```php
use Illuminate\Support\Facades\Auth;

$user = Auth::user();
```

# Adapters

An adapter is attached to the LittleJWT guard to specify how the JWT is validated. It can also specify how the JWT is built.

Set the adapter to be used by the guard in the ``config/auth.php`` file:

```php
return [
    'guards' => [
        'jwt' => [
            'adapter' => 'fingerprint',
        ],
    ],
];
```

# Available Adapters

The following describes each adapter in more detail, including the validation steps.  Little JWT comes with the following adapters:

 * Generic Adapter (``'generic'``)
 * Fingerprint Adapter (``'fingerprint'``)

## Generic Adapter

The generic adapter is intended for building a JWT for a user and validating it. The generic adapter uses the [Guard Validatable](/validatables#guard-validatable) on top of the [Default Validatable](/validatables#default-validatable).

Set the adapter to ``generic`` in the ``config/auth.php`` file:

```php
return [
    'guards' => [
        'jwt' => [
            'adapter' => 'generic',
        ],
    ],
];
```

### Configuration

The configuration options for the generic adapter are located in the ``config/littlejwt.php`` file. There's currently no configuration options for the generic adapter.

```php
return [
    'guard' => [
        'adapters' => [
            'generic' => [
                /**
                 * The class for the adapter.
                 * This should not be changed.
                 */
                'adapter' => \LittleApps\LittleJWT\Guards\Adapters\GenericAdapter::class,
            ],
        ],
    ],
];
```

### Building

A JWT can be be built for a user using the generic adapter from the ``Auth`` facade:

```php
use Illuminate\Support\Facades\Auth;

$jwt = Auth::buildJwtForUser($user);

$token = (string) $jwt;
```

Additional header and payload claims can also be included:

```php
use Illuminate\Support\Facades\Auth;

$payload = [
    'abc' => 'def'
];

$header = [
    'abc' => 'def'
];

$jwt = Auth::buildJwtForUser($user, $payload, $header);

$token = (string) $jwt;
```

A ``Response`` instance can be built from the user:

```php
use Illuminate\Support\Facades\Auth;

$response = Auth::createJwtResponse($user);
```

The response will be a JSON object as shown on the [Request & Response Helpers page](/request-response-helpers#json-object).


## Fingerprint Adapter

The fingerprint adapter is meant for building and validating JWTs that are secured against [token sidejacking](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html#token-sidejacking). After the [default](/validatables#default-validatable) and [guard](/validatables#guard-validatable) validatables are used, the [fingerprint validatable](/validatables#fingerprint-validatable) is used.

Set the adapter to ``fingerprint`` in the ``config/auth.php`` file:

```php
return [
    'guards' => [
        'jwt' => [
            'adapter' => 'fingerprint',
        ],
    ],
];
```

### Configuration

The configuration options for the fingerprint adapter are located in the ``config/littlejwt.php`` file:

```php
return [
    'guard' => [
        'adapters' => [
            'fingerprint' => [
                /**
                 * The class for the adapter.
                 * This should not be changed.
                 */
                'adapter' => \LittleApps\LittleJWT\Guards\Adapters\FingerprintAdapter::class,
                
                /**
                 * Name of the cookie to hold the fingerprint.
                 */
                'cookie' => 'fingerprint',
                
                /**
                 * How long the fingerprint cookie should live for (in minutes).
                 * If 0, the cookie has no expiry.
                 */
                'ttl' => 0,
            ],
        ],
    ],
];
```

### Building

#### Automatically

A fingerprint is a unique UUID that's stored as a cookie and passed along with a JWT (which contains a hash of the fingerprint).

You can generate a JWT with a random fingerprint and have the cookie automatically included in the response:

```php
use Illuminate\Support\Facades\Auth;

$response = Auth::createJwtResponse($user);
```

The response will be a JSON object (the same shown in the [Request & Response Helpers page](/request-response-helpers#json-object)) and include a cookie containing the fingerprint.

#### Manually

You can manually send the JWT and/or cookie yourself:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Response;
use Illuminate\Support\Str;

// Generate the fingerprint UUID
$fingerprint = (string) Str::uuid();

// Hash the fingerprint UUID
$hash = Auth::hashFingerprint($fingerprint);

// Generate the JWT with the fingerprint hash
$jwt = Auth::createJwtWithFingerprint($user, $hash);

// Respond with information about the JWT and fingerprint cookie
return 
    Response::withJwt($jwt)
        ->withCookie(Auth::getFingerprintCookieName(), $fingerprint, Auth::getFingerprintCookieTtl());
```

### Validation Steps

The following is a list of steps done by this adapter to validate the JWT:

 1. Uses the generic adapter to perform the key validations first.
 2. Securely compares the fingerprint hash in the 'fgpt' claim to the hash of the fingerprint cookie included with the request.

# Custom Adapters

A custom guard adapter can be created to incorporate your own validation checks.

## 1. Create Adapter Class

Define a class that provides the validation functionality for the guard using one of the following options:

### Option A: Extend ``AbstractAdapter``

The ``AbstractAdapter`` class provides the needed functionality to parse a token, validate a JWT, and get the associated User model (using the ``sub`` payload claim).

Create a class that extends ``AbstractAdapter`` and in the ``getValidatorCallback`` method, return a callback that sets the [Validator rules](/the-validator#rules):

```php
use LittleApps\LittleJWT\Guards\Adapters\AbstractAdapter;
use LittleApps\LittleJWT\Validation\Validator;

class MyAdapter extends AbstractAdapter {
    /**
     * Gets a callback that receives a Validator to specify the JWT validations.
     *
     * @return callable
     */
    protected function getValidatorCallback()
    {
        return function(Validator $validator) {
            // ...
        };
    }
}
```

### Option B: Implement ``GuardAdapter``

The ``GuardAdapter`` interface allows you to have more control over how the token is parsed, validated, and the user is retrieved.

Create a class that implements the ``GuardAdapter`` interface and implement the required methods:

```php
use LittleApps\LittleJWT\Contracts\GuardAdapter;
use LittleApps\LittleJWT\LittleJWT;
use LittleApps\LittleJWT\JWT\JsonWebToken;

class MyAdapter implements GuardAdapter {
    /**
     * The instance for building and validating JWTs
     *
     * @var LittleJWT
     */
    protected $jwt;

    /**
     * The options to use for the adapter.
     *
     * @var array
     */
    protected $config;

		/**
     * Create a new adapter instance.
     *
     * @return void
     */
    public function __construct(LittleJWT $jwt, array $config)
    {
        $this->jwt = $jwt;

        $this->config = $config;
    }

    /**
     * Parse a token from a string to a JWT.
     * This does NOT check if the JWT is valid.
     *
     * @param string $token
     * @return JsonWebToken JWT instance or null if unable to be parsed.
     */
    public function parse(string $token)
    {
        // ...
    }

    /**
     * Validate the JWT.
     *
     * @param JsonWebToken $jwt
     * @return bool True if JWT is validated.
     */
    public function validate(JsonWebToken $jwt)
    {
        // ...
    }

    /**
     * Gets a user from the JWT
     *
     * @param UserProvider $provider
     * @param JsonWebToken $jwt
     * @return Authenticatable
     */
    public function getUserFromJwt(UserProvider $provider, JsonWebToken $jwt)
    {
        // ...
    }
}
```

#### Notes

* If implementing the ``GuardAdapter`` interface, use [dependency injection](https://laravel.com/docs/8.x/container#method-invocation-and-injection) with the constructor to automatically include any needed dependencies for the adapter.
* If extending the ``AbstractAdapter`` class, a callback array referencing the ``validate`` method in a [Validatable instance](/validatables) can also be returned by the ``getValidatorCallback`` method.
* Any additional public methods specified in the guard adapter can be called using the ``Illuminate\Support\Facades\Auth`` facade.
* The [``BuildsJwt`` trait](https://github.com/little-apps/LittleJWT/blob/main/src/Guards/Adapters/Concerns/BuildsJwt.php) can be used if the guard adapter builds JWTs for users. It currently only works with the ``AbstractAdapter`` since it requires the ``$jwt`` property.
* Use the [``HasRequest`` trait](https://github.com/little-apps/LittleJWT/blob/main/src/Guards/Adapters/Concerns/HasRequest.php) if the guard adapter will need anything from the request.

## 2. Configuration

Add an entry to the the ``config/littlejwt.php`` file that has the class for the adapter and any other configuration options will be passed to the adapter class constructor when it's created:

```php
return [
    'guard' => [
        'adapters' => [
            'custom' => [
                /**
                 * The fully qualified class name for the adapter.
                 */
                'adapter' => \Namespace\For\MyAdapter::class,
                
                /**
                 * Add any additional configuration options below.
                 */
            ],
        ],
    ],
];
```

## 3. Change Adapter

Finally, set the guard adapter in the ``config/auth.php`` file to your custom adapter name:

```php
return [
    'guards' => [
        'jwt' => [
            'adapter' => 'custom',
        ],
    ],
];
```