# Field Interactive Features Reference

Conditional display, showWhen, inline editing, onChange, reactivity, and custom field creation for MoonShine v3.

## Conditional Methods

### canSee

```php
->canSee(Closure $callback)
```

```php
Text::make('Name')->canSee(fn(Text $field) => $field->toValue() !== 'hide')

BelongsTo::make('Item', resource: ItemResource::class)
    ->canSee(fn(Comment $comment, BelongsTo $field) => $comment->is_active)
```

### when / unless

```php
->when($value, ?callable $callback, ?callable $default)
->unless($value, ?callable $callback, ?callable $default)
```

```php
Text::make('Slug')->when(fn() => true, fn(Field $field) => $field->locked())
```

---

## Dynamic Display (showWhen)

Show/hide fields based on other field values without page reload.

```php
->showWhen(string $column, mixed $operator = null, mixed $value = null)
->showWhenDate(string $column, mixed $operator = null, mixed $value = null)
```

```php
Text::make('Name')->showWhen('category_id', 1)              // show when category_id = 1
Text::make('Name')->showWhen('category_id', 'in', [1, 2])   // show when in list
Text::make('Content')->showWhenDate('created_at', '>', '2024-09-15')
```

Supported operators: `=`, `!=`, `>`, `<`, `>=`, `<=`, `in`, `not in`.

Multiple conditions (AND logic):
```php
BelongsTo::make('Category')
    ->showWhenDate('created_at', '>', '2024-08-05 10:00')
    ->showWhenDate('created_at', '<', '2024-08-05 19:00')
```

Nested fields (Json):
```php
Text::make('Parts')->showWhen('attributes.1.size', '!=', 2)
```

To keep hidden field values in form submission:
```php
class ArticleResource extends ModelResource
{
    protected bool $submitShowWhen = true;
}
```

---

## Inline Editing (updateOnPreview)

Available for: `Text`, `Number`, `Checkbox`, `Select`, `Date`.

### updateOnPreview

Edit field values directly in table cells:

```php
->updateOnPreview(?Closure $url = null, ?ResourceContract $resource = null, mixed $condition = null, array $events = [])
```

```php
Text::make('Name')->updateOnPreview()
Select::make('Status')->updateOnPreview()
```

### withUpdateRow

Update the entire table row after editing:

```php
->withUpdateRow(string $component)
```

```php
Text::make('Name')->withUpdateRow('index-table-post-resource')
```

Combine with custom URL:
```php
Text::make('Name')
    ->updateOnPreview(url: '/my/url')
    ->withUpdateRow('index-table')
```

### updateInPopover

Edit in a popover instead of inline:

```php
->updateInPopover(string $component)
```

```php
Text::make('Name')->updateInPopover('index-table-post-resource')
```

---

## onChange Methods

### onChangeUrl

```php
->onChangeUrl(Closure $url, HttpMethod $method = HttpMethod::GET, array $events = [], ?string $selector = null, ?AsyncCallback $callback = null)
```

```php
Switcher::make('Active')
    ->onChangeUrl(fn() => '/endpoint')

Switcher::make('Active')
    ->onChangeUrl(fn() => '/endpoint', selector: '#my-selector')
```

### onChangeMethod

Call a resource/page method asynchronously:

```php
->onChangeMethod(string $method, array|Closure $params = [], ...)
```

```php
Switcher::make('Active')
    ->onChangeMethod('toggleActive')
```

In resource/page:
```php
public function toggleActive(MoonShineRequest $request): void
{
    // Logic
}
```

### onChangeEvent

Dispatch AlpineJS events:

```php
->onChangeEvent(array|string $events, array $exclude = [], bool $withoutPayload = false)
```

```php
Select::make('Country')
    ->onChangeEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'selects'))
```

---

## Before/After Render

```php
->beforeRender(Closure $closure)
->afterRender(Closure $closure)
```

```php
Text::make('Title')
    ->beforeRender(fn(Field $field) => $field->preview())
```

### onBeforeRender Hook

```php
->onBeforeRender(Closure $callback)
```

```php
Text::make('Thumbnail')
    ->onBeforeRender(fn(Text $ctx) => /* ... */)
```

---

## Request Value

```php
->onRequestValue(Closure $callback)
```

```php
// Override how value is extracted from request
Text::make('Field')
    ->onRequestValue(fn($value, $name, $default, $ctx) => /* custom logic */)
```

---

## Reactivity

```php
->reactive(?Closure $callback = null, bool $lazy = false, int $debounce = 0, int $throttle = 0)
```

```php
Text::make('Title')
    ->reactive(function(Fields $fields, ?string $value): Fields {
        return tap($fields, fn($f) =>
            $f->findByColumn('slug')?->setValue(str($value ?? '')->slug()->value())
        );
    }),

Text::make('Slug')->reactive()
```

> A reactive field can change OTHER fields' state, but NOT its own.

To change the initiating field's own state:
```php
Select::make('Category', 'category_id')
    ->reactive(function(Fields $fields, ?string $value, Field $field, array $values): Fields {
        $field->setValue($value);  // explicitly set own value
        // ... modify other fields
        return $fields;
    })
```

---

## Assets

```php
->addAssets(array $assets)
```

```php
Text::make('Name')->addAssets([
    new Css(Vite::asset('resources/css/text-field.css'))
])
```

In custom fields:
```php
protected function assets(): array
{
    return [
        Js::make('/js/custom.js'),
        Css::make('/css/styles.css')
    ];
}
```

---

## Macroable

All fields support Laravel's Macroable trait:

```php
Field::macro('myMethod', fn() => /* implementation */)
Text::make()->myMethod()

Field::mixin(new MyNewMethods())
```

---

## Creating Custom Fields

```shell
php artisan moonshine:field MyCustomField
```

This generates a field class with a Blade view. Key methods to override:

- `resolvePreview()` - how the field looks in preview
- `resolveOnApply()` - how the field saves data
- View file for default rendering
- `assets()` - CSS/JS assets
