---
title: Configuration
description: 
published: true
date: 2022-08-21T08:16:49.608Z
tags: config, configuration, littlejwt.php, publish, package, file
editor: markdown
dateCreated: 2022-02-05T06:57:08.423Z
---

The configuration options for Little JWT are located in the [``config/littlejwt.php``](https://github.com/little-apps/LittleJWT/blob/main/config/littlejwt.php) file. 

If you don't see this file, you will need to publish it with the following command:

```bash
php artisan vendor:publish --tag="littlejwt-config"
```

If you want to use the database driver for the [blacklist manager](), you may need to publish the migration for the it:

```bash
php artisan vendor:publish --tag="littlejwt-migrations"
```