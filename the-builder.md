---
title: The Builder
description: 
published: true
date: 2022-02-21T06:19:29.163Z
tags: claims, create, creating, builder, header, payload, method overloading, property overloading
editor: markdown
dateCreated: 2022-02-05T07:02:02.152Z
---

The ``LittleApps\LittleJWT\Build\Builder`` instance specifies the claims to include in the header or payload of a JWT. There's various ways to set the claims.

# Method Overloading

The most common way to set claims is using method overloading. It is cleaner and allows for chaining.

Simply call a method with the name as the claim key and parameter as the claim value:

```php
$builder->foo('bar'); // Adds 'foo' => 'bar' to payload claims.
```

Use chaining as a cleaner way to set multiple claims:

```php
$builder
   ->foo('bar')
   ->pwd('secret');
```

Pass a boolean value as the second parameter to indicate if the claim should be added to the header or payload:

```php
$builder
   ->foo('bar', true) // Adds 'foo' => 'bar' to header claims.
   ->foo('bar', false); // Adds 'foo' => 'bar' to payload claims.
```

If a second parameter isn't passed, Little JWT determines if the claim should be added to the header or payload claims automatically. Set the claims to be included in the header or payload in the ``config/littlejwt.php`` file (under the ``builder.claims.header`` and ``builder.claims.payload`` whitelists). If the claim key is not included in either whitelist, it is added to the payload by default.

```php
$builder
   ->cty('bar')  // Adds 'cty' => 'bar' to the header claims since it's set in the header claims whitelist and not the payload claims whitelist.
   ->foo('bar'); // Adds 'foo' => 'bar' to the header claims since it's not set in the header or payload claims whitelist
```

# Property Overloading

You can use the setter magic method to set claims and the getter magic method to get claims.

```php
$builder->foo = 'bar'; // Adds 'foo' => 'bar' to payload claims.

$foo = $builder->foo; // Sets $foo to 'bar'
```

# Defined Methods

The Builder instance has defined methods for adding claims to either the header or payload. These methods also allow for chaining.

```php
$builder
   ->addHeaderClaim('foo', 'bar')  // Adds 'foo' => 'bar' to header claims.
   ->addPayloadClaim('foo', 'bar') // Adds 'foo' => 'bar' to payload claims.
   ->addClaim('cty', 'bar');       // Adds 'cty' => 'bar' to header claims (automatically).
```
