# Controllers, Routes & JSON Response Reference

MoonShine v3 controller API, custom route registration, and JSON response helpers.

## MoonShineController Full API

Source: `MoonShine\Laravel\Http\Controllers\MoonShineController`

MoonShineController extends `Illuminate\Routing\Controller` and uses two traits:
- `InteractsWithUI` -- toast and UI helpers
- `InteractsWithAuth` -- authentication helpers

### Constructor

```php
public function __construct(
    protected MoonShineNotificationContract $notification,
) {}
```

### Methods

#### `view(string $path, array $data = []): PageContract`

Renders a Blade view wrapped in the MoonShine layout using `QuickPage`.

```php
final class CustomController extends MoonShineController
{
    public function __invoke(): PageContract
    {
        return $this->view('admin.custom-view', ['key' => 'value']);
    }
}
```

#### `json(string $message, array $data, ?string $redirect, ToastType $messageType, int $status): MoonShineJsonResponse`

Returns a `MoonShineJsonResponse` with toast and optional redirect.

```php
return $this->json(
    message: 'Item saved',
    data: ['id' => 1],
    redirect: '/admin/items',
    messageType: ToastType::SUCCESS,
    status: 200
);
```

#### `toast(string $message, ToastType $type): void`

Triggers a flash toast notification (from `InteractsWithUI` trait).

```php
$this->toast('Hello world', ToastType::SUCCESS);
return back();
```

#### `notification(string $message): void`

Sends a notification to the MoonShine notification center (from `InteractsWithUI` trait).

```php
$this->notification('New order received');
return back();
```

#### `reportAndResponse(bool $isAjax, Throwable $e, ?string $redirectRoute): Response`

Internal error handler. In production, reports the exception and returns a sanitized message. In development, re-throws non-validation exceptions. Handles `ValidationException` specially by including error bags.

#### `responseWithTable(TableBuilderContract $table, ?CrudResourceContract $resource): TableBuilderContract|TableRowContract|string`

Returns the full table or a single row (when `_key` is present in the request). Used internally for async table row updates.

### Generating a Controller

```shell
php artisan moonshine:controller {className?} {--base-dir=} {--base-namespace=}
```

Creates a class in `app/MoonShine/Controllers/`.

---

## Custom Routes

MoonShine uses standard Laravel routing. All pages render through `PageController`.

### Route::moonshine() Directive

```php
Route::moonshine(static function (Router $router) {
    $router->post('permissions/{resourceItem}', PermissionController::class)
        ->name('permissions');
}, withResource: true, withPage: true, withAuthenticate: true);
```

Parameters:
- `withResource` (bool) -- adds `{resourceUri}` prefix
- `withPage` (bool) -- adds `{pageUri}` prefix
- `withAuthenticate` (bool) -- adds `Authenticate::class` middleware

Result: `POST /admin/resource/{resourceUri}/{pageUri}/permissions/{resourceItem}` with `moonshine` + `Authenticate` middleware.

### Standard Route Definition

```php
Route::get('/admin/resource/{resourceUri}/{pageUri}', CustomController::class)
    ->middleware(['moonshine', \MoonShine\Laravel\Http\Middleware\Authenticate::class])
    ->name('moonshine.name');
```

### Route Retrieval

From resource context:
```php
$this->getRoute('permissions')
```

Outside resource:
```php
route('moonshine.permissions', [
    'resourceUri' => 'user-resource',
    'pageUri' => 'custom-page'
])
```

### Best Practice

Create `routes/moonshine.php` and declare custom routes there. Remember to register it in your application's route service provider.

> WARNING: Never use `web` and `moonshine` middleware groups simultaneously -- they both start sessions.

---

## MoonShineJsonResponse

Extends `Illuminate\Http\JsonResponse` with frontend interaction methods.

### Factory

```php
MoonShineJsonResponse::make(data: $optionalData);
```

### toast(string $value, ToastType $type, null|int|false $duration): self

```php
MoonShineJsonResponse::make()->toast('Saved!', ToastType::SUCCESS, duration: 3000);
```

### redirect(string $value): self

```php
MoonShineJsonResponse::make()->redirect('/admin/posts');
```

### events(array $events): self

Triggers JS events after async request processing.

```php
MoonShineJsonResponse::make()->events([
    AlpineJs::event(JsEvent::TABLE_UPDATED, 'index')
]);
```

### html(string|array $value, HtmlMode $mode): self

Inserts HTML into the requesting component's selector.

```php
// Single value into the default selector
MoonShineJsonResponse::make()->html('Content');

// Multiple selectors via array
MoonShineJsonResponse::make()->html([
    '.selector1' => 'Content 1',
    '.selector2' => 'Content 2',
]);
```

HtmlMode enum values: `INNER_HTML`, `OUTER_HTML`, `BEFORE_BEGIN`, `AFTER_BEGIN`, `BEFORE_END`, `AFTER_END`.

### htmlData(string|array $value, string $selector, HtmlMode $mode): self

Insert HTML into a specific selector (can be chained).

```php
MoonShineJsonResponse::make()
    ->htmlData((string) Text::make('One'), '#selector1')
    ->htmlData((string) Text::make('Two'), '#selector2', HtmlMode::BEFORE_END);
```

### fieldsValues(array $values): self

Set field values via CSS selectors. Also triggers a `change` event on each field.

```php
MoonShineJsonResponse::make()->fieldsValues([
    '.field-title-1' => 'some value 1',
    '.field-title-2' => 'some value 2',
]);
```
