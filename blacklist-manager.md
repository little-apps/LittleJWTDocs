---
title: Blacklist Manager
description: 
published: true
date: 2023-05-15T06:22:03.039Z
tags: blacklist, cache, database, manager
editor: markdown
dateCreated: 2022-08-21T08:16:03.511Z
---

 Unlike sessions where the data (or "payload") is stored on the server-side and identifier on the client-side, JWTs (JSON Web Tokens) store both the identifier and payload on the client-side (which is verified on the server using the signature). A JWT can be invalidated or blocked on the server-side by adding the identifier to a blacklist.

# Configuration

Little JWT provides the following drivers:

 * **Cache Driver**: Stores blacklisted JWTs in [Laravel's cache](https://laravel.com/docs/9.x/cache). 
 * **Database Driver**: Stores blacklisted JWTs in a table in the database. 
 
The configuration options are located in the [``config/littlejwt.php`` file](https://github.com/little-apps/LittleJWT/blob/main/config/littlejwt.php#L220). You can set the driver that will maintain the blacklist and the configuration options for each driver. 
 
# Usage

## Checking

Use the ``Blacklist`` facade to access the default driver. The following demonstrates checking if a JWT is blacklisted:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use LittleApps\LittleJWT\Facades\Blacklist;

// The current JsonWebToken instance can be retrieved from the \Illuminate\Http\Request object.
$jwt = $request->getJwt();

$blacklisted = Blacklist::isBlacklisted($jwt);
```

If you don't want to use the default driver, you can specify the driver:

```php
$driver = Blacklist::driver('database');
$blacklisted = $driver->isBlacklisted($jwt);
```

 > [The Validator](/the-validator#allowed) also provides the 'allowed' rule to ensure a JWT isn't blacklisted when it's validated.


## Blacklisting

The following demonstrates blacklisting a JWT for the default amount of time (which is forever):

```php
use LittleApps\LittleJWT\Facades\Blacklist;

$jwt = /* ... */;

// The blacklist method only accepts instances of \LittleApps\LittleJWT\JWT\JWT.
Blacklist::blacklist($jwt);
```

The following demonstrates blacklisting a JWT for 1 hour:

```php
use LittleApps\LittleJWT\Facades\Blacklist;

$jwt = /* ... */;
$ttl = 3600;

Blacklist::blacklist($jwt, $ttl);
```

If TTL (Time To Live) is 0 then the JWT will be blacklisted forever:


```php
use LittleApps\LittleJWT\Facades\Blacklist;

$jwt = /* ... */;
$ttl = 0;

Blacklist::blacklist($jwt, $ttl);
```

## Purging

The blacklist can be purged from the command line:

```bash
php artisan littlejwt:purge
```

The driver to use can be specified:

```bash
php artisan littlejwt:purge database
```

> Schedule calls to the command to purge the blacklist on a regular basis with [Laravel's Task Scheduling](https://laravel.com/docs/9.x/scheduling#scheduling-artisan-commands).

The blacklist can also be purged pragmatically by calling the purge method:


```php
use LittleApps\LittleJWT\Facades\Blacklist;

Blacklist::purge();
```

> Note: If the cache driver is used, a purge will do nothing since Laravel handles expired cache entries itself.

# Custom Drivers

If you want to create your own blacklist driver, you will need to create a class that implements ``BlacklistDriver``:

```php
<?php

namespace App\Components\LittleJWT;

use LittleApps\LittleJWT\Contracts\BlacklistDriver;
use LittleApps\LittleJWT\JWT\JsonWebToken;

class MyCustomDriver implements BlacklistDriver
{
    /**
     * Checks if JWT is blacklisted.
     *
     * @param JsonWebToken $jwt
     * @return bool True if blacklisted.
     */
    public function isBlacklisted(JsonWebToken $jwt)
    {
        // ...
    }

    /**
     * Blacklists a JWT.
     *
     * @param JsonWebToken $jwt
     * @param int $ttl Length of time (in seconds) a JWT is blacklisted (0 means forever). If negative, the default TTL is used. (default: -1)
     * @return $this
     */
    public function blacklist(JsonWebToken $jwt, $ttl = -1)
    {
        // ...
        return $this;
    }

    /**
     * Cleanup blacklist.
     *
     * @return $this
     */
    public function purge()
    {
        // ...
        return $this;
    }
}
```

The ``JWTHelpers`` trait provides a method to determine a unique ID from a JWT. You can use it if you want to blacklist just the JTI (a UUID) or a SHA-1 hash for the entire JWT (if a JTI doesn't exist):

```php
use LittleApps\LittleJWT\Contracts\BlacklistDriver;
use LittleApps\LittleJWT\JWT\JsonWebToken;
use LittleApps\LittleJWT\Concerns\JWTHelpers;

class MyCustomDriver implements BlacklistDriver
{
    use JWTHelpers;

    /**
     * Checks if JWT is blacklisted.
     *
     * @param JsonWebToken $jwt
     * @return bool True if blacklisted.
     */
    public function isBlacklisted(JsonWebToken $jwt)
    {
        $id = $this->getUniqueId($jwt);
        
        // ...
    }
   
    // ...
}
```

The custom driver will need to be registered in a service provider:

```php
use Illuminate\Support\ServiceProvider;
use LittleApps\LittleJWT\Facades\Blacklist;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blacklist::extend('mydriver', function ($app) {
        // Return instance of your custom driver here
    });
}
```

If you want the custom driver to be the default driver, you will need to set it in the ``config/littlejwt.php`` configuration file:

```php
<?php

return [
    // ...
    
    'blacklist' => [
        'driver' => 'mydriver',
        
        // ...
    ]
];
```
