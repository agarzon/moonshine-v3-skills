---
name: moonshine-setup-v3
description: Use when installing, configuring, or bootstrapping a MoonShine v3 admin panel. Covers installation, configuration, routing, middleware, localization, and artisan commands.
---

# MoonShine v3 Setup

## Requirements

- PHP 8.2+
- Laravel 10.48+
- Composer 2+

## Installation

### Step 1: Install via Composer

```shell
composer require moonshine/moonshine
```

### Step 2: Run the installer

```shell
php artisan moonshine:install
```

The installer will interactively prompt you to configure:

1. **Authentication** -- enable/disable the middleware that checks panel access.
2. **Migrations** -- required for built-in user and role management (`moonshine_users`, `moonshine_user_roles` tables).
3. **Notifications** -- enable/disable the notification system; optionally use the database driver.
4. **Template theme** -- standard or compact layout.
5. **Superuser** -- create an initial admin user (only if migrations are enabled).

### Quick mode (non-interactive)

```shell
php artisan moonshine:install -Q
```

Accepts all defaults: authentication on, migrations on, notifications on, standard theme, then prompts only for superuser credentials.

### Installer flags

| Flag | Short | Effect |
|------|-------|--------|
| `--quick-mode` | `-Q` | Accept all defaults, skip confirmations |
| `--without-auth` | `-a` | Disable built-in authentication |
| `--without-migrations` | `-m` | Skip system migrations |
| `--without-notifications` | `-d` | Disable notifications |
| `--without-user` | `-u` | Skip superuser creation |
| `--default-layout` | `-l` | Use default (non-compact) layout |
| `--tests-mode` | `-t` | For automated testing |

### Starter kit (single command)

If you have `laravel/installer` globally:

```shell
laravel new example-app --using=moonshine/app
```

## What the installer creates

| Path | Purpose |
|------|---------|
| `app/Providers/MoonShineServiceProvider.php` | Registers resources, pages, and global settings |
| `app/MoonShine/` | Main directory for resources, pages, and layouts |
| `app/MoonShine/Pages/Dashboard.php` | Default dashboard page |
| `app/MoonShine/Layouts/MoonShineLayout.php` | Default layout template |
| `app/MoonShine/Resources/` | Directory for model resources |
| `config/moonshine.php` | Configuration file |
| `lang/vendor/moonshine/` | Language files |
| `public/vendor/moonshine/` | Published assets |

The provider is automatically added to `bootstrap/providers.php`. A `storage:link` is also executed.

After installation, the admin panel is available at `/admin`.

## Project directory structure

```
app/
  MoonShine/
    Layouts/
      MoonShineLayout.php    # Page template (menu, header, sidebar)
    Pages/
      Dashboard.php          # Default landing page
    Resources/               # ModelResource classes for CRUD
  Providers/
    MoonShineServiceProvider.php
config/
  moonshine.php
lang/
  vendor/
    moonshine/               # Translation files
public/
  vendor/
    moonshine/               # Published frontend assets
```

## Configuration methods

MoonShine can be configured two ways. Provider configuration takes precedence over the config file.

### Via config/moonshine.php

```php
return [
    'title' => env('MOONSHINE_TITLE', 'MoonShine'),
    'logo' => 'vendor/moonshine/logo.svg',
    'logo_small' => 'vendor/moonshine/logo-small.svg',
    'use_migrations' => true,
    'use_notifications' => true,
    'use_database_notifications' => true,
    'use_routes' => true,
    'use_profile' => true,
    'prefix' => 'admin',
    'locale' => 'en',
    'auth' => [
        'enabled' => true,
        'guard' => 'moonshine',
    ],
    'layout' => \MoonShine\Laravel\Layouts\AppLayout::class,
];
```

You can keep only the keys that differ from defaults. See [references/config-options.md](references/config-options.md) and [references/config-auth-localization.md](references/config-auth-localization.md) for the full option reference.

### Via MoonShineServiceProvider

```php
use Illuminate\Support\ServiceProvider;
use MoonShine\Contracts\Core\DependencyInjection\CoreContract;
use MoonShine\Laravel\DependencyInjection\MoonShine;
use MoonShine\Laravel\DependencyInjection\MoonShineConfigurator;
use MoonShine\Laravel\DependencyInjection\ConfiguratorContract;

class MoonShineServiceProvider extends ServiceProvider
{
    public function boot(
        CoreContract $core,
        ConfiguratorContract $config,
    ): void
    {
        $config
            ->title('My Application')
            ->logo('/assets/logo.png')
            ->logo('/assets/logo_small.png', true)
            ->useMigrations()
            ->useNotifications()
            ->useDatabaseNotifications()
            ->dir('app/MoonShine', 'App\MoonShine')
            ->locale('ru')
            ->locales(['en', 'ru'])
            ->guard('moonshine')
            ->layout(\App\MoonShine\Layouts\CustomLayout::class)
            ->disk('public')
            ->cacheDriver('redis')
            ->homeRoute('moonshine.index');

        $core
            ->resources([...])
            ->pages([...]);
    }
}
```

> **Important**: Route-related configuration (`prefix`, `domain`, `page_prefix`, `resource_prefix`) must be set in `config/moonshine.php` because routes are loaded before the service provider's `boot` method runs.

> **Important**: `use_migrations`, `use_notifications`, and `use_database_notifications` must always be present in either `moonshine.php` or `MoonShineServiceProvider`.

## Key configuration areas

### Routing

```php
// config/moonshine.php
'domain' => env('MOONSHINE_DOMAIN'),          // e.g., 'admin.example.com'
'prefix' => env('MOONSHINE_ROUTE_PREFIX', 'admin'),
'page_prefix' => env('MOONSHINE_PAGE_PREFIX', 'page'),
'resource_prefix' => env('MOONSHINE_RESOURCE_PREFIX', 'resource'),
'home_route' => 'moonshine.index',
```

Resulting URL pattern: `/{prefix}/{resource_prefix}/{resourceUri}/{pageUri}`

### Authentication

```php
// config/moonshine.php
'auth' => [
    'enabled' => true,               // set false to disable built-in auth
    'guard' => 'moonshine',          // Laravel auth guard name
    'model' => MoonshineUser::class, // user model (config file only)
    'middleware' => Authenticate::class,
    'pipelines' => [],               // e.g., [TwoFactor::class]
],

'user_fields' => [
    'username' => 'email',
    'password' => 'password',
    'name' => 'name',
    'avatar' => 'avatar',
],
```

To use your own User model, set `auth.model` in `config/moonshine.php` and adjust `user_fields` to match your column names.

### Localization

```php
// config/moonshine.php
'locale' => 'en',
'locales' => ['en', 'ru'],
'locale_key' => '_lang',  // query parameter name for locale switching
```

Language files are stored in `lang/vendor/moonshine/`. The `ChangeLocale` middleware handles locale switching automatically.

### Branding

```php
// In MoonShineServiceProvider
$config
    ->logo('/images/logo.png')
    ->logo('/images/logo-mini.png', small: true);

// Colors via ColorManager
$colors
    ->primary('#2563EB')
    ->secondary('#93C5FD');
```

## Common artisan commands

| Command | Purpose |
|---------|---------|
| `php artisan moonshine:install` | Install MoonShine (run once) |
| `php artisan moonshine:install -Q` | Quick install with defaults |
| `php artisan moonshine:user` | Create an admin user |
| `php artisan moonshine:resource {Name}` | Generate a model resource |
| `php artisan moonshine:page {Name}` | Generate a custom page |
| `php artisan moonshine:layout {Name}` | Generate a layout template |

### Creating your first resource

```shell
php artisan moonshine:resource User
```

This generates a `UserResource` in `app/MoonShine/Resources/` and automatically registers it. The resource is then accessible at:

```
http://127.0.0.1:8000/admin/resource/user-resource/index-page
```

## Replacing default pages and forms

```php
// config/moonshine.php
'pages' => [
    'dashboard' => \App\MoonShine\Pages\DashboardPage::class,
    'profile' => ProfilePage::class,
    'login' => LoginPage::class,
    'error' => ErrorPage::class,
],

'forms' => [
    'login' => LoginForm::class,
    'filters' => FiltersForm::class,
],
```

Retrieve pages and forms programmatically:

```php
$dashboard = moonshineConfig()->getPage('dashboard');
$loginForm = moonshineConfig()->getForm('login');
```

## IDE support

For PhpStorm users, install the [MetaStorm](https://plugins.jetbrains.com/plugin/26121-metastorm/) plugin for improved MoonShine autocompletion and navigation.

## Cross-references

- **moonshine-resources-v3** -- Creating and configuring ModelResource classes, CRUD operations, fields, filters, and actions.
- **moonshine-pages-v3** -- Building custom pages, page components, and layout customization.
- **moonshine-fields-v3** -- Field types, validation, and field configuration.
- **moonshine-components-v3** -- UI components (Grid, Column, Box, LineBreak, etc.).
- **moonshine-menu-v3** -- Menu configuration and navigation structure.
- **moonshine-auth-v3** -- Authentication customization, guards, pipelines, and user management.
- **moonshine-localization-v3** -- Advanced localization, translation files, and multi-language support.
