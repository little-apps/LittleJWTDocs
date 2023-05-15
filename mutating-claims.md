---
title: Mutating Claims
description: 
published: true
date: 2023-05-02T06:24:45.486Z
tags: claims, parsing, mutating, mutators, dates, numbers, encryption, objects
editor: markdown
dateCreated: 2022-02-05T07:00:11.445Z
---

Claim values can be mutated to an appropriate data type. The data type to mutate to depends on what is set for the claim key. If no mutation for the claim key is set, the claim value is whatever it was decoded to by ``LittleApps\LittleJWT\Utils\JsonEncoder`` (could be a string, array, number, etc.). 

# Setting Mutators

Mutators can be specified in one of the following ways:

## 1. Configuration File

In the ``config/littlejwt.php`` file, set the mutator to use for the claim key in either the 'header' or 'payload' array. The claims specified will be mutated when the JWT is both created and parsed.

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

## 2. Buildables

Add a public method called ``getMutators`` that returns the mutators to use when claims in the header or payload are serialized:

```php
use LittleApps\LittleJWT\Build\Builder;

class MyBuildable
{
    public function getMutators()
    {
        return [
            'header' => [
                'foo' => 'date',
            ],
            'payload' => [
                'baz' => 'float',
            ]
        ];
    }

    public function __invoke(Builder $builder)
    {
        $builder
            ->foo(time(), true)
            ->baz(NAN);
    }
}
```

When the JWT is created using the buildable, the claims specified will be mutated:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$buildable = new MyBuildable();

$jwt = LittleJWT::createJWT($buildable);

// $jwt->getHeaders()->foo = '2023-04-27';
// $jwt->getPayload()->baz = 'NaN';
```

 > If both the configuration and buildable have a mutator set for the same claim key, the mutator specified in the *buildable* will take precedence.
 
## 3. Parse Token

An array of mutators can be passed to the ``parseToken`` method.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$mutators = [
  'payload' => [
    'foo' => 'array',
  ],
];

//$token = 'ey...';

$jwt = LittleJWT::parseToken($token, $mutators);

// $jwt->getPayload()->foo = [];
```

 > If both the configuration and ``parseToken`` mutators have the same claim key, the mutator passed to ``parseToken`` will take precedence.
 
## 4. Validatables

Including mutators in a validatables can be useful when a token needs to be mutated before being validated. This only works with the ``validateToken`` method and not the ``validateJWT`` method (since it expects an already parsed token).

Add a public method called ``getMutators`` that returns the mutators to use when claims in the header or payload are parsed:

```php
use LittleApps\LittleJWT\Validation\Validator;

class MyValidatable
{
    public function getMutators()
    {
        return [
            'header' => [],
            'payload' => [
                'foo' => 'array',
            ]
        ];
    }

    public function __invoke(Validator $validator)
    {
        $validator
            // Strictly checks 'foo' claim is empty array.
            ->equals('foo', [], true);
    }
}
```

When the token is being validated, the mutated claims will be used:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

// $token = 'ey...';

$validatable = new MyValidatable();

$valid = LittleJWT::validateToken($token, $validatable);

// $valid = true;
```

 > Mutators can also be included in the [Default Validatable](https://docs.getlittlejwt.com/validatables) and will be used anytime the optional third argument for ``validateToken`` is ``true``.

# Available Mutators

A list of possible mutators is below:

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

## Dates

Claim values can be mutated as a date using the ``custom_datetime``, ``date``, ``datetime``, and ``timestamp`` mutators.

Each mutator is different in the format it uses in the JWT:

```php
return [
    'builder' => [
        /**
         * Mutators to use for claims in the header and payload.
         */
        'mutators' => [
            'header' => [],
            'payload' => [
                /**
                 * Serializes the date/time in the specified format.
                 * See https://www.php.net/manual/en/datetime.format.php for possible formats.
                 */
                'iat' => 'custom_datetime:Y-m-d',

                /**
                 * Serializes the date/time in the 'Y-m-d' format.
                 */
                'iat' => 'date',

                /**
                 * Serializes the date/time in the ISO8601 format.
                 */
                'iat' => 'datetime',

                /**
                 * Serializes the date/time as Unix timestamp.
                 */
                'iat' => 'timestamp',
            ]
        ],
    ],
];
```

## Numbers

Claim values can be mutated to various number formats.

```php
return [
    'builder' => [
        /**
         * Mutators to use for claims in the header and payload.
         */
        'mutators' => [
            'header' => [],
            'payload' => [
                /**
                 * Can be 'double', 'float', or 'real'.
                 */
                'a' => 'double',

                /**
                 * Serializes the claim value with 2 decimals.
                 */
                'b' => 'decimal:2',
            ]
        ],
    ],
];
```

The ``double``, ``float``, and ``real`` mutators perform the same. If the value is positive or negative infinite, it is serialized as ``'Infinity'`` or ``'-Infinity'`` (respectively). If the value is not a number (NaN), it is serialized as ``'NaN'``. Otherwise, the number is serialized by casting it to a string.

The ``decimal`` mutator uses PHP's built-in [``number_format``](https://www.php.net/number_format) function to serialize the claim value as a string with a specified number of decimals. If no number of decimals is specified, no decimals are included.

## Encrypting & Decrypting

Claims can be encrypted when serialized and decrypted when deserialized using the ``encrypted`` mutator. The claim value will be encrypted and decrypted using [Laravel's built-in encryption service](https://laravel.com/docs/8.x/encryption).

## Objects

The ``json`` and ``object`` mutators use PHP's built-in [``json_encode``](https://www.php.net/manual/en/function.json-encode.php) function to serialize the claim value. The difference between them is when deserialized, the ``json`` mutator mutates it to an associative array, while the ``object`` mutator mutates it to an ``stdClass`` instance.