# Authentication, Localization & Configurator Reference

MoonShine v3 authentication configuration, user field mapping, layout/pages/forms, localization, configuration priority, and MoonShineConfigurator method reference.

---

## Table of Contents

- [Authentication](#authentication)
- [User field mapping](#user-field-mapping)
- [Layout, pages, and forms](#layout-pages-and-forms)
- [Localization](#localization)
- [Configuration priority](#configuration-priority)
- [MoonShineConfigurator methods](#moonshineconfigurator-methods)

---

## Authentication

**Config key:** `auth`

### auth.enabled

**Default:** `true`

Enables or disables MoonShine's built-in authentication system entirely. When set to `false`, no authentication middleware is applied and no login page is shown.

```php
'auth' => [
    'enabled' => false,
],
```

### auth.guard

**Default:** `'moonshine'`
**Provider method:** `$config->guard('moonshine')`

The Laravel authentication guard used by MoonShine. MoonShine registers its own `moonshine` guard automatically when using the default `MoonshineUser` model.

```php
'auth' => [
    'guard' => 'admin',
],
```

### auth.model

**Default:** `MoonshineUser::class`
**Provider method:** N/A (config file only)

The Eloquent model used for authentication. This setting is read during application bootstrapping, so it must be defined in the config file, not in the service provider.

```php
'auth' => [
    'model' => \App\Models\User::class,
],
```

> **Note**: When changing the model, you will likely also need to update `user_fields` to match your model's column names, and configure the guard in `config/auth.php`.

### auth.middleware

**Default:** `Authenticate::class`

The middleware class that checks for a valid authenticated session. Applied to all MoonShine routes when `auth.enabled` is `true`.

### auth.pipelines

**Default:** `[]`
**Provider method:** `$config->authPipelines([...])`

Authentication pipelines allow you to add extra steps to the authentication flow (e.g., two-factor authentication).

```php
'auth' => [
    'pipelines' => [
        \App\MoonShine\Pipelines\TwoFactor::class,
    ],
],
```

```php
// MoonShineServiceProvider
$config->authPipelines([\App\MoonShine\Pipelines\TwoFactor::class]);
```

### Authorization rules

**Provider method only:**

```php
$config->authorizationRules(
    function(ResourceContract $ctx, mixed $user, Ability $ability, mixed $data): bool {
        return true;
    }
);
```

Define custom authorization logic applied across all resources.

---

## User field mapping

**Config key:** `user_fields`

Maps MoonShine's internal user field names to actual database columns on your user model.

| Key | Default | Description |
|-----|---------|-------------|
| `username` | `'email'` | The field used as the login identifier |
| `password` | `'password'` | The password column |
| `name` | `'name'` | The display name field |
| `avatar` | `'avatar'` | The avatar image field |

```php
'user_fields' => [
    'username' => 'email',
    'password' => 'password',
    'name' => 'name',
    'avatar' => 'avatar',
],
```

If your custom model uses different column names:

```php
'user_fields' => [
    'username' => 'login',
    'password' => 'pass_hash',
    'name' => 'full_name',
    'avatar' => 'profile_photo',
],
```

Provider equivalent (per-field):

```php
$config->userField('username', 'login');
$config->userField('name', 'full_name');
```

---

## Layout, pages, and forms

### layout

**Config key:** `layout`
**Default:** `AppLayout::class` (`MoonShine\Laravel\Layouts\AppLayout`)
**Provider method:** `$config->layout(\App\MoonShine\Layouts\CustomLayout::class)`

The default layout class used for all MoonShine pages. Override this to customize the admin panel structure (sidebar, header, footer, etc.).

```php
'layout' => \App\MoonShine\Layouts\CustomLayout::class,
```

### pages

**Config key:** `pages`

Maps page names to their implementing classes. MoonShine uses these internally to render the dashboard, profile, login, and error pages.

```php
'pages' => [
    'dashboard' => \App\MoonShine\Pages\Dashboard::class,
    'profile'   => \MoonShine\Laravel\Pages\ProfilePage::class,
    'login'     => \MoonShine\Laravel\Pages\LoginPage::class,
    'error'     => \MoonShine\Laravel\Pages\ErrorPage::class,
],
```

To swap a page with your own implementation:

```php
// config/moonshine.php
'pages' => [
    'login' => \App\MoonShine\Pages\MyLoginPage::class,
],
```

```php
// MoonShineServiceProvider
$config->changePage(LoginPage::class, MyLoginPage::class);
```

Retrieve a page instance at runtime:

```php
$page = moonshineConfig()->getPage('dashboard');

// With DI
public function index(ConfiguratorContract $config)
{
    $page = $config->getPage('dashboard');
}
```

### forms

**Config key:** `forms`

Maps form names to their implementing classes.

```php
'forms' => [
    'login'   => \MoonShine\Laravel\Forms\LoginForm::class,
    'filters' => \MoonShine\Laravel\Forms\FiltersForm::class,
],
```

To replace with a custom form:

```php
// config/moonshine.php
'forms' => [
    'login' => \App\MoonShine\Forms\MyLoginForm::class,
],
```

```php
// MoonShineServiceProvider
$config->set('forms.login', MyLoginForm::class);
```

Retrieve a form instance at runtime:

```php
$form = moonshineConfig()->getForm('login');
```

---

## Localization

### locale

**Config key:** `locale`
**Default:** `'en'`
**Provider method:** `$config->locale('en')`

The default language for the admin panel interface.

### locales

**Config key:** `locales`
**Default:** `[]` (empty, only default locale available)
**Provider method:** `$config->locales(['en', 'ru'])`

List of available locales. When multiple locales are configured, MoonShine displays a language switcher in the admin panel.

```php
'locales' => ['en', 'ru', 'de', 'fr'],
```

### locale_key

**Config key:** `locale_key`
**Default:** `ChangeLocale::KEY` (resolves to `'_lang'`)
**Provider method:** `$config->localeKey('_lang')`

The query string parameter name used for locale switching. When a user switches locales, this parameter is appended to the URL.

### Language files

Translation files should be placed in `lang/vendor/moonshine/`. The structure follows Laravel's standard vendor translation layout:

```
lang/
  vendor/
    moonshine/
      en/
        ui.php
        validation.php
      ru/
        ui.php
        validation.php
```

### ChangeLocale middleware

The `MoonShine\Laravel\Http\Middleware\ChangeLocale` middleware is included in the default middleware stack. It:

1. Reads the locale from the `locale_key` query parameter.
2. Validates it against the configured `locales` list.
3. Stores the selected locale in the session.
4. Applies the locale for the current request.

No additional setup is needed for locale switching to work.

---

## Configuration priority

When the same setting is defined in both `config/moonshine.php` and `MoonShineServiceProvider`, the service provider value takes precedence.

**Recommended approach:**

| Setting type | Where to configure |
|-------------|-------------------|
| Route settings (`prefix`, `domain`, `page_prefix`, `resource_prefix`) | `config/moonshine.php` only (loaded before boot) |
| `auth.model` | `config/moonshine.php` only (needed at bootstrap) |
| Feature flags (`use_migrations`, etc.) | Either location |
| Dynamic settings (conditional logic, environment-based) | `MoonShineServiceProvider` |
| Simple static values | `config/moonshine.php` |

You can combine both approaches: use the config file for basic settings and the service provider for complex, programmatic configuration.

---

## MoonShineConfigurator methods

Quick reference of all `MoonShineConfigurator` methods available in `MoonShineServiceProvider`:

| Method | Description |
|--------|-------------|
| `title(string $title)` | Set the page `<title>` |
| `logo(string $path, bool $small = false)` | Set logo (call twice for full and small) |
| `dir(string $dir, string $namespace)` | Set MoonShine directory and namespace |
| `useMigrations()` | Enable built-in migrations |
| `useNotifications()` | Enable notification system |
| `useDatabaseNotifications()` | Enable database notification driver |
| `guard(string $guard)` | Set the auth guard |
| `authPipelines(array $pipelines)` | Set authentication pipelines |
| `authorizationRules(Closure $rules)` | Define global authorization rules |
| `layout(string $class)` | Set the default layout class |
| `locale(string $locale)` | Set the default locale |
| `locales(array $locales)` | Set available locales |
| `localeKey(string $key)` | Set the locale query parameter name |
| `disk(string $disk, array $options = [])` | Set the filesystem disk |
| `cacheDriver(string $driver)` | Set the cache driver |
| `homeRoute(string $route)` | Set the home page by route name |
| `homeUrl(string $url)` | Set the home page by URL |
| `notFoundException(string $class)` | Set the 404 exception class |
| `changePage(string $old, string $new)` | Replace a default page class |
| `set(string $key, mixed $value)` | Set an arbitrary config value |
| `getPage(string $name, ...)` | Get a page instance by config name |
| `getForm(string $name, ...)` | Get a form instance by config name |
| `userField(string $field, string $column)` | Map a user field to a column |
