---
name: moonshine-advanced-v3
description: Use when working with custom controllers, handlers, routes, type casts, notifications, testing, package development, CrudResource (non-Eloquent), artisan commands, or practical recipe patterns in MoonShine v3.
---

# MoonShine v3 Advanced Topics

## Custom Controllers

MoonShine provides a base `MoonShineController` with helper methods for views, toasts, notifications, and JSON responses. Inheriting from it is optional but convenient.

### Generate a Controller

```shell
php artisan moonshine:controller CustomController
```

Creates a controller in `app/MoonShine/Controllers/`.

### Show a Blade View Inside MoonShine Layout

```php
use MoonShine\Laravel\Http\Controllers\MoonShineController;
use MoonShine\Contracts\Core\PageContract;

final class CustomViewController extends MoonShineController
{
    public function __invoke(): PageContract
    {
        return $this->view('path_to_blade', ['param' => 'value']);
    }
}
```

### Return a MoonShine Page Directly

```php
use App\MoonShine\Pages\MyPage;
use MoonShine\Laravel\Http\Controllers\MoonShineController;

final class CustomViewController extends MoonShineController
{
    public function __invoke(MyPage $page): MyPage
    {
        return $page;
    }
}
```

### Toast, Notification, and JSON Helpers

```php
// Toast notification
$this->toast('Hello world', ToastType::SUCCESS);
return back();

// Send persistent notification to notification center
$this->notification('Message');
return back();

// JSON response with toast + optional redirect
return $this->json(message: 'Saved', data: [], redirect: '/url');
```

### Access Page or Resource from Request

```php
public function __invoke(MoonShineRequest $request)
{
    $page = $request->getPage();
    $resource = $request->getResource();
}
```

> See `moonshine-components-v3` skill for FormBuilder/TableBuilder usage in custom pages.

---

## Handlers

Handlers are reusable action classes that automatically generate UI buttons on resource index pages. They do not require separate controllers.

### Generate a Handler

```shell
php artisan moonshine:handler MyCustomHandler
```

### Handler Structure

```php
use MoonShine\Laravel\Handlers\Handler;
use MoonShine\UI\Components\ActionButton;
use Symfony\Component\HttpFoundation\Response;

class MyCustomHandler extends Handler
{
    public function handle(): Response
    {
        if (! $this->hasResource()) {
            throw new ActionButtonException('Resource is required');
        }

        if ($this->isQueue()) {
            // Dispatch job
            MoonShineUI::toast(__('moonshine::ui.resource.queued'));
            return back();
        }

        // Perform action
        self::process();

        return back();
    }

    public static function process()
    {
        // Your logic here
    }

    public function getButton(): ActionButtonContract
    {
        return ActionButton::make($this->getLabel(), $this->getUrl());
    }
}
```

### Register in a Resource

```php
class PostResource extends ModelResource
{
    protected function handlers(): ListOf
    {
        return parent::handlers()->add(new MyCustomHandler());
    }
}
```

Handler capabilities:
- Access current resource via `$this->getResource()`
- Queue support via `isQueue()` / `WithQueue` trait
- Notification recipients via `notifyUsers(array|Closure $ids)`
- Button customization via `modifyButton(Closure $callback)`

---

## Custom Routes

MoonShine uses standard Laravel routing. The `Route::moonshine()` directive simplifies route registration with proper middleware and parameter prefixes.

```php
// In routes/moonshine.php
Route::moonshine(static function (Router $router) {
    $router->post('permissions/{resourceItem}', PermissionController::class)
        ->name('permissions');
}, withResource: true, withPage: true, withAuthenticate: true);
```

Result: `POST /admin/resource/{resourceUri}/{pageUri}/permissions/{resourceItem}` with `moonshine` + `Authenticate` middleware.

Get route from resource context:
```php
$this->getRoute('permissions')
```

Get route outside resource:
```php
route('moonshine.permissions', ['resourceUri' => 'user-resource', 'pageUri' => 'custom-page'])
```

> WARNING: Do not use `web` and `moonshine` middleware groups simultaneously -- they start sessions at the same time.

---

## MoonShineJsonResponse

Extends `JsonResponse` with frontend interaction helpers:

```php
use MoonShine\Laravel\Http\Responses\MoonShineJsonResponse;

// Toast
MoonShineJsonResponse::make()->toast('Message', ToastType::SUCCESS, duration: 3000);

// Redirect
MoonShineJsonResponse::make()->redirect('/dashboard');

// Trigger JS events
MoonShineJsonResponse::make()->events([AlpineJs::event(JsEvent::TABLE_UPDATED, 'index')]);

// Insert HTML into a selector
MoonShineJsonResponse::make()->html('Content');  // into the requesting component's selector

// Multiple selectors
MoonShineJsonResponse::make()
    ->htmlData((string) Text::make('One'), '#selector1')
    ->htmlData((string) Text::make('Two'), '#selector2', HtmlMode::BEFORE_END);

// Set field values by CSS selector
MoonShineJsonResponse::make()->fieldsValues([
    '.field-title-1' => 'some value 1',
    '.field-title-2' => 'some value 2',
]);
```

---

## CrudResource (Non-Eloquent Data)

`CrudResource` lets you work with any data source -- APIs, files, custom databases -- without Eloquent.

> Cross-ref: `moonshine-resources-v3` skill for ModelResource (Eloquent-based).

### Abstract Methods to Implement

```php
use MoonShine\Laravel\Resources\CrudResource;

final class RestCrudResource extends CrudResource
{
    public function findItem(bool $orFail = false): mixed { /* ... */ }
    public function getItems(): mixed { /* ... */ }
    public function massDelete(array $ids): void { /* ... */ }
    public function delete(mixed $item, ?FieldsContract $fields = null): bool { /* ... */ }
    public function save(mixed $item, ?FieldsContract $fields = null): mixed { /* ... */ }
}
```

### REST API Example

```php
final class RestCrudResource extends CrudResource
{
    public function getItems(): iterable
    {
        yield from Http::get('https://jsonplaceholder.typicode.com/todos')->json();
    }

    public function findItem(bool $orFail = false): array
    {
        yield from Http::get('https://jsonplaceholder.typicode.com/todos/' . $this->getItemID())->json();
    }

    public function save(mixed $item, ?FieldsContract $fields = null): mixed
    {
        $data = request()->all();
        if ($item['id'] ?? false) {
            return Http::put('https://api.example.com/todos/' . $item['id'], $data)->json();
        }
        $this->isRecentlyCreated = true;
        return Http::post('https://api.example.com/todos', $data)->json();
    }

    public function delete(mixed $item, ?FieldsContract $fields = null): bool
    {
        return Http::delete('https://api.example.com/todos/' . $item['id'])->successful();
    }

    public function massDelete(array $ids): void
    {
        $this->beforeMassDeleting($ids);
        foreach ($ids as $id) {
            $this->delete(['id' => $id]);
        }
        $this->afterMassDeleted($ids);
    }
}
```

For maximum flexibility, implement `CrudResourceContract` directly.

---

## Type Casts

Fields work with primitive types by default. TypeCasts bridge typed data (models, DTOs) to MoonShine components.

Implement `DataCasterContract` and `DataWrapperContract`:

```php
interface DataCasterContract
{
    public function cast(mixed $data): DataWrapperContract;
    public function paginatorCast(mixed $data): ?PaginatorContract;
}

interface DataWrapperContract
{
    public function getOriginal(): mixed;
    public function getKey(): int|string|null;
    public function toArray(): array;
}
```

Usage with FormBuilder/TableBuilder:

```php
TableBuilder::make(items: User::paginate())
    ->fields([Text::make('Email')])
    ->cast(new ModelCaster(User::class));

FormBuilder::make()
    ->fields([Text::make('Email')])
    ->fillCast(User::query()->first(), new ModelCaster(User::class));
```

Generate a custom TypeCast:
```shell
php artisan moonshine:type-cast MyCustomCaster
```

---

## Testing Quick Start

### Generate Tests with Resources

```shell
php artisan moonshine:resource PostResource --test   # PHPUnit
php artisan moonshine:resource PostResource --pest   # Pest
```

### Test Setup

```php
protected function setUp(): void
{
    parent::setUp();
    $user = MoonshineUser::factory()->create();
    $this->be($user, 'moonshine');
}

public function test_index_page_successful(): void
{
    $response = $this->get(
        $this->getResource()->getIndexPageUrl()
    )->assertSuccessful();
}
```

> Cross-ref: `moonshine-setup-v3` skill for all artisan commands including `--test`/`--pest` flags.

---

## Notifications and Toasts

### Notifications (Notification Center)

```php
use MoonShine\Laravel\Notifications\MoonShineNotification;
use MoonShine\Laravel\Notifications\NotificationButton;
use MoonShine\Support\Enums\Color;

MoonShineNotification::send(
    message: 'Notification text',
    button: new NotificationButton('Click me', 'https://example.com'),
    ids: [1, 2, 3],       // admin user IDs (null = all)
    color: Color::GREEN,
    icon: 'information-circle'
);
```

### Toast Notifications

```php
use MoonShine\Laravel\MoonShineUI;
use MoonShine\Support\Enums\ToastType;

MoonShineUI::toast('Success', ToastType::SUCCESS, duration: 3000);
MoonShineUI::toast('Sticky toast', duration: false);  // stays until clicked
```

---

## Common Recipes Index

Practical patterns for common MoonShine tasks. Full code in `references/recipes-dashboard.md`, `references/recipes-resources.md`, `references/recipes-forms-tables.md`, and `references/recipes-ui-other.md`.

| Category | Recipe |
|----------|--------|
| Dashboard | Async metrics with date filters, Dashboard settings form |
| Resources | Reorderable rows, Soft deletes, Index page as cards |
| Forms | Form events (table refresh + reset), Fields group via Template, Mass edit modal, HasMany with parent ID |
| Tables | Custom paginator for TableBuilder, updateOnPreview with pivot |
| UI | Custom breadcrumbs, Relationship fields in tabs, Template field for config files |
| Menu | Conditional menu items (Gate, Policy, role checks) |
| Select | Async options, Reactive selects, ShowWhen, onChangeMethod, Fragments |
| Other | Async remove on click (Image/Json), Save to config file, Multiple fragments + selectors |

---

## Cross-References

- **moonshine-resources-v3**: ModelResource, query scopes, filters, actions -- the foundation that CrudResource and Handlers extend.
- **moonshine-setup-v3**: Full artisan command reference (`moonshine:install`, `moonshine:resource`, `moonshine:controller`, `moonshine:handler`, `moonshine:type-cast`, `moonshine:apply`, etc.).
- **moonshine-components-v3**: FormBuilder, TableBuilder, CardsBuilder, Fragment, ActionButton -- used extensively in custom pages and controllers.
