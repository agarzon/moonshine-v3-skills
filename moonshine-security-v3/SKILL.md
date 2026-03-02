---
name: moonshine-security-v3
description: Use when working with MoonShine v3 authentication, authorization, login, guards, custom user models, Socialite, 2FA, JWT, role-based access, Laravel policies, auth pipelines, or middleware.
---

# MoonShine v3 -- Security (Authentication & Authorization)

## Overview

MoonShine ships with a self-contained authentication system backed by its own guard, user model, and middleware. Authorization is handled through standard Laravel Policies, enabled per-resource with `$withPolicy`. The auth pipeline mechanism lets you inject additional steps (2FA, phone verification) into the login flow without modifying core code.

## Built-in Authentication System

### How it works

MoonShine registers a dedicated `moonshine` auth guard with its own Eloquent user provider. The `MoonShineAuth` helper class exposes static methods to retrieve the guard, provider, and model at runtime:

```php
use MoonShine\Laravel\MoonShineAuth;

// Get the active guard instance
$guard = MoonShineAuth::getGuard();

// Get the guard name (string)
$guardName = MoonShineAuth::getGuardName(); // 'moonshine'

// Get a fresh instance of the configured user model
$userModel = MoonShineAuth::getModel();
```

The `Authenticate` middleware checks `moonshineConfig()->isAuthEnabled()` and, if enabled, verifies the user is authenticated via the MoonShine guard. Unauthenticated requests are redirected to the login page.

### Default configuration

```php
// config/moonshine.php
'auth' => [
    'enabled' => true,
    'guard' => 'moonshine',
    'model' => MoonshineUser::class,
    'middleware' => Authenticate::class,
    'pipelines' => [],
],

'user_fields' => [
    'username' => 'email',
    'password' => 'password',
    'name' => 'name',
    'avatar' => 'avatar',
],
```

- `enabled` -- toggle the entire auth system on or off.
- `guard` -- the Laravel guard name MoonShine uses.
- `model` -- Eloquent model representing admin users.
- `middleware` -- the middleware class protecting MoonShine routes.
- `pipelines` -- ordered list of pipeline classes that run during login (see below).

### Disabling authentication

Set `enabled` to `false` to remove all auth checks from the panel:

```php
'auth' => [
    'enabled' => false,
],
```

## Custom User Model

Replace the default `MoonshineUser` with your own model by updating the config:

```php
// config/moonshine.php
'auth' => [
    'model' => App\Models\Admin::class,
],
```

### Custom user fields

When your model uses different column names, map them in `user_fields`:

```php
// config/moonshine.php
'user_fields' => [
    'username' => 'login',
    'password' => 'pass',
    'name' => 'full_name',
    'avatar' => 'profile_image',
],
```

Or configure programmatically in `MoonShineServiceProvider`:

```php
$config
    ->userField('username', 'login')
    ->userField('password', 'pass')
    ->userField('name', 'full_name')
    ->userField('avatar', 'profile_image');
```

### Custom profile page

Replace the built-in profile page:

```php
// config/moonshine.php
'pages' => [
    'profile' => App\MoonShine\Pages\CustomProfile::class,
],
```

Or via the service provider:

```php
$config->changePage(
    \MoonShine\Laravel\Pages\ProfilePage::class,
    \App\MoonShine\Pages\CustomProfile::class
);
```

### Custom login page

Replace the built-in login page:

```php
// config/moonshine.php
'pages' => [
    'login' => App\MoonShine\Pages\CustomLogin::class,
],
```

## Authorization with Laravel Policies

MoonShine uses standard Laravel Policies. By default, policy checks are disabled on resources. Enable them by setting `$withPolicy = true`.

### Enabling policies on a resource

```php
namespace App\MoonShine\Resources;

use MoonShine\Laravel\Resources\ModelResource;

class PostResource extends ModelResource
{
    protected bool $withPolicy = true;

    // ...
}
```

### Available policy methods

| Policy method | Controls | MoonShine Ability enum |
|---------------|----------|----------------------|
| `viewAny` | Index page | `Ability::VIEW_ANY` |
| `view` | Detail page | `Ability::VIEW` |
| `create` | Creating a record | `Ability::CREATE` |
| `update` | Editing a record | `Ability::UPDATE` |
| `delete` | Deleting a record | `Ability::DELETE` |
| `massDelete` | Mass deletion | `Ability::MASS_DELETE` |
| `restore` | Restoring soft-deleted record | `Ability::RESTORE` |
| `forceDelete` | Permanent deletion | `Ability::FORCE_DELETE` |

### Writing a policy

```php
namespace App\Policies;

use App\Models\Post;
use Illuminate\Auth\Access\HandlesAuthorization;
use MoonShine\Laravel\Models\MoonshineUser;

class PostPolicy
{
    use HandlesAuthorization;

    public function viewAny(MoonshineUser $user)
    {
        return true;
    }

    public function view(MoonshineUser $user, Post $model)
    {
        return true;
    }

    public function create(MoonshineUser $user)
    {
        return true;
    }

    public function update(MoonshineUser $user, Post $model)
    {
        return true;
    }

    public function delete(MoonshineUser $user, Post $model)
    {
        return true;
    }

    public function restore(MoonshineUser $user, Post $model)
    {
        return true;
    }

    public function forceDelete(MoonshineUser $user, Post $model)
    {
        return true;
    }

    public function massDelete(MoonshineUser $user)
    {
        return true;
    }
}
```

### Generating a policy

```shell
php artisan moonshine:policy PostPolicy
```

This creates a policy class in `app/Policies/` with all MoonShine-compatible methods pre-defined.

### Registering policies for non-app models

Laravel auto-discovers policies only for models in `app/Models`. For MoonShine's own models (or any model outside that directory), register them manually:

```php
use Illuminate\Support\Facades\Gate;

// In AuthServiceProvider or MoonShineServiceProvider boot()
Gate::policy(MoonshineUser::class, MoonshineUserPolicy::class);
Gate::policy(MoonshineUserRole::class, MoonshineUserRolePolicy::class);
```

### Custom authorization logic (isCan)

Override `isCan()` in your resource to supplement or replace policy checks:

```php
use MoonShine\Laravel\Enums\Ability;

class PostResource extends ModelResource
{
    protected bool $withPolicy = true;

    protected function isCan(Ability $ability): bool
    {
        // Custom logic alongside the policy
        return parent::isCan($ability);
    }
}
```

### Global authorization rules

Add authorization rules that apply across all resources:

```php
use MoonShine\Contracts\Core\ResourceContract;
use MoonShine\Laravel\Enums\Ability;
use Illuminate\Database\Eloquent\Model;

// In MoonShineServiceProvider boot()
$config->authorizationRules(
    static function (ResourceContract $resource, Model $user, Ability $ability, Model $item): bool {
        return true;
    }
);
```

## Authentication Pipelines

Pipelines run after username/password validation but before the session is created. They can redirect users (e.g., to a 2FA challenge page) or add custom checks.

### How the pipeline executes

The `AuthenticateController::authenticate()` method sends the request through all configured pipelines using Laravel's `Pipeline` facade:

```php
// Simplified from AuthenticateController
if (filled(moonshineConfig()->getAuthPipelines())) {
    $request = Pipeline::send($request)->through(
        moonshineConfig()->getAuthPipelines()
    )->thenReturn();
}
```

If a pipeline returns a `RedirectResponse` or `MoonShineJsonResponse`, the login flow stops and that response is returned. Otherwise, authentication proceeds normally.

### Configuring pipelines

```php
// config/moonshine.php
'auth' => [
    'pipelines' => [
        TwoFactorAuthentication::class,
        PhoneVerification::class,
    ],
],
```

Or in `MoonShineServiceProvider`:

```php
$config->authPipelines([
    \App\MoonShine\AuthPipelines\TwoFactorAuthentication::class,
    \App\MoonShine\AuthPipelines\PhoneVerification::class,
]);
```

### Writing a custom pipeline

A pipeline class must implement a `handle` method that receives the request and a `$next` closure:

```php
namespace App\MoonShine\AuthPipelines;

use Closure;
use Illuminate\Http\Request;
use MoonShine\Laravel\Models\MoonshineUser;

class PhoneVerification
{
    public function handle(Request $request, Closure $next)
    {
        $user = MoonshineUser::query()
            ->where('email', $request->get('username'))
            ->first();

        if (! is_null($user) && ! is_null($user->getAttribute('phone'))) {
            $request->session()->put([
                'login.id' => $user->getKey(),
                'login.remember' => $request->boolean('remember'),
            ]);

            return redirect()->route('sms-challenge');
        }

        return $next($request);
    }
}
```

## Role-Based Access via Middleware

Create middleware to restrict panel access by role or other user attributes:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAdminRole
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->hasRole('admin')) {
            abort(403, 'Access denied.');
        }

        return $next($request);
    }
}
```

Register the middleware in `config/moonshine.php`:

```php
'middleware' => [
    // ... other middleware
    \App\Http\Middleware\CheckAdminRole::class,
],
```

## Extension Packages (Quick Reference)

MoonShine provides official packages for common auth extensions. For full setup instructions, see [references/auth-extensions.md](references/auth-extensions.md).

### Socialite (Social Login)

```shell
composer require moonshine/socialite
php artisan migrate
php artisan vendor:publish --provider="MoonShine\Socialite\Providers\SocialiteServiceProvider"
```

Configure drivers in `config/moonshine-socialite.php` and add `HasMoonShineSocialite` trait to your user model.

### Two-Factor Authentication (2FA)

```shell
composer require moonshine/two-factor
php artisan migrate
```

Add `TwoFactorAuthPipe` to the auth pipelines and `TwoFactorAuthenticatable` trait to your user model.

### JWT (API Authentication)

MoonShine can run in API mode with token-based authentication. Install the JWT package and configure the `AuthenticateApi` middleware. See [references/auth-extensions.md](references/auth-extensions.md) for details.

## Login Flow Summary

1. User submits credentials to `AuthenticateController::authenticate()`.
2. `LoginFormRequest` validates `username` and `password` fields, applies rate limiting.
3. If auth pipelines are configured, the request passes through each one in order.
4. If a pipeline returns a redirect or JSON response, the flow stops (e.g., 2FA challenge).
5. Otherwise, `LoginFormRequest::authenticate()` calls `MoonShineAuth::getGuard()->attempt()`.
6. On success, the session is regenerated and the user is redirected to the home route.
7. On failure, a `ValidationException` is thrown with rate-limiter tracking.

## Cross-references

- **moonshine-setup-v3** -- Auth configuration keys in `config/moonshine.php`, guard setup, user field mapping.
- **moonshine-resources-v3** -- Resource-level authorization with `$withPolicy`, `isCan()`, and `getGateAbilities()`.
- **moonshine-frontend-v3** -- JWT-based API authentication for headless/SPA use cases.
