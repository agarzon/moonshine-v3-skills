# API Mode, JWT Authentication & OpenAPI Generator

## API Mode Setup

MoonShine can serve as a pure JSON API backend. Add `Accept: application/json` to any request header, and CRUD operations will return JSON responses instead of HTML.

### Available CRUD Routes

```
GET    /admin/resource/{resourceUri}/crud                    -- Listing (paginated)
POST   /admin/resource/{resourceUri}/crud                    -- Create record
PUT    /admin/resource/{resourceUri}/crud/{resourceItem}     -- Update record
DELETE /admin/resource/{resourceUri}/crud/{resourceItem}     -- Delete record
DELETE /admin/resource/{resourceUri}/crud                    -- Mass delete (ids[] in body)
```

The `{resourceUri}` corresponds to the resource's URI segment (e.g., `post-resource` for `PostResource`).

### Disabling Session Middleware

When running MoonShine entirely in API mode (no browser-based sessions), remove session-related middleware from the configuration:

```php
// config/moonshine.php
return [
    'middleware' => [
        // Remove session/cookie middleware for pure API usage
    ],
    // ...
];
```

This prevents MoonShine from attempting to create sessions for stateless API requests.

### JSON Response Structure

Standard CRUD responses include:

```json
{
    "message": "Success message",
    "messageType": "success",
    "redirect": "/admin/resource/post-resource/crud",
    "events": "table_updated:index",
    "htmlData": [],
    "fields_values": {}
}
```

Key fields:
- `message` -- Toast message text.
- `messageType` -- One of `default`, `success`, `error`, `warning`, `info`.
- `redirect` -- URL to redirect after the operation.
- `events` -- Comma-separated event strings to dispatch on the client.
- `htmlData` -- Array of `{html, selector, htmlMode}` for DOM updates.
- `fields_values` -- Key-value pairs for updating specific form field values.

### Custom JSON Responses

Use `MoonShineJsonResponse` in resource methods or controllers:

```php
use MoonShine\Laravel\Http\Responses\MoonShineJsonResponse;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\Support\Enums\ToastType;

return MoonShineJsonResponse::make()
    ->toast('Record saved', ToastType::SUCCESS)
    ->redirect('/admin/dashboard')
    ->events([
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'index'),
    ]);
```

Available `MoonShineJsonResponse` methods:

| Method | Description |
|---|---|
| `toast(string $value, ToastType $type, null\|int\|false $duration)` | Set toast message and type |
| `redirect(string $value)` | Set redirect URL |
| `events(array $events)` | Set events to dispatch |
| `html(string\|array $value, HtmlMode $mode)` | Set HTML content for DOM update |
| `htmlData(string $value, ?string $selector, HtmlMode $mode)` | Add HTML data entry with selector |
| `fieldsValues(array $value)` | Set field values to update |

### Response Modifiers on Resources

Override response construction for CRUD operations at the resource level:

```php
class PostResource extends ModelResource
{
    public function modifySaveResponse(MoonShineJsonResponse $response): MoonShineJsonResponse
    {
        return $response->toast('Post saved!', ToastType::SUCCESS);
    }

    public function modifyDestroyResponse(MoonShineJsonResponse $response): MoonShineJsonResponse
    {
        return $response->toast('Post deleted', ToastType::INFO);
    }

    public function modifyMassDeleteResponse(MoonShineJsonResponse $response): MoonShineJsonResponse
    {
        return $response;
    }
}
```

---

## JWT Authentication

The `moonshine/jwt` package replaces session-based authentication with stateless JWT tokens for API usage.

### Installation

```shell
composer require moonshine/jwt
```

### Publish Configuration

```shell
php artisan vendor:publish --provider="MoonShine\JWT\Providers\JWTServiceProvider"
```

### Set the JWT Secret

Add a base64-encoded secret key to your `.env` file:

```ini
JWT_SECRET=YOUR_BASE64_SECRET_HERE
```

Generate a base64 secret (example):

```shell
php -r "echo base64_encode(random_bytes(32));"
```

### Configure MoonShine for JWT

Replace the default session-based middleware and add the JWT auth pipeline:

```php
// config/moonshine.php
use MoonShine\JWT\JWTAuthPipe;
use MoonShine\JWT\Http\Middleware\AuthenticateApi;

return [
    'middleware' => [
        // No session middleware for API mode
    ],
    'auth' => [
        'enabled' => true,
        'guard' => 'moonshine',
        'model' => \MoonShine\Laravel\Models\MoonshineUser::class,
        'middleware' => AuthenticateApi::class,
        'pipelines' => [
            JWTAuthPipe::class,
        ],
    ],
    // ...
];
```

### Authentication Flow

1. **Login** -- POST credentials to the MoonShine login endpoint with `Accept: application/json`.
2. **Receive token** -- On successful authentication, the response includes a JWT token.
3. **Use token** -- Include the token in subsequent requests via the `Authorization` header:

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
```

### How JWT Middleware Works

- `AuthenticateApi` replaces the default `Authenticate` middleware. Instead of checking for a session, it validates the JWT token from the `Authorization` header.
- `JWTAuthPipe` is an auth pipeline that runs during the login process, generating and returning the JWT token upon successful credential validation.

### Combining JWT with Other Pipelines

JWT can be combined with other auth pipelines (e.g., 2FA):

```php
'auth' => [
    'middleware' => AuthenticateApi::class,
    'pipelines' => [
        JWTAuthPipe::class,
        TwoFactorAuthPipe::class, // Additional pipeline
    ],
],
```

---

## OpenAPI Generator (moonshine/oag)

The `moonshine/oag` package generates OpenAPI specifications automatically from your declared MoonShine resources.

### Installation

```shell
composer require moonshine/oag
```

### Publish Configuration

```shell
php artisan vendor:publish --provider="MoonShine\OAG\Providers\OAGServiceProvider"
```

### Configuration

```php
// config/oag.php
return [
    // Documentation title
    'title' => 'Docs',

    // Location path for the specification file
    'path' => realpath(resource_path('oag.yaml')),

    // Route name for the JSON specification endpoint
    'route' => 'oag.json',

    // View used for rendering documentation
    'view' => 'oag::docs',
];
```

### Generate Specification

Run the artisan command to generate the specification based on all declared resources:

```shell
php artisan oag:generate
```

This creates two specification files in the `resources` directory:
- `resources/oag.yaml` -- YAML format
- `resources/oag.json` -- JSON format

### Access Documentation

After generation, the interactive API documentation is available at:

```
GET /docs
```

### What Gets Generated

The generator inspects each registered MoonShine resource and produces OpenAPI paths for:
- Listing endpoints (GET with pagination parameters)
- Create endpoints (POST with field schemas)
- Update endpoints (PUT with field schemas)
- Delete endpoints (DELETE)
- Mass delete endpoints (DELETE with ids[] parameter)

Field types from the resource's `fields()` method are mapped to appropriate OpenAPI schema types.

---

## Complete API Mode Example

A minimal setup for running MoonShine as a pure API backend with JWT auth and OpenAPI docs:

```shell
# Install packages
composer require moonshine/jwt moonshine/oag

# Publish configs
php artisan vendor:publish --provider="MoonShine\JWT\Providers\JWTServiceProvider"
php artisan vendor:publish --provider="MoonShine\OAG\Providers\OAGServiceProvider"
```

```ini
# .env
JWT_SECRET=YOUR_BASE64_SECRET_HERE
```

```php
// config/moonshine.php
use MoonShine\JWT\JWTAuthPipe;
use MoonShine\JWT\Http\Middleware\AuthenticateApi;

return [
    'middleware' => [],
    'auth' => [
        'enabled' => true,
        'guard' => 'moonshine',
        'model' => \MoonShine\Laravel\Models\MoonshineUser::class,
        'middleware' => AuthenticateApi::class,
        'pipelines' => [
            JWTAuthPipe::class,
        ],
    ],
    // ... rest of config
];
```

```shell
# Generate OpenAPI docs
php artisan oag:generate
```

API consumers can then:
1. `POST /admin/authenticate` with `Accept: application/json` to get a JWT token.
2. Use `Authorization: Bearer <token>` and `Accept: application/json` on all subsequent CRUD requests.
3. Reference `/docs` for interactive API documentation.
