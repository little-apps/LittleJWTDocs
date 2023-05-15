---
title: Buildables
description: 
published: true
date: 2023-05-15T06:24:58.011Z
tags: build, buildable, config, configuration, createtoken, creating, custom buildable, default buildable, littlejwt.php, resolving
editor: markdown
dateCreated: 2022-02-13T09:05:19.044Z
---

Rather than passing an anonymous function to the ``create()`` method, you could create an instance of an [invokable class](https://www.php.net/manual/en/language.oop5.magic.php#object.invoke) to re-use the claims:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

// Create invokable instance directly:
$buildable = new MyBuildable();

// Pass $buildable to the create method:
$jwt = LittleJWT::create($buildable);
```

# Configuration

The ``config/littlejwt.php`` file contains configuration options for buildables:

```php
return [
    'buildables' => [
        'default' => [
            /**
             * Buildable instance to use for this builder.
             */
            'buildable' => \LittleApps\LittleJWT\Build\Builders\DefaultBuilder::class,

            /* ... */
        ]
    ]
];
```

# Bundled Buildables

Little JWT comes bundled with the following pre-existing buildables:

## Default Buildable

The configuration options for setting the default claims are located in the ``config/littlejwt.php`` file under ``buildables.default``.

The claims added by the Default Buildable are listed below:

| Claim Key  | Description |
| ---------- | ------------- |
| alg        | See the configuration options.  |
| iat        | Set to current date and time.  |
| nbf        | Set to current date and time.  |
| exp        | Set to current date and time plus the number of seconds set in the configuration.  |
| iss        | See the configuration options.  |
| jti        | Set to a newly generated UUID.  |
| aud        | See the configuration options.  |

# Custom Buildables

Create a custom buildable by creating a class that implements the ``__invoke`` method:


```php
use LittleApps\LittleJWT\Build\Builder;

class MyBuildable
{
    public function __construct()
    {
    }

    public function __invoke(Builder $builder)
    {
        /* ... */
    }
}
```

Send the ``$buildable`` instance to the ``create()`` method:

```php
use LittleApps\LittleJWT\Facades\LittleJWT;

$buildable = new MyBuildable();

$jwt = LittleJWT::create($buildable);
```

## Configuration File

There's advantages to including the custom buildable in the ``config/littlejwt.php`` configuration file:

 1. Configuration options can be easily changed for the buildable.
 2. The buildable instance can be set as the default. 
 3. The buildable instance can be created using Laravel's container.

### Steps
 
To take advantage of the configuration file, add an entry to the ``config/littlejwt.php`` configuration file. Set 'buildable' to the fully qualified class name.


```php
return [
    'buildables' => [
        /* ... */
        
        'custom' => [
            /**
             * Fully qualified buildable class to use.
             */
            'buildable' => \Path\To\MyBuildable::class,
            
            // Configuration options are passed to the buildable.
            'foo' => 'bar'
        ],
    ],
];
```

Make sure the class constructor accepts the configuration as a parameter and sets it to a property. Any keys set in the configuration file entry (besides 'buildable') will be passed when the buildable instance is created by the Laravel container.

```php
use LittleApps\LittleJWT\Build\Builder;

class MyBuildable
{
    protected $config;
    
    public function __construct()
    {
        $this->config = $config;
    }

    public function __invoke(Builder $builder)
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

## Resolving Buildables

If a buildable is registered in the configuration file, it can be resolved using Laravel's container. The  alias is ``littlejwt.buildables.`` followed by the key entry specified in the configuration file.

```php
use LittleApps\LittleJWT\Facades\LittleJWT;
use Illuminate\Support\Facades\App;

$buildable = App::make('littlejwt.buildables.custom');

$jwt = LittleJWT::create($buildable);
```

# Changing The Default Buildable

The default buildable can be changed in the ``config/littlejwt.php`` file.

```php
return [
    'defaults' => [
        /* ... */

        'buildable' => 'default'
    ]
];
```

The value for the ``defaults.buildable`` key corresponds with a key set under ``buildables.*``.

```php
return [
    'buildables' => [
        'default' => [
            /* ... */
        ]
    ]
];
```
