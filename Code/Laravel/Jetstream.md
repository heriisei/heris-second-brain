---
tags: php, laravel, jetstream, inertia, vue
---

```ad-example
title: Navigation
👈 Previous - [[]]

👉 Next - [[]]
```

# Initialization
## Prerequisite
- Install [[Composer]].

## Install
- Run `composer create-project laravel/laravel yourprojectname`.
- Test it by run the `php artisan serve` command.
- Follow the instructions from [the documentation](https://jetstream.laravel.com/2.x/installation.html)
- `composer require laravel/jetstream`
- `php artisan jetstream:install inertia`
- `npm install && npm run build`
- `npm run dev`
- Open `./.env` and update the `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` using the database & user that you have created (create a new one if not yet).
- `sudo apt install php8.1-mysql`.
- `php artisan migrate`.
- `php artisan serve`, now you can use the authentication / user management feature from inside the laravel homepage `localhost:8000`.
- 