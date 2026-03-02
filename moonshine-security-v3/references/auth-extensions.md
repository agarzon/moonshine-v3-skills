# Auth Extensions Reference

Detailed setup instructions for MoonShine's official authentication extension packages: Socialite (social login), Two-Factor Authentication, JWT (API tokens), plus patterns for custom login/profile pages, pipeline customization, role-based middleware, and global authorization rules.

---

## Socialite (Social Login)

The `moonshine/socialite` package integrates Laravel Socialite into MoonShine, allowing users to link social network accounts and authenticate through OAuth providers.

### Prerequisites

- [Laravel Socialite](https://laravel.com/docs/socialite) must be installed and configured with at least one driver.

### Installation

```shell
composer require moonshine/socialite
php artisan migrate
php artisan vendor:publish --provider="MoonShine\Socialite\Providers\SocialiteServiceProvider"
```

### Configure drivers

Edit `config/moonshine-socialite.php` to define which OAuth providers are available and their button images:

```php
return [
    'drivers' => [
        'github' => '/images/github.png',
        'facebook' => '/images/facebook.svg',
    ],
];
```

Each driver key must match a driver already configured in Laravel Socialite (via `config/services.php`).

### Add trait to user model

Add the `HasMoonShineSocialite` trait to the model used for MoonShine authentication.

If using the default `MoonshineUser`, publish and extend it:

```php
namespace App\Models;

use MoonShine\Socialite\Traits\HasMoonShineSocialite;

final class MoonshineUser extends \MoonShine\Laravel\Models\MoonshineUser
{
    use HasMoonShineSocialite;
}
```

Then update `config/moonshine.php` to use the custom model:

```php
'auth' => [
    'model' => \App\Models\MoonshineUser::class,
],
```

### UI components

MoonShine automatically adds the `SocialAuth` component to the profile page and `LoginLayout`. If you have overridden those pages, add the component manually:

```php
use MoonShine\Socialite\Components\SocialAuth;

protected function components(): iterable
{
    return [
        // ... other components
        SocialAuth::make(profileMode: true),
    ];
}
```

Set `profileMode: true` when rendering on the profile page (shows link/unlink buttons). Omit it or set `false` for the login page (shows sign-in buttons).

---

## Two-Factor Authentication (2FA)

The `moonshine/two-factor` package adds TOTP-based two-factor authentication via an auth pipeline.

### Installation

```shell
composer require moonshine/two-factor
php artisan migrate
```

### Register the auth pipeline

Add `TwoFactorAuthPipe` to the authentication pipelines:

```php
// config/moonshine.php
use MoonShine\TwoFactor\TwoFactorAuthPipe;

return [
    'auth' => [
        'pipelines' => [
            TwoFactorAuthPipe::class,
        ],
    ],
];
```

Or via `MoonShineServiceProvider`:

```php
use MoonShine\TwoFactor\TwoFactorAuthPipe;

$config->authPipelines([
    TwoFactorAuthPipe::class,
]);
```

### Add trait to user model

Add `TwoFactorAuthenticatable` to your user model:

```php
namespace App\Models;

use MoonShine\TwoFactor\Traits\TwoFactorAuthenticatable;

final class MoonshineUser extends \MoonShine\Laravel\Models\MoonshineUser
{
    use TwoFactorAuthenticatable;
}
```

Update the config to point to your custom model:

```php
'auth' => [
    'model' => App\Models\MoonshineUser::class,
],
```

### Profile component

The `TwoFactor` component is automatically added to the profile page. If you have a custom profile page, add it manually:

```php
use MoonShine\TwoFactor\ComponentSets\TwoFactor;

protected function components(): iterable
{
    return [
        // ... other components
        TwoFactor::make(),
    ];
}
```

### How it works

1. User submits login credentials.
2. `TwoFactorAuthPipe` intercepts the pipeline, checks if the user has 2FA enabled.
3. If enabled, the user is redirected to a TOTP code challenge page.
4. After entering a valid code, authentication completes normally.
5. If 2FA is not enabled for the user, the pipeline passes through to the next step.

### Combining with Socialite

When using both packages, the user model needs both traits:

```php
namespace App\Models;

use MoonShine\Socialite\Traits\HasMoonShineSocialite;
use MoonShine\TwoFactor\Traits\TwoFactorAuthenticatable;

final class MoonshineUser extends \MoonShine\Laravel\Models\MoonshineUser
{
    use HasMoonShineSocialite;
    use TwoFactorAuthenticatable;
}
```

---

## JWT (API Authentication)

MoonShine can operate in API mode with JWT token-based authentication, enabling headless or SPA integrations.

### Installation

```shell
composer require moonshine/jwt
```

### Configuration

After installation, configure the JWT guard and middleware. The package provides an `AuthenticateApi` middleware that replaces the default session-based `Authenticate` middleware for API routes.

Typical setup in `config/moonshine.php`:

```php
'auth' => [
    'guard' => 'moonshine-jwt',
    'middleware' => \MoonShine\JWT\Http\Middleware\AuthenticateApi::class,
],
```

### Usage pattern

- API clients obtain a JWT token by posting credentials to the MoonShine login endpoint.
- Subsequent requests include the token in the `Authorization: Bearer <token>` header.
- The `AuthenticateApi` middleware validates the token and sets the authenticated user on the MoonShine guard.

For full API integration details, see the **moonshine-frontend** skill (API section).

---

## Custom Login Page

Replace the default login page with a custom implementation:

### Via config

```php
// config/moonshine.php
'pages' => [
    'login' => App\MoonShine\Pages\CustomLoginPage::class,
],
```

### Via service provider

```php
$config->changePage(
    \MoonShine\Laravel\Pages\LoginPage::class,
    \App\MoonShine\Pages\CustomLoginPage::class
);
```

### Custom login page structure

A custom login page extends `Page` and uses the `LoginLayout`:

```php
namespace App\MoonShine\Pages;

use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Core\Attributes\Layout;
use MoonShine\Laravel\Layouts\LoginLayout;
use MoonShine\Laravel\Pages\Page;
use MoonShine\MenuManager\Attributes\SkipMenu;

#[SkipMenu]
#[Layout(LoginLayout::class)]
class CustomLoginPage extends Page
{
    /**
     * @return list<ComponentContract>
     */
    protected function components(): iterable
    {
        return [
            // Your custom login form component
            moonshineConfig()->getForm('login'),
        ];
    }
}
```

### Custom login form

Replace the default login form:

```php
// config/moonshine.php
'forms' => [
    'login' => App\MoonShine\Forms\CustomLoginForm::class,
],
```

---

## Custom Profile Page

### Via config

```php
// config/moonshine.php
'pages' => [
    'profile' => App\MoonShine\Pages\CustomProfile::class,
],
```

### Via service provider

```php
$config->changePage(
    \MoonShine\Laravel\Pages\ProfilePage::class,
    \App\MoonShine\Pages\CustomProfile::class
);
```

### Customizing user fields displayed on profile

Map which database columns the profile page reads/writes:

```php
// config/moonshine.php
'user_fields' => [
    'username' => 'email',
    'password' => 'password',
    'name' => 'full_name',
    'avatar' => 'profile_image',
],
```

---

## Authentication Pipeline Customization

### Pipeline execution model

Pipelines use Laravel's `Pipeline` facade. Each pipeline class receives the `Request` and a `$next` closure. Returning `$next($request)` continues the chain. Returning a `RedirectResponse` or `MoonShineJsonResponse` halts login and sends that response to the client.

### Registering pipelines

```php
// config/moonshine.php
'auth' => [
    'pipelines' => [
        \App\MoonShine\AuthPipelines\CheckIpWhitelist::class,
        \MoonShine\TwoFactor\TwoFactorAuthPipe::class,
        \App\MoonShine\AuthPipelines\LogSuccessfulAttempt::class,
    ],
],
```

Or programmatically:

```php
$config->authPipelines([
    \App\MoonShine\AuthPipelines\CheckIpWhitelist::class,
    \MoonShine\TwoFactor\TwoFactorAuthPipe::class,
    \App\MoonShine\AuthPipelines\LogSuccessfulAttempt::class,
]);
```

Pipeline order matters -- they execute sequentially.

### Pipeline class template

```php
namespace App\MoonShine\AuthPipelines;

use Closure;
use Illuminate\Http\Request;

class CheckIpWhitelist
{
    public function handle(Request $request, Closure $next)
    {
        $allowedIps = config('moonshine.allowed_ips', []);

        if (! empty($allowedIps) && ! in_array($request->ip(), $allowedIps, true)) {
            abort(403, 'IP not whitelisted.');
        }

        return $next($request);
    }
}
```

### Common pipeline patterns

| Pattern | Description |
|---------|-------------|
| Redirect to challenge | Store user ID in session, redirect to a challenge page (2FA, SMS) |
| Abort on condition | Call `abort()` to block login entirely (IP whitelist, maintenance) |
| Log and continue | Record an audit entry, then call `$next($request)` |
| Modify request | Add data to the request before passing it along |

---

## Role-Based Middleware Patterns

### Basic role check middleware

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

### Registration

Add to MoonShine's middleware stack in `config/moonshine.php`:

```php
'middleware' => [
    \App\Http\Middleware\CheckAdminRole::class,
],
```

### Multi-role pattern

```php
class CheckPanelAccess
{
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();

        if (! $user || ! $user->hasAnyRole(['admin', 'editor', 'moderator'])) {
            abort(403, 'Access denied.');
        }

        return $next($request);
    }
}
```

### Combining with resource-level policies

Middleware controls access to the entire panel. Policies provide fine-grained per-resource and per-action control. Use both for defense in depth:

1. Middleware ensures only users with panel-access roles can reach any MoonShine route.
2. Policies on individual resources restrict which actions (view, create, edit, delete) each user can perform.

---

## Global Authorization Rules

Add authorization logic that applies across all resources using `authorizationRules()`. This is useful for packages or cross-cutting concerns.

### Registration

```php
use MoonShine\Contracts\Core\ResourceContract;
use MoonShine\Laravel\Enums\Ability;
use Illuminate\Database\Eloquent\Model;

// In MoonShineServiceProvider boot()
$config->authorizationRules(
    static function (ResourceContract $resource, Model $user, Ability $ability, Model $item): bool {
        // Example: deny all delete operations for non-superadmins
        if ($ability === Ability::DELETE && ! $user->is_superadmin) {
            return false;
        }

        return true;
    }
);
```

### How global rules interact with policies

- Global rules registered via `authorizationRules()` run alongside (not instead of) resource-level policies.
- If a resource has `$withPolicy = true`, both the policy and all global rules must return `true` for access to be granted.
- Global rules receive the resource instance, the authenticated user, the ability being checked, and the model instance involved.

### Ability enum values

The `MoonShine\Laravel\Enums\Ability` enum defines all possible abilities:

```php
enum Ability: string
{
    case CREATE = 'create';
    case VIEW = 'view';
    case VIEW_ANY = 'viewAny';
    case UPDATE = 'update';
    case DELETE = 'delete';
    case MASS_DELETE = 'massDelete';
    case RESTORE = 'restore';
    case FORCE_DELETE = 'forceDelete';
}
```

---

## Key Source Files

| File | Purpose |
|------|---------|
| `MoonShine\Laravel\MoonShineAuth` | Static helper for guard, provider, and model access |
| `MoonShine\Laravel\Http\Middleware\Authenticate` | Route protection middleware |
| `MoonShine\Laravel\Http\Controllers\AuthenticateController` | Login, authenticate, logout actions |
| `MoonShine\Laravel\Http\Requests\LoginFormRequest` | Login validation, rate limiting, credential mapping |
| `MoonShine\Laravel\Traits\Resource\ResourceWithAuthorization` | `$withPolicy`, `can()`, `isCan()` on resources |
| `MoonShine\Laravel\Enums\Ability` | Enum of all authorization abilities |
| `MoonShine\Laravel\Pages\LoginPage` | Default login page |
| `MoonShine\Laravel\Traits\Controller\InteractsWithAuth` | Controller trait for guard access |
