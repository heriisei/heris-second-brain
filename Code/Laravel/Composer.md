---
tags: php, composer, laravel
---

```ad-example
title: Navigation
👈 Previous - [[]]

👉 Next - [[]]
```

# Install
- Follow the instruction from [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-composer-on-ubuntu-22-04).

## Optional
These steps may be required if there are errors occurs when you try to do a `composer create-project laravel/laravel yourproject`.
```txt
- Root composer.json requires laravel/pint ^1.0 -> satisfiable by laravel/pint[v1.0.0, ..., v1.2.0].
- laravel/pint[v1.0.0, ..., v1.2.0] require ext-xml * -> it is missing from your system. Install or enable PHP's xml extension.
Problem 2
- phpunit/phpunit[9.5.10, ..., 9.5.x-dev] require ext-dom * -> it is missing from your system. Install or enable PHP's dom extension.
- Root composer.json requires phpunit/phpunit ^9.5.10 -> satisfiable by phpunit/phpunit[9.5.10, ..., 9.5.x-dev].

```
- Solution:
	- `sudo apt install php-xml`

```txt
- spatie/laravel-ignition[1.0.0, ..., 1.5.2] require ext-curl * -> it is missing from your system. Install or enable PHP's curl extension.
- Root composer.json requires spatie/laravel-ignition ^1.0 -> satisfiable by spatie/laravel-ignition[1.0.0, ..., 1.5.2].
```
- Solution:
	- `sudo apt install php-curl`
