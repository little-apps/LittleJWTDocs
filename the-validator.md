---
title: The Validator
description: 
published: true
date: 2022-08-21T08:18:44.764Z
tags: rules, validation, checks, validator, validate, claims, logic
editor: markdown
dateCreated: 2022-02-05T07:03:17.390Z
---

The ``LittleApps\LittleJWT\Validation\Validator`` instance provides methods to specify checks to be performed on a JWT or token. This is the received by the callback sent to ``validateToken`` and ``validateJWT`` methods (as documented in the [Validating page](/validating))

# Rules

The following is a list of available rules:

 * [Algorithms](#algorithms)
 * [Allowed](#allowed)
 * [Callback](#callback)
 * [Claim Callback](#claim-callback)
 * [Contains](#contains)
 * [Equals](#equals)
 * [Future](#future)
 * [One Of](#one-of)
 * [Past](#past)
 * [Secure Equals](#secure-equals)
 * [Valid](#valid)

## Algorithms

Checks that the 'alg' claim is one of the specified algorithms.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$algorithms`` | ``array`` | Algorithm keys to check for (HS256, RS256, etc.) |
| ``$inHeader`` | ``bool`` | Checks the 'alg' claim in the header (if ``true``) or in the payload (if ``false``). (Default: ``true``) |

### Example

```php
// Checks the 'alg' claim in the header is HS256.
$validator->algorithms(['HS256']);

// Checks the 'alg' claim in the payload is HS256.
$validator->algorithms(['HS256'], false);
```

## Allowed

Checks that the JWT is not blacklisted.

 > The [Blacklist Manager page](/blacklist-manager) has more information on configuring the blacklist.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$driver`` | ``string`` ``null`` | The blacklist driver to use. If ``null``, the blacklist driver specified in the config file is used. (Default: ``null``) |
| ``$before`` | ``bool`` | If ``true``, this check is performed others. If ``false``, this check is ran before or after other checks. (Default: ``true``) |

### Example

```php
// Checks the JWT is not blacklisted and then checks that the 'abc' claim equals 'def'.
$validator
    ->allowed() 
    ->equals('abc', 'def');

// Checks the JWT is not in the database blacklist either before or after checking the 'abc' claim equals 'def'.
$validator
    ->allowed('database', false) 
    ->equals('abc', 'def');
```

## Contains

Checks that claims with keys exist in header or payload.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$keys`` | ``array`` | The claim keys to check if they exist in the header or payload. |
| ``$strict`` | ``bool`` | If ``true``, the JWT can **only** have the specified keys in the header or payload. (Default: ``false``) |
| ``$inHeader`` | ``bool`` | If ``true``, checks the claim keys exist in the header. If ``false``, checks the payload. (default: ``false``) |
| ``$before`` | ``bool`` | If ``true``, this check is performed others. If ``false``, this check is ran before or after other checks. (Default: ``true``) |

### Example

```php
// Checks the JWT payload has the keys 'abc' and 'xyz'.
$validator->contains(['abc', 'xyz']);

// Checks the JWT payload ONLY has the keys 'abc' and 'xyz'.
$validator->contains(['abc', 'xyz'], true);
```

## Callback

Allows a custom check to be done using a custom callback that receives a JWT instance and returns a boolean value. See [The JWT](https://github.com/little-apps/LittleJWT/wiki/The-JWT) for more information on the JWT.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$callback`` | ``callable`` | Callback that receives the JWT as a parameter and returns ``true`` or ``false``. |

### Example

```php
use LittleApps\LittleJWT\JWT\JWT;

// Checks the JWT has at least 1 payload claim.
$validator->(function(JWT $jwt) {
    return $jwt->getPayload()->count() >= 1;
});
```

## Claim Callback

Allows a custom claim check to be done using a custom callback that receives the claim value and returns a boolean value. The claim value is deserialized before being sent to the callback.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$key`` | ``string`` | Claim key |
| ``$callback`` | ``callable`` |  Callback that receives the claim value, key, and JWT instance. |
| ``$inHeader`` | ``bool`` | If true, checks claim in header. (default: ``false``) |

### Example

```php
// Checks the 'abc' payload claim value is all uppercase
$validator->claimCallback('abc', function($value) {
    return strtoupper($value) === $value;
});
```

## Equals

Checks value of claim with key equals expected.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$key`` | ``string`` | Claim key |
| ``$expected`` | ``mixed`` | Expected value. |
| ``$strict`` | ``bool`` | If true, performs type comparison. (default: ``true``) |
| ``$inHeader`` | ``bool`` | If true, checks claim in header. (default: ``false``) |

### Example

```php
// Checks the claim with key 'abc' equals 'def'
$validator->equals('abc', 'def');

// Checks the claim with key 'abc' equals 1, '1', or true.
$validator->equals('abc', 1, false);
```

## Future

Checks that the claim date/time is in the future.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$key`` | ``string`` | Claim key |
| ``$leeway`` | ``int`` | Leeway (in seconds) to allow after claims set date/time. (default: 0) |
| ``$inHeader`` | ``bool`` | If true, checks claim in header. (default: ``false``) |

### Example

```php
// Checks the date/time for the 'exp' payload claim is in the future.
$validator->future('exp');

// Checks the date/time for the 'exp' payload claim is in the future +/- 3600 seconds.
$validator->future('exp', 3600);
```

## One Of

Checks value of claim is one of the expected values

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$key`` | ``string`` | Claim key |
| ``$haystack`` | ``array`` | Expected values |
| ``$strict`` | ``bool`` | If true, performs type comparison. (default: ``true``) |
| ``$inHeader`` | ``bool`` | If true, checks claim in header. (default: ``false``) |

### Example

```php
// Checks the 'abc' payload claim is 'cba' or 'def'
$validator->oneOf('abc', ['cba', 'def']);

// Checks the 'abc' payload claim is 1, '1', true, or 'cba'
$validator->oneOf('abc', [1, 'cba'], false);
```

## Past

Checks that the claim date/time is in the past.

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$key`` | ``string`` | Claim key |
| ``$leeway`` | ``int`` | Leeway (in seconds) to allow before claims set date/time. (default: 0) |
| ``$inHeader`` | ``bool`` | If true, checks claim in header. (default: ``false``) |

### Example

```php
// Checks the date/time for the 'iat' payload claim is in the past.
$validator->past('iat');

// Checks the date/time for the 'iat' payload claim is in the past +/- 3600 seconds.
$validator->past('exp', 3600);
```

## Secure Equals

Securely checks value of claim with key equals expected. The comparison uses the [``hash_equals`` function](https://www.php.net/hash_equals) to prevent timing attacks .

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$key`` | ``string`` | Claim key |
| ``$expected`` | ``mixed`` | Expected value. |
| ``$inHeader`` | ``bool`` | If true, checks claim in header. (default: ``false``) |


### Example

```php
// Securely checks the 'abc' payload claim equals 'def'.
$validator->secureEquals('abc', 'def');
```

## Valid

Checks if the JWT has a valid signature using the default or a custom JSON Web Key (JWK). For more information on setting JWKs, see [JSON Web Keys (JWKs)](/json-web-keys).

### Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$jwk`` | ``Jose\Component\Core\JWK`` ``null`` | JWK instance to use. If null, the default JWK is used. (Default: ``null``) |
| ``$before`` | ``bool`` | If ``true``, this check is performed others. If ``false``, this check is ran before or after other checks. (Default: ``true``) |

### Example

You can use the default JWK:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;

$passes = LittleJWT::validateToken($token, function (Validator $validator) {
    // Checks the JWT signature is valid using the default JWK before performing other checks.
    $validator->valid();
});
```

You can use a different JWK:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Validation\Validator;
use Jose\Component\KeyManagement\JWKFactory;

$jwk = JWKFactory::createFromSecret(/* ... */);

// Checks the JWT signature is valid using the specified JWK before performing other checks.
$passes = LittleJWT::validateToken($token, function (Validator $validator) use ($jwk) {
    $validator->valid($jwk);
});

// Checks the JWT signature is valid using the JWK associated with LittleJWT before performing other checks.
$passes = LittleJWT::withJwk($jwk)->validateToken($token, function (Validator $validator) {
    $validator->valid();
});
```

# Custom Rules

Besides using a callback to define a custom rule, you can define a class.

## Custom JWT Rule

To define a custom rule that checks entire JWT, create a class that extends the ``LittleApps\LittleJWT\JWT\Rules\Rule`` abstract class:

```php
use LittleApps\LittleJWT\JWT\Rules\Rule;

class MyRule extends Rule
{
    /**
     * Checks if JWT passes rule.
     *
     * @param \LittleApps\LittleJWT\JWT\JWT $jwt
     * @return bool True if JWT passes rule check.
     */
    public function passes(JWT $jwt)
    {
        // Check the $jwt here and return true or false.
    }

    /**
     * Gets the error message for when the rule fails.
     *
     * @return string
     */
    public function message()
    {
        return 'This is the error message to use if the rule does not pass.';
    }
}
```

Add the rule to the Validator instance:

```php
$validator->addRule(new MyRule);
```

## Custom Claim Rule

To define a custom rule that checks a claim value, create a class that extends the ``LittleApps\LittleJWT\JWT\Rules\Claims\Rule`` abstract class:

```php
use LittleApps\LittleJWT\JWT\Rules\Claims\Rule;

class MyClaimRule extends Rule
{
    /**
     * Checks that a claim is valid, if it exists.
     *
     * @param JWT $jwt
     * @param mixed $value
     * @return bool
     */
    protected function checkClaim(JWT $jwt, $value)
    {
        // Check the $value here and return true or false.
    }

    /**
     * Formats a message for a claim check.
     *
     * @return string
     */
    protected function formatMessage()
    {
        return "Claim with key ':key' is invalid.";
    }
}
```

Add the rule to the Validator instance:

```php
$validator->addRule(new MyClaimRule);
```

# Method Chaining

Similar to [The Builder](/the-builder), method chaining can be done to specify multiple rules:

```php
$validator
    ->valid()               // Checks the signature is valid.
    ->equals('abc', 'def'); // Checks claim 'abc' in payload equals 'def'
```

# Stop On Failure

The validation will not perform any further checks once one of the checks fails with stop on failure. This is enabled by default.

## Parameters

| Parameter | Type | Description |
|-|-|-|
| ``$enabled`` | ``bool`` | If true, validation stops when first rule fails. (default: ``true``) |

## Example

This can be enabled or disabled with the ``stopOnFailure()`` method:

```php
// Enables stop on failure
$validator->stopOnFailure(true);

// Disables stop on failure
$validator->stopOnFailure(false);
```