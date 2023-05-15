---
title: Home
description: Generate and verify JSON Web Tokens (JWTs) simply in Laravel using Little JWT.
published: true
date: 2022-04-14T06:07:15.106Z
tags: jwt, token, getting started, composer, artisan
editor: markdown
dateCreated: 2022-02-05T06:45:08.312Z
---

# Getting Started

Install the package via composer:

```bash
composer require little-apps/littlejwt
```

Publish the config file with:

```bash
php artisan vendor:publish --tag="littlejwt-config"
```

Generate a secret phrase for building and validating JWTs:

```bash
php artisan littlejwt:phrase
```

 > See [JSON Web Keys (JWKs)](/json-web-keys) for generating other types of keys.

# Token vs. JWT

You will see the terms "token" and "JWT" mentioned in this document. The difference between the two is:

 * A JWT is a JSON Web Token represented as an [JWT instance](/the-jwt). It contains the de-serialized claims and signature.
 * A token is a JSON Web Token represented as a string. This is after the header/payload claims have been encoded in JSON form and the 3 parts (header claims, payload claims, and signature) have been encoded as base64. 
 
# Security

 * Please review the [JSON Web Token Cheat Sheet for Java](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html#json-web-token-cheat-sheet-for-java) to find out about JWT best practices.
 * The [security policy](https://github.com/little-apps/LittleJWT/security/policy) contains information on reporting vulnerabilities.
 * Any [security advisories](https://github.com/little-apps/LittleJWT/security/advisories) directly related to Little JWT can also be found on GitHub.