# Minimal example for issue \#670 of nWidart/laravel-modules package

For the minimal Bug example to run out of the box, please set up the Laravel app as usual. For that, copy the `.env.example` file to `.env` and generate an app key by running `php artisan key:generate`. Don't forget to install all requirements – run `composer install` if `composer` is [globally installed](https://getcomposer.org/doc/03-cli.md#global).

The `.env.example` expects you to point the URL `example.com` and the corresponding subdomain `subdomain.example.com` to this Laravel instance.

## working commit – without Module

The commit `633a1c1e9240adc2d1d8d30a4962b50154e16933` ("basic reconstructions – SubDomain WORKING") shows the basic minimal setup with working route `/` for subdomain. Route is defined within `routes/web.php` as that – where the 5 first lines are the relevant ones:

```php
Route::group(['domain' => env('APP_2URL')], function(){
    Route::get('/', function () {
        return view('subdomain');
    });
});

Route::get('/', function () {
    return view('default');
});
```

## non-working – configured Module

Within the history, the next commit just added the Laravel requirement `nWidart/laravel-modules` to the Laravel app (commit `2b764c0def62a4b2071811d46260920ec15e8ecf`).

After that, I created a Module `SubDomain` by the artisan command and added it without changing the routing. Therefor even commit `32c60d0e91ce60b6a4cb216d37b58abffc9c4581` still is working.

Now, commit `40333aca9c43a2e98d71bcf28fdc982a3218e8d5` is the relevant commit – I moved the relevant snippet to `Modules/SubDomain/Routes/web.php`. Now the route-files look like that:


```php
<?php

// routes/web.php

Route::get('/', function () {
    return view('default');
});
```

```php
<?php

// Modules/SubDomain/Routes/web.php

Route::group(['domain' => env('APP_2URL')], function(){
    Route::get('/', function () {
        return view('subdomain');
    });
});
```

And now both previously working routes provide the default content, not the expected one.

## `php artisan route:list`

### working commit `633a1c1e9240adc2d1d8d30a4962b50154e16933`

```
+-----------------------+----------+----------+------+---------+--------------+
| Domain                | Method   | URI      | Name | Action  | Middleware   |
+-----------------------+----------+----------+------+---------+--------------+
|                       | GET|HEAD | /        |      | Closure | web          |
| subdomain.example.com | GET|HEAD | /        |      | Closure | web          |
|                       | GET|HEAD | api/user |      | Closure | api,auth:api |
+-----------------------+----------+----------+------+---------+--------------+
```

### non-working commit `40333aca9c43a2e98d71bcf28fdc982a3218e8d5`

```
+-----------------------+----------+---------------+------+---------+--------------+
| Domain                | Method   | URI           | Name | Action  | Middleware   |
+-----------------------+----------+---------------+------+---------+--------------+
|                       | GET|HEAD | /             |      | Closure | web          |
| subdomain.example.com | GET|HEAD | /             |      | Closure | web          |
|                       | GET|HEAD | api/subdomain |      | Closure | api,auth:api |
|                       | GET|HEAD | api/user      |      | Closure | api,auth:api |
+-----------------------+----------+---------------+------+---------+--------------+
```

## last confusing fact

Thinking about Laravel mixing up routes totally, I tried to additionally add the `/` route for the primary URL to the module – and now the main `routes/web.php` doesn't do anything anymore.

So ... let's write down the configs:

```php
<?php

// routes/web.php

Route::get('/', function () {
    return view('default');
});
```

```php
<?php

// Modules/SubDomain/Routes/web.php

Route::group(['domain' => env('APP_2URL')], function(){
    Route::get('/', function () {
        return view('subdomain');
    });
});

Route::get('/', function () {
    return view('subdomain2');
});
```

With this configuration, both – `example.com` and `subdomain.example.com` – show up the `subdomain2`-View ... now, the newly added `/` route overrides every route defined until now.
