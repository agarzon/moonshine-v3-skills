# Configuration Options Reference

Complete reference for MoonShine v3 general configuration, feature flags, routing, middleware, and storage options.

---

## Table of Contents

- [Full default config/moonshine.php](#full-default-config)
- [General options](#general-options)
- [Feature flags](#feature-flags)
- [Routing](#routing)
- [Error handling](#error-handling)
- [Middleware stack](#middleware-stack)
- [Storage and cache](#storage-and-cache)

---

## Full default config/moonshine.php

This is the complete default configuration file as shipped with MoonShine v3:

```php
<?php

use Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse;
use Illuminate\Cookie\Middleware\EncryptCookies;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;
use Illuminate\Routing\Middleware\SubstituteBindings;
use Illuminate\Session\Middleware\AuthenticateSession;
use Illuminate\Session\Middleware\StartSession;
use Illuminate\View\Middleware\ShareErrorsFromSession;
use MoonShine\Laravel\Exceptions\MoonShineNotFoundException;
use MoonShine\Laravel\Forms\FiltersForm;
use MoonShine\Laravel\Forms\LoginForm;
use MoonShine\Laravel\Http\Middleware\Authenticate;
use MoonShine\Laravel\Http\Middleware\ChangeLocale;
use MoonShine\Laravel\Layouts\AppLayout;
use MoonShine\Laravel\Models\MoonshineUser;
use MoonShine\Laravel\Pages\Dashboard;
use MoonShine\Laravel\Pages\ErrorPage;
use MoonShine\Laravel\Pages\LoginPage;
use MoonShine\Laravel\Pages\ProfilePage;

return [
    'title' => env('MOONSHINE_TITLE', 'MoonShine'),
    'logo' => 'vendor/moonshine/logo.svg',
    'logo_small' => 'vendor/moonshine/logo-small.svg',

    // Default flags
    'use_migrations' => true,
    'use_notifications' => true,
    'use_database_notifications' => true,
    'use_routes' => true,
    'use_profile' => true,

    // Routing
    'domain' => env('MOONSHINE_DOMAIN'),
    'prefix' => env('MOONSHINE_ROUTE_PREFIX', 'admin'),
    'page_prefix' => env('MOONSHINE_PAGE_PREFIX', 'page'),
    'resource_prefix' => env('MOONSHINE_RESOURCE_PREFIX', 'resource'),
    'home_route' => 'moonshine.index',

    // Error handling
    'not_found_exception' => MoonShineNotFoundException::class,

    // Middleware
    'middleware' => [
        EncryptCookies::class,
        AddQueuedCookiesToResponse::class,
        StartSession::class,
        AuthenticateSession::class,
        ShareErrorsFromSession::class,
        VerifyCsrfToken::class,
        SubstituteBindings::class,
        ChangeLocale::class,
    ],

    // Storage
    'disk' => 'public',
    'disk_options' => [],
    'cache' => 'file',

    // Authentication
    'auth' => [
        'enabled' => true,
        'guard' => 'moonshine',
        'model' => MoonshineUser::class,
        'middleware' => Authenticate::class,
        'pipelines' => [],
    ],

    // User field mapping
    'user_fields' => [
        'username' => 'email',
        'password' => 'password',
        'name' => 'name',
        'avatar' => 'avatar',
    ],

    // Layout, pages, forms
    'layout' => AppLayout::class,

    'forms' => [
        'login' => LoginForm::class,
        'filters' => FiltersForm::class,
    ],

    'pages' => [
        'dashboard' => Dashboard::class,
        'profile' => ProfilePage::class,
        'login' => LoginPage::class,
        'error' => ErrorPage::class,
    ],

    // Localizations
    'locale' => 'en',
    'locale_key' => ChangeLocale::KEY,
    'locales' => [
        // en
    ],
];
```

---

## General options

### title

**Config key:** `title`
**Default:** `env('MOONSHINE_TITLE', 'MoonShine')`
**Provider method:** `$config->title('My Application')`

Sets the HTML `<title>` meta tag displayed on all admin panel pages.

```php
// config/moonshine.php
'title' => 'My Admin Panel',
```

### logo / logo_small

**Config keys:** `logo`, `logo_small`
**Default:** `'vendor/moonshine/logo.svg'`, `'vendor/moonshine/logo-small.svg'`
**Provider method:** `$config->logo('/path/to/logo.png')` and `$config->logo('/path/to/logo-small.png', small: true)`

Paths to the full-size and collapsed sidebar logo images.

```php
// config/moonshine.php
'logo' => '/assets/logo.png',
'logo_small' => '/assets/logo-small.png',
```

```php
// MoonShineServiceProvider
$config
    ->logo('/assets/logo.png')
    ->logo('/assets/logo-small.png', small: true);
```

### dir / namespace

**Config keys:** `dir`, `namespace`
**Default:** `'app/MoonShine'`, `'App\MoonShine'`
**Provider method:** `$config->dir('app/MoonShine', 'App\MoonShine')`

Directory and namespace used by artisan generators to create MoonShine classes (resources, pages, layouts). MoonShine itself is not tied to this directory at runtime.

```php
// config/moonshine.php
'dir' => 'app/MoonShine',
'namespace' => 'App\MoonShine',
```

---

## Feature flags

These boolean flags control which MoonShine subsystems are active. They must always be explicitly present in either `config/moonshine.php` or `MoonShineServiceProvider`.

### use_migrations

**Default:** `true`
**Provider method:** `$config->useMigrations()`

Enables MoonShine's built-in database migrations (`moonshine_users`, `moonshine_user_roles` tables). Set to `false` if you manage your own user/role tables.

### use_notifications

**Default:** `true`
**Provider method:** `$config->useNotifications()`

Enables the MoonShine notification system in the admin panel UI.

### use_database_notifications

**Default:** `true`
**Provider method:** `$config->useDatabaseNotifications()`

When enabled, notifications are persisted using Laravel's database notification driver. Requires the `notifications` migration table.

### use_routes

**Default:** `true`

Enables MoonShine's built-in route registration. Set to `false` only if you need to register all MoonShine routes manually.

### use_profile

**Default:** `true`

Enables the built-in user profile page in the admin panel.

---

## Routing

> **Warning**: Route-related configuration must be defined in `config/moonshine.php`, not in the service provider. Routes are loaded before the provider's `boot` method runs.

### domain

**Config key:** `domain`
**Default:** `env('MOONSHINE_DOMAIN')` (null)

Restricts MoonShine routes to a specific domain. Useful for subdomain-based admin panels.

```php
'domain' => 'admin.example.com',
```

### prefix

**Config key:** `prefix`
**Default:** `env('MOONSHINE_ROUTE_PREFIX', 'admin')`

The URL prefix for all MoonShine routes. The admin panel will be available at `/{prefix}`.

```php
'prefix' => 'admin',     // Panel at /admin
'prefix' => 'dashboard', // Panel at /dashboard
```

### page_prefix

**Config key:** `page_prefix`
**Default:** `env('MOONSHINE_PAGE_PREFIX', 'page')`

URL segment for standalone pages: `/{prefix}/{page_prefix}/{pageUri}`.

### resource_prefix

**Config key:** `resource_prefix`
**Default:** `env('MOONSHINE_RESOURCE_PREFIX', 'resource')`

URL segment for resources: `/{prefix}/{resource_prefix}/{resourceUri}/{pageUri}`.

> **Warning**: You can leave `resource_prefix` empty so URLs look like `/admin/{resourceUri}/{pageUri}`, but this may create conflicts with package routes.

### home_route / home_url

**Config key:** `home_route` or `home_url`
**Default:** `'moonshine.index'`
**Provider methods:** `$config->homeRoute('moonshine.index')` or `$config->homeUrl('/admin/page/some-page')`

The home page of the admin panel. Used for post-login redirects, the logo link, and the 404 page "go home" link.

```php
// Named route
'home_route' => 'moonshine.index',

// Or a direct URL
'home_url' => '/admin/page/some-page',
```

---

## Error handling

### not_found_exception

**Config key:** `not_found_exception`
**Default:** `MoonShineNotFoundException::class`
**Provider method:** `$config->notFoundException(MyNotFoundException::class)`

The exception class thrown when a MoonShine route is not found. Replace with your own to customize 404 behavior.

---

## Middleware stack

**Config key:** `middleware`

The default middleware stack applied to all MoonShine routes. This operates independently from your application's `web` middleware group.

### Default middleware (in order)

| Middleware | Purpose |
|-----------|---------|
| `EncryptCookies` | Encrypts/decrypts cookies |
| `AddQueuedCookiesToResponse` | Attaches queued cookies to the response |
| `StartSession` | Initializes the session |
| `AuthenticateSession` | Validates the authenticated session |
| `ShareErrorsFromSession` | Shares validation errors with views |
| `VerifyCsrfToken` | CSRF protection |
| `SubstituteBindings` | Route model binding |
| `ChangeLocale` | Handles admin panel locale switching |

### Customizing the middleware stack

You can override the entire stack or append additional middleware:

```php
// config/moonshine.php
'middleware' => [
    \Illuminate\Cookie\Middleware\EncryptCookies::class,
    \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \Illuminate\Foundation\Http\Middleware\VerifyCsrfToken::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \MoonShine\Laravel\Http\Middleware\ChangeLocale::class,
    // Add your custom middleware here:
    \App\Http\Middleware\LogAdminActions::class,
],
```

### Authentication middleware

The auth-specific middleware is configured separately in `auth.middleware`:

```php
'auth' => [
    'middleware' => \MoonShine\Laravel\Http\Middleware\Authenticate::class,
],
```

This middleware checks whether the user has a valid session. It is applied in addition to the main middleware stack when `auth.enabled` is `true`.

---

## Storage and cache

### disk

**Config key:** `disk`
**Default:** `'public'`
**Provider method:** `$config->disk('public')`

The Laravel filesystem disk used for file uploads in MoonShine (images, file fields, etc.).

### disk_options

**Config key:** `disk_options`
**Default:** `[]`
**Provider method:** `$config->disk('public', options: [...])`

Additional options passed to the filesystem disk when storing files.

```php
'disk' => 's3',
'disk_options' => [
    'visibility' => 'public',
],
```

### cache

**Config key:** `cache`
**Default:** `'file'`
**Provider method:** `$config->cacheDriver('redis')`

The Laravel cache driver used by MoonShine for internal caching (e.g., menu caching, configuration caching).

```php
// Use Redis for MoonShine cache
'cache' => 'redis',
```
