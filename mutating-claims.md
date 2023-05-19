---
title: Mutating
description: 
published: true
date: 2023-05-19T06:11:03.800Z
tags: claims, dates, encryption, mutating, mutators, numbers, objects, parsing
editor: markdown
dateCreated: 2022-02-05T07:00:11.445Z
---

Claim values can be mutated to an appropriate data type. The data type to mutate to depends on what is set for the claim key. If no mutation for the claim key is set, the claim value is whatever it was decoded to by ``LittleApps\LittleJWT\Utils\JsonEncoder`` (could be a string, array, number, etc.). 

# Specifying Mutators

Mutating is split into serializing and unserializing. Serializing is performed before a JWT is built, while unserializing is performed after a token is parsed.

Before the JWT is serialized or unserialized, the mutators must be specified.

Use the ``mutate`` method to specify the mutators and they will be applied to any building, validating, etc. done after:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
});
```

The following demonstrates serializing, building, and signing a JWT:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;
use LittleApps\LittleJWT\Build\Builder;
use LittleApps\LittleJWT\Validation\Validator;

$serialized = LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
})->create(function (Builder $builder) {
  $builder
    ->foo(1684215857);
});

// $serialized has payload claim 'foo' with value '2023-05-16'
```

The following demonstrates validating and unserializing a JWT:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;
use LittleApps\LittleJWT\Validation\Validator;

// Validating:
$result = LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
})->validate($serialized, function (Validator $validator) {
  $validator
    ->valid();
});

$unserialized = $result->unserialized();

// $unserialized has payload claim 'foo' with value a Carbon instance (representing '2023-05-16')
```

If you want to serialize a JWT (without signing it) or unserialize a JWT (without validating it), you can call the ``serialize`` or ``unserialize`` methods instead:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

// Serializing:
$serialized = LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
})->serialize($jwt);

// Unserializing:
$unserialized = LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
})->unserialize($serialized);
```

## Additional Callbacks
 
Each ``mutate`` method call creates a new ``MutateHandler`` instances, meaning existing mutations will not be stored with the main LittleJWT instance:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
});

// The mutators set above won't be applied when this JWT is created:
$jwt = LittleJWT::create();
```

That is, unless you use the ``passMutatorsThru`` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

LittleJWT::passMutatorsThru(function (Mutators $mutators) {
  $mutators
    ->foo('date');
});

// The mutators set above will be applied when this JWT is created:
$jwt = LittleJWT::create();
```

You can also use this method to apply mutators to an existing ``MutateHandler`` instance:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

$handler = LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
});

$handler->passMutatorsThru(function (Mutators $mutators) {
  $mutators
    ->bar('datetime');
});

// The mutators set above will be applied when this JWT is created:
$jwt = LittleJWT::create();
```


Additional calls to ``mutate`` can be made after the first ``mutate`` method call. The mutations will be merged, with the later mutations taking precedence:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->foo('date');
})->mutate(function (Mutators $mutators) {
  $mutators
    ->foo('datetime') // Overrides previously set mutator
    ->bar('date');
});

// The 'foo' claim mutator is 'datetime'.
// The 'bar' claim mutator is 'date'.
```

# Available Mutators

A list of possible primitive mutators is below:

 * ``array``
 * ``bool``
 * ``custom_datetime``
 * ``date``
 * ``datetime``
 * ``decimal``
 * ``encrypted``
 * ``double``
 * ``float``
 * ``real``
 * ``int``
 * ``json``
 * ``object``
 * ``timestamp``
 * ``model``

## Dates

Claim values can be mutated as a date using the ``custom_datetime``, ``date``, ``datetime``, and ``timestamp`` mutators.

Each mutator is different in the format it uses in the JWT:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    /**
     * Serializes the date/time in the specified format.
     * See https://www.php.net/manual/en/datetime.format.php for possible formats.
     */
    ->a('custom_datetime:Y-m-d')
    /**
     * Serializes the date/time in the 'Y-m-d' format.
     */
    ->b('date')
    /**
     * Serializes the date/time in the ISO8601 format.
     */
    ->c('datetime')
    /**
     * Serializes the date/time as Unix timestamp (in seconds).
     */
    ->d('timestamp');
});
```

## Numbers

Claim values can be mutated to various number formats.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

LittleJWT::mutate(function (Mutators $mutators) {
  $mutators
    ->a('double') // Can be 'double', 'float', or 'real'.
    ->b('decimal:2'); // Serializes the claim value with 2 decimals.
});
```

The ``double``, ``float``, and ``real`` mutators perform the same. If the value is positive or negative infinite, it is serialized as ``'Infinity'`` or ``'-Infinity'`` (respectively). If the value is not a number (NaN), it is serialized as ``'NaN'``. Otherwise, the number is serialized by casting it to a string.

The ``decimal`` mutator uses PHP's built-in [``number_format``](https://www.php.net/number_format) function to serialize the claim value as a string with a specified number of decimals. If no number of decimals is specified, no decimals are included.

## Encrypting & Decrypting

Claims can be encrypted when serialized and decrypted when deserialized using the ``encrypted`` mutator. The claim value will be encrypted and decrypted using [Laravel's built-in encryption service](https://laravel.com/docs/8.x/encryption).

## Objects

The ``json`` and ``object`` mutators use PHP's built-in [``json_encode``](https://www.php.net/manual/en/function.json-encode.php) function to serialize the claim value. The difference between them is when deserialized, the ``json`` mutator mutates it to an associative array, while the ``object`` mutator mutates it to an ``stdClass`` instance.

## Model

The ``model`` mutator will transform a ``Model`` instance into the primary key value (when serializing) and the primary key value back to the ``Model`` instance (when unserializing):

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;
use App\Models\User;

LittleJWT::mutate(function (Mutators $mutators) {
  $mutators->sub(sprintf('model:%s', User::class)); // The Model class needs to be specified.
});
```

## Stacking

Multiple mutators can be stacked for a single claim using the ``StackMutator``:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;
use LittleApps\LittleJWT\Mutate\Mutators;
use LittleApps\LittleJWT\Mutate\Mutators\StackMutator;
use LittleApps\LittleJWT\Mutate\Mutators\DoubleMutator;
use LittleApps\LittleJWT\Mutate\Mutators\EncryptMutator;

$stack =
  (new StackMutator())
    // Only \LittleApps\LittleJWT\Contracts\Mutator instances can be passed. 
    ->mutator($mutator1)
    ->mutator($mutator2);

$serialized = LittleJWT::mutate(function (Mutators $mutators) use ($stack) {
  $mutators->foo($stack);
})->create(function (Builder $builder) {
  $builder->foo('abc');
});
```

When JWTs are unserialized, the stack is reversed so the input is changed to the output and the output is changed to the input in the correct order:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Build\Builder;
use LittleApps\LittleJWT\Mutate\Mutators;
use LittleApps\LittleJWT\Mutate\Mutators\StackMutator;
use LittleApps\LittleJWT\Mutate\Mutators\DoubleMutator;
use LittleApps\LittleJWT\Mutate\Mutators\EncryptMutator;

$stack =
  (new StackMutator())
    // Adding the EncryptMutator first will cause an error.
    ->mutator(new DoubleMutator)
    ->mutator(new EncryptMutator);
    
$handler = LittleJWT::mutate(function (Mutators $mutators) use ($stack) {
  $mutators->foo($stack);
});

$serialized = $handler->create(function (Builder $builder) {
  $builder->foo(1234.1234);
});

// $serialized->getPayload()->get('foo') == 'ey...'

$unserialized = $handler->validate($serialized)->unserialized();

// $unserialized->getPayload()->get('foo') == 1234.1234
```

# Default Mutators

Default mutators are automatically applied when on top of any other mutators. The default mutators are specified in the ``config/littlejwt.php`` file:

```php
return [
  /* ... */
  'builder' => [
    /**
     * Mutators to use for claims in the header and payload.
     */
    'mutators' => [
      'header' => [],
      'payload' => [
        'iat' => 'timestamp',
        'nbf' => 'timestamp',
        'exp' => 'timestamp'
      ]
    ],
  ]
  /* ... */
];
```

The default mutators can be enabled/disabled by calling the ``applyDefaultMutators`` method. You may want to do this in [the boot method of a service provider](https://laravel.com/docs/10.x/providers#the-boot-method), as the mutators will be enabled/disabled for any subsequent serializing/unserializing. If not set, the default mutators are enabled.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

// Enables default mutators:
LittleJWT::applyDefaultMutators(true);

// Disables default mutators:
LittleJWT::applyDefaultMutators(false);
```

# Enabling and Disabling Mutations

Mutations can be enabled or disabled completely. If disabled, the mutation functionality will not work. You may also want to do this in [the boot method of a service provider](https://laravel.com/docs/10.x/providers#the-boot-method).

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

// Enables mutating:
LittleJWT::alwaysMutate(true);

LittleJWT::mutate(function (Mutators $mutators) { }); // Works!

// Disables mutating:
LittleJWT::alwaysMutate(false);

LittleJWT::mutate(function (Mutators $mutators) { }); // Doesn't work!
```

The ``withMutate`` and ``withoutMutate`` methods can also be used to enable or disable mutations when building, validating, etc.:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

// With mutate:
LittleJWT::withMutate()->mutate(function (Mutators $mutators) { }); // Works!

// Without mutate:
LittleJWT::withoutMutate()->mutate(function (Mutators $mutators) { }); // Doesn't work!
```

# Custom Mutators

To create a custom mutator, first create a class that implements the ``LittleApps\LittleJWT\Contracts\Mutator`` interface:

```php
<?php

namespace App\Components\LittleJWT;

use LittleApps\LittleJWT\Contracts\Mutator;
use LittleApps\LittleJWT\JWT\JsonWebToken;

class MyMutator implements Mutator
{
    /**
     * @inheritDoc
     */
    public function serialize($value, string $key, array $args, JsonWebToken $jwt)
    {
        /* ... */
    }

    /**
     * @inheritDoc
     */
    public function unserialize($value, string $key, array $args, JsonWebToken $jwt)
    {
        /* ... */
    }
}
```

The ``serialize`` and ``unserialize`` methods both take in the existing serialized (for ``unserialize``) or unserialized (for ``serialize``) claim value, claim key, any specified arguments, and the original JWT. They return the serialized or unserialized value. 

The following example serializes the claim value as an upper-case string and unserializes it back as a lower-case string:

```php
<?php

namespace App\Components\LittleJWT;

use LittleApps\LittleJWT\Contracts\Mutator;
use LittleApps\LittleJWT\JWT\JsonWebToken;

class MyMutator implements Mutator
{
    /**
     * @inheritDoc
     */
    public function serialize($value, string $key, array $args, JsonWebToken $jwt)
    {
        return strtoupper($value);
    }

    /**
     * @inheritDoc
     */
    public function unserialize($value, string $key, array $args, JsonWebToken $jwt)
    {
        return strtolower($value);
    }
}
```

 > If no value is returned by the ``serialize`` or ``unserialize`` methods, the return value is assumed to be ``null``.
 
## Applying Custom Mutators

### Mutator Instance

Custom mutators can be applied, much like the other mutators, by applying the ``Mutator`` instance to the ``Mutators`` instance:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;
    
LittleJWT::mutate(function (Mutators $mutators) {
  $mutators->foo(new MyMutator);
});
```

### Custom Mutator Mapping

A custom string key can also be used to map to the custom mutator.

First, it will need to be mapped in the boot method of a service provider:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use LittleApps\LittleJWT\Facades\LittleJWT;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        LittleJWT::customMutator('example', MyMutator::class);

        // You may need to bind the custom mutator with the app container:
        $this->app->bind(MyMutator::class, function ($app) {
          return new MyMutator();
        });
    }
}
```

The string key can then be used instead of the ``Mutator`` instance:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;
    
LittleJWT::mutate(function (Mutators $mutators) {
  $mutators->foo('example');
});
```

### Resolve Method

Alternatively, a resolve method can be injected into the ``MutatorResolver`` class to allow a mutator to be mapped to a custom key:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Mutate\Mutators;

// Probably add this to the boot method of a service provider:
MutatorResolver::macro('resolveCustom', function () {
  return new MyMutator();
});

// Later on in your code:
LittleJWT::mutate(function (Mutators $mutators) {
  $mutators->foo('custom');
});
```

## Order

The mutator instances are resolved in the following order. If a mutator is found, none of the remaining checks are done.

 1. Primitive mappings (see above)
 2. Resolve method
 3. Custom mutator mapping