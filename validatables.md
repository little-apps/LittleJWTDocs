---
title: Validatables
description: 
published: true
date: 2023-04-16T18:35:43.569Z
tags: config, configuration, guard, resolving, validatable, default validatable, custom validatable, fingerprint, stack, generic
editor: markdown
dateCreated: 2022-02-13T04:32:07.736Z
---

Rather than passing an anonymous function to either the ``validateToken()`` or ``validateJWT()`` methods, you could create an [invokable class](https://www.php.net/manual/en/language.oop5.magic.php#object.invoke) in order to reuse the same validations:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use Illuminate\Support\Facades\App;

// Create validatable instance directly:
$validatable = new MyValidator();

// Pass $validatable callback array to validateToken method.
$passes = LittleJWT::validateToken($token, $validatable);
```

# Configuration

There's advantages to including the validatable in the ``config/littlejwt.php`` configuration file:

1. Configuration options can be easily changed for the validatable.
2. The validatable key can be set as [the default validatable](#changing-the-default-validatable). 
3. It can be created using Laravel's container (see [resolving validatables](#resolving-validatables) below).
4. The validatable key can be specified as [a middleware parameter](/middleware#parameters).
5. The validatable key can be specified as [a parameter for the implicit validator rule](/validator-rules#implicit-rule).

The following is an example of configuration options for validatables:

```php
return [
    'validatables' => [
        'default' => [
            /**
             * Validatable instance to use for this validator.
             */
            'validatable' => \LittleApps\LittleJWT\Validation\Validators\DefaultValidator::class,
            
            /* ... */
        ],
        'guard' => [
            /**
             * Validatable instance to use for this validator.
             */
            'validatable' => \LittleApps\LittleJWT\Validation\Validators\GuardValidator::class,
            
            /* ... */
        ],
    ],
];
```

# Bundled Validatables

Little JWT comes bundled with a few pre-existing validatables. Each validatable has a key that can be used when specifying validatables for things like [the middleware](/middleware) or [validator rules](/validator-rules).

* [Default Validatable](#default-validatable)
* [Guard Validatable](#guard-validatable)

## Default Validatable

 * Key: ``default``

The default validatable is used when it is set as [the default validatable](/validating#default-validatable) and the third parameter (``$applyDefault``) is passed to the ``validateToken`` or ``validateJWT`` method as ``true``:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

// Applies the default validatable:
$passes = LittleJWT::validateToken($token, null, true);

// The second parameter is null and third parameter is true by default:
$passes = LittleJWT::validateToken($token);
```

### Rules

The default validatable checks that:

 * The [algorithm](/the-validator#algorithms) matches what is set in config file.
 * The header [contains the claims](/the-validator#contains) specified in the config file.
 * The payload [contains the claims](/the-validator#contains) specified in the config file.
 * The [signature is valid](/the-validator#valid) using either the default JWK or the JWT associated with the LittleJWT instance.
 * The JWT is [allowed (not blacklisted)](/the-validator#allowed).
 * The ``exp`` (expires) claim is in [the future](/the-validator#future).
 * The ``nbf`` (not before) claim is in [the past](/the-validator#past).
 * The ``iat`` (issued at) claim is in [the past](/the-validator#past).
 * The ``aud`` (audience) claim [equals](/the-validator#equals) what is set in config file.
 * The ``iss`` (issuer) claim [equals](/the-validator#equals) what is set in config file.

### Configuration

The following outlines the configuration options you can set for the ``DefaultValidatable`` in the ``config/littlejwt.php`` file under ``validatables.default``:

| Option | Type | Description |
|-|-|-|
| ``required`` | ``array`` | Indicates which claim keys must exist in the header or payload. This merely checks the claim keys, not if they're valid. |
| ``leeway`` | ``int`` | How many seconds after the JWT expires to still be valid. |
| ``alg`` | ``string``| The expected value for the algorithm ('alg') header claim. |
| ``iss`` | ``string`` | The expected value for the issuer ('iss') payload claim. |
| ``aud`` | ``string`` | The expected value for the audience ('aud') payload claim. |

## Stack Validatable

The stack validatable allows multiple validatables to be sent as a single validatable:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validatables\StackValidatable;

$validatable1 = new MyValidatable();
$validatable2 = new MyOtherValidatable();

$stack = new StackValidatable([$validatable1, $validatable2]);

$passes = LittleJWT::validateToken(
    $token,
    // Pass the invokable StackValidatable class.
    $stack,
    // The default validatable will still be used if the third parameter is true.
    false
);
```

You can also stack multiple callbacks as a single validatable:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;
use LittleApps\LittleJWT\Validation\Validatables\StackValidatable;

$callback1 = function(Validator $validator) { /* ... */ };
$callback2 = function(Validator $validator) { /* ... */ };

$stack = new StackValidatable([$callback1, $callback2]);

$passes = LittleJWT::validateToken(
    $token,
    // Pass in a callback array referencing the validate method in the StackValidatable.
    $stack,
    // The default validatable will still be used if the third parameter is true.
    false
);
```

## Guard Validatable

 * Key: ``guard``

The guard validatable is used when [the generic guard adapter](/guard#generic-adapter-generic) is used. Since it's meant for the generic guard adapter, it's not recommended that you instantiate the ``GuardValidatable`` class directly.

### Rules

The guard validatable checks that:

 * A user exists with the ``sub`` (subject) claim identifier (if the ``exists`` option is ``true``).
 * The ``prv`` (provider) claim [equals](/the-validator#equals) the model class set in the configuration file (if the ``model`` option is not ``false``).
 * The payload [contains the claims](/the-validator#contains):
   * ``sub`` (if the ``exists`` option is ``true``).
   * ``prv`` (if the ``model`` option is not ``false``).

### Configuration

The following outlines the configuration options you can set for the ``DefaultValidatable`` in the ``config/littlejwt.php`` file under ``validatables.guard``:

| Option | Type | Description |
|-|-|-|
| ``exists`` | ``boolean`` | If true, checks that a user exists with the ``sub`` (subject) claim identifier.  This is separate than the user provider retrieving the user to associate with the guard. |
| ``model`` | ``string`` ``boolean`` | The expected value for the provider ('prv') payload claim. If false, the 'prv' payload claim is not validated (not recommended). |

## Fingerprint Validatable

The fingerprint validatable is used when the guard adapter is set to the [fingerprint adapter](/guard#fingerprint-adapter). Since it's meant for the fingerprint guard adapter, it's not recommended that you instantiate the ``FingerprintValidatable`` class directly.

### Rules

The fingerprint validatable checks that:

 * The ``fgpt`` claim [securely equals](/the-validator#secure-equals) the hash of the fingerprint cookie.

### Configuration

There are no configuration options for the the fingerprint validatable.

# Custom Validatables

 > As of version 1.6, the ``Validatable`` interface is deprecated and will be removed in future versions. Instead, use [invokable classes](https://www.php.net/manual/en/language.oop5.magic.php#object.invoke).

Create a custom validatable by creating a class that implements the ``Validatable`` interface:

```php
use LittleApps\LittleJWT\Validation\Validator;
use LittleApps\LittleJWT\Contracts\Validatable;

class MyValidatable implements Validatable
{
    public function __construct()
    {
    }

    public function validate(Validator $validator)
    {
        /* ... */
    }
}
```

Send a callback array referencing the ``validate()`` method in the ``$validatable`` instance to either the ``validateToken()`` or ``validateJWT()`` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$validatable = new MyValidatable();

$passes = LittleJWT::validateToken($token, [$validatable, 'validate']);
```

## Configuration File
 
To take advantage of the configuration file (as explained above), add an entry to the ``config/littlejwt.php`` configuration file. You must set the value of the ``'validatable'`` key to the fully qualified class name.

```php
return [
    'validatables' => [
        /* ... */
        
        'custom' => [
            /**
             * Validatable instance to use for this validator.
             */
            'validatable' => \Path\To\MyValidatable::class,
            
            // Configuration options are passed to the MyValidator instance.
            'foo' => 'bar'
        ],
    ],
];
```

Make sure the  constructor accepts the configuration as a parameter and sets it to a property. Any keys set in the configuration file entry (besides 'validatable') will be passed when the instance is created by the Laravel container.

```php
use LittleApps\LittleJWT\Validation\Validator;

class MyValidatable
{
    protected $config;

    public function __construct(array $config)
    {
        $this->config = $config;
    }

    public function __invoke(Validator $validator)
    {
        /*
         * Use the config property to access configuration options.
         * 
         * Example:
         *   $foo = $this->config['foo'];
         */
    }
}
```

## Resolving Validatables

Now that the validatable is registered in the configuration file, it can be resolved using Laravel's container. The  alias is ``littlejwt.validatables.`` followed by the key entry specified in the configuration file:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use Illuminate\Support\Facades\App;

$validatable = App::make('littlejwt.validatables.custom');

$passes = LittleJWT::validateToken($token, $validatable);
```

It can also be used with the provided Valid Token middleware:

```php
Route::get('/protected', function () {
    // Reaches here after token is validated with the custom validatable.
})->middleware('validtoken:custom');
```

# Changing The Default Validatable

The default validatable can be changed in the ``config/littlejwt.php`` file.

```php
return [
    'defaults' => [
        /* ... */

        'validatable' => 'default'
    ]
];
```

The value for the ``defaults.validatable`` key corresponds with a key set under ``validatables.*``.

```php
return [
    'validatables' => [
        'default' => [
            /* ... */
        ]
    ]
];
```
