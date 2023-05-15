---
title: Testing
description: 
published: true
date: 2022-03-27T04:44:16.091Z
tags: claims, custom jwk, default jwk, jwk, rules, validation, validator, testing
editor: markdown
dateCreated: 2022-03-27T04:19:59.314Z
---

Little JWT provides support for testing JSON Web Tokens from within PHPUnit tests.

Normally, the [validator rules](https://docs.getlittlejwt.com/en/the-validator) would be used to determine if a JWT is valid or not. For testing purposes, Little JWT can be faked so the rules are assertions that cause the test to pass or fail instead.

Before performing assertions on the JWT, call the ``fake`` method in the ``LittleJWT`` facade. This will cause the callback for the ``validateJWT`` and ``validateToken`` methods to receive a ``TestValidator`` instance. The ``TestValidator`` class provides assertion methods (listed below) as well as the [existing ``Validator`` methods](/the-validator).

```php
use Tests\TestCase;

use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Testing\TestValidator;

class MyTests extends TestCase {
    public function test_example() {
        LittleJWT::fake();
        
        $jwt = LittleJWT::createJWT();

        LittleJWT::validateJWT($jwt, function (TestValidator $validator) {
            // Method chaining can be used (just like with Validator methods)
            $validator
                ->assertPasses()
                ->assertClaimMatches('abc', 'def');
        });
    }
}

```

# Available Assertions

The following is a list of available assertions:

## assertPasses/assertFails

Asserts that the JWT is valid or invalid. Only ``assertPasses`` or ``assertFails`` can be used, not both.

```php
// Assertion passes if the JWT is valid.
$validator->assertPasses();

// Assertion passes if the JWT is invalid.
$validator->assertFails();
```

## assertErrorCount/assertNotErrorCount

Asserts that there is or isn't ``$n`` number of errors after the validation. Specifying ``false`` will disable the assertion.

```php
$n = 5;

// Assertion passes if there is 5 errors from the JWT validation.
$validator->assertErrorCount($n);

// Assertion passes if there isn't 5 errors from the JWT validation.
$validator->assertNotErrorCount($n);
```

## assertErrorKeyExists/assertErrorKeyDoesntExist

Asserts that an error with ``$key`` does or doesn't exist after the validation. The ``$key`` will be either the the fully qualified class name for rules that apply to the entire JWT, or for rules that apply to specific claims, the key of the claim that failed.

```php
$key = 'sub';

// Assertion passes if an error with key 'sub' exists.
$validator->assertErrorKeyExists($key);

$key = \LittleApps\LittleJWT\JWT\Rules\ValidSignature::class;

// Assertion passes if no error with the key 'LittleApps\LittleJWT\JWT\Rules\ValidSignature' exists.
$validator->assertErrorKeyDoesntExist($key);
```

## assertRulePasses/assertRuleFails

Asserts that the specified ``$rule`` instance passes or fails. A ``$message`` can optionally be specified which will be displayed if the assertion fails.

```php
use LittleApps\LittleJWT\JWT\Rules\Claims\Equals;

$rule = new Equals('abc', 'def');
$message = "Claim 'abc' does not equal 'def'";

// Assertion passes if claim with key 'abc' equals 'def'.
// PHPUnit will display $message if the assertion fails.
$validator->assertRulePasses($rule, $message);

// Assertion passes if claim with key 'abc' is not equal to 'def'.
$validator->assertRuleFails($rule);
```

## assertSubjectModel/assertNotSubjectModel

Asserts that the 'prv' claim does or doesn't match the specified ``$model`` class.

```php
$model = \App\Models\User::class;

// Assertion passes if the value of the 'prv' claim equals the hash of 'App\Models\User' (d59a159d1113876f898c70f94a3971da6bb3165b1a8f0c07236c92d911df85ed)
$validator->assertSubjectModel($model);

// Assertion passes if the value of the 'prv' claim is not equal to the hash of 'App\Models\User'
$validator->assertNotSubjectModel($model);
```

## assertPastPasses/assertPastFails

Asserts that the date/time claim with ``$key`` is or isn't in the past. The ``$leeway`` can optionally be specified as the number of seconds from now for it to be considered in the past. If the claim is in the header, set ``$inHeader`` as true.

```php
$key = 'iat';
$leeway = 60;
$inHeader = true;

// Asserts the 'iat' claim in the header is in the past by +/- 60 seconds.
$validator->assertPastPasses($key, $leeway, $inHeader);

// Asserts the 'iat' claim in the payload is in the past.
$validator->assertPastPasses($key);

// Asserts the 'iat' claim in the payload is not in the past.
$validator->assertPastFails($key);
```

## assertFuturePasses/assertFutureFails

Asserts that the date/time claim with ``$key`` is or isn't in the future. The ``$leeway`` can optionally be specified as the number of seconds after now for it to be considered in the future. If the claim is in the header, set ``$inHeader`` as true.

```php
$key = 'exp';
$leeway = 60;
$inHeader = true;

// Asserts the 'exp' claim in the header is in the future by +/- 60 seconds.
$validator->assertFuturePasses($key, $leeway, $inHeader);

// Asserts the 'exp' claim in the payload is in the future.
$validator->assertFuturePasses($key);

// Asserts the 'exp' claim in the payload is not in the future.
$validator->assertFutureFails($key);
```

## assertClaimMatches/assertClaimDoesntMatch

Asserts that the claim with ``$key`` does or doesn't matches the ``$value``. Optionally, specify ``$strict`` as true to perform an [identical comparison](https://www.php.net/manual/en/language.operators.comparison.php). If the claim is in the header, set ``$inHeader`` as true.

```php
$key = 'abc';
$value = 100;
$strict = true;
$inHeader = true;

// Asserts that the 'abc' claim in the header is identical to 100
$validator->assertClaimMatches($key, $value, $strict, $inHeader);

// Asserts that the 'abc' claim in the payload equals 100 or '100'
$validator->assertClaimMatches($key, $value);

// Asserts that the 'abc' claim in the header isn't identical to 100
$validator->assertClaimDoesntMatch($key, $value, $strict, $inHeader);
```

## assertClaimsExists/assertClaimsDoesntExist

Asserts that claims with the specified keys exist or don't exist in the header or payload. Optionally, specify ``$strict`` as true to assert that *only* the specified keys exist in the header or payload. If checking the header, specify ``$inHeader`` as true.

```php
$expected = ['abc', 'def'];
$strict = true;
$inHeader = true;

// Asserts that the header only has claims with the keys 'abc' and 'def'.
$validator->assertClaimsExists($expected, $strict, $inHeader);

// Asserts that the payload contains claims with the keys 'abc' and 'def'.
$validator->assertClaimsExists($expected);

// Each claim key to check for can be specified as seperate parameters instead of a single array.
// This doesn't allow for a strict comparison or for header claims to be checked.
$validator->assertClaimsExists('abc', 'def');

// Asserts that the header doesn't only have the claims with the keys 'abc' and 'def'.
$validator->assertClaimsDoesntExist($expected, $strict, $inHeader);
```

## assertValidSignature/assertInvalidSignature

Asserts that the JWT signature is valid or invalid. A [JSON Web Key (JWK)](/json-web-keys) can optionally be specified as ``$jwk``. If not specified, the default JWK will be used.

```php
$validator->assertValidSignature();
$validator->assertInvalidSignature();
```
## assertAllowed/assertNotAllowed

Asserts that the JWT is or isn't blacklisted. The blacklist driver to check can optionally be specified as ``$driver``. If not specified, the default blacklist driver is used.

```php
$driver = 'cache';

// Asserts the JWT is allowed (not blacklisted) by the 'cache' blacklist driver.
$validator->assertAllowed($driver);

// Asserts the JWT isn't allowed (blacklisted) by the default blacklist driver.
$validator->assertNotAllowed();
```

## assertCustomPasses/assertCustomFails

Asserts that the JWT passes or doesn't pass the specified ``$callback``. The ``$callback`` accepts the ``JWT`` instance and returns either true (meaning it passes) or false (meaning it fails).

```php
use LittleApps\LittleJWT\JWT\JWT;

$callback = function (JWT $jwt) {
    if ($jwt->getPayload()->count() == 1)
        return true;
    else
        return false;
};

// Assertion will pass if JWT only has 1 claim in the payload.
$validator->assertCustomPasses($callback);

// Assertion will pass if JWT doesn't have 1 claim in the payload.
$validator->assertCustomFails($callback);
```

## assertCustomClaimPasses/assertCustomClaimFails

Similiar to ``assertCustomPasses`` and ``assertCustomFails``, except for perform checks on specific claims with ``$key``. The ``$callback`` recieves the claim ``$value``, the claim ``$key``, and the associated JWT instance as ``$jwt``. If the claim is in the header, specify ``$inHeader`` as true.

```php
use LittleApps\LittleJWT\JWT\JWT;

$key = 'abc';
$callback = function($value, string $key, JWT $jwt) {
    // Checks the $value is all lowercase.
    if (strtolower($value) == $value)
        return true;
    else
        return false;
};
$inHeader = true;

// Assertion will pass if the header claim with key 'abc' is all lowercase.
$validator->assertCustomClaimPasses($key, $callback, $inHeader);

// Assertion will pass if the payload claim with key 'abc' is all lowercase.
$validator->assertCustomClaimPasses($key, $callback);

// Assertion will pass if the payload claim with key 'abc' is not all lowercase.
$validator->assertCustomClaimFails($key, $callback);
```