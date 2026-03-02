# Field Values & Apply Lifecycle Reference

Value methods, fill logic, preview customization, and apply lifecycle for MoonShine v3 fields.

## Value Methods

### Default & Nullable

```php
->default(mixed $default)
->nullable(Closure|bool|null $condition = null)
```

### Fill

```php
->fill(mixed $value = null, ?DataWrapperContract $casted = null, int $index = 0)
```

### Change Fill Logic

```php
->changeFill(Closure $callback)   // modify what value is filled
->afterFill(Closure $callback)    // run logic after fill completes
```

```php
Select::make('Images')
    ->changeFill(fn(Article $article, Select $ctx) =>
        $article->images->map(fn($v) => "https://cdn.example.com$v")->toArray()
    )
```

```php
Select::make('Links')
    ->afterFill(function(Select $ctx) {
        if (collect($ctx->toValue())->every(fn($v) => str_contains($v, 'cdn'))) {
            return $ctx->customWrapperAttributes(['class' => 'full-url']);
        }
        return $ctx;
    })
```

### Change Preview

```php
->changePreview(Closure $callback)
```

```php
Text::make('Thumbnail')
    ->changePreview(fn(?string $value, Text $field) => Thumbnails::make($value))
```

### Change Render

Completely replace field rendering:

```php
->changeRender(Closure $callback)
```

```php
Select::make('Links')
    ->changeRender(fn(?array $values, Select $ctx) =>
        Text::make($ctx->getLabel())->fill(implode(',', $values))
    )
```

### Custom View

```php
->customView(string $view, array $data = [])
```

```php
Text::make('Title')->customView('fields.my-custom-input')
```

### Raw Value Modification

```php
->modifyRawValue(Closure $callback)   // modify the value in raw/export mode
->fromRaw(Closure $callback)          // convert raw value back to usable value (import)
```

```php
BelongsTo::make('User')
    ->modifyRawValue(fn(int $rawUserId, Article $model, BelongsTo $ctx) => $model->user->name)

BelongsTo::make('User')
    ->fromRaw(fn(string $name) => User::where('name', $name)->value('id'))
```

---

## Apply Lifecycle Methods

### onApply

Override the default apply behavior:

```php
->onApply(Closure $callback)
```

```php
// Model saving context:
Text::make('Thumbnail', 'thumbnail')
    ->onApply(function(Model $item, $value, Text $field) {
        $item->thumbnail = Storage::put('thumb.jpg', file_get_contents($value));
        return $item;
    })

// Filter context:
Text::make('Title')
    ->onApply(function(Builder $query, mixed $value, Text $field) {
        $query->where('title', $value);
    })
```

### onBeforeApply / onAfterApply

```php
->onBeforeApply(Closure $callback)   // before main apply
->onAfterApply(Closure $callback)    // after model save
```

### canApply

```php
->canApply(Closure $callback)        // conditionally skip apply
```

```php
Text::make('Title')->canApply(fn() => false) // never applies
```

### refreshAfterApply

```php
->refreshAfterApply(?Closure $callback = null)
->disableRefreshAfterApply()
```

File/Image fields auto-refresh after apply. Disable with `disableRefreshAfterApply()`.

### Global Apply Logic

Create a custom apply class:

```shell
php artisan moonshine:apply FileModelApply
```

```php
final class FileModelApply implements ApplyContract
{
    public function apply(FieldContract $field): Closure
    {
        return function(mixed $item) use ($field): mixed {
            $requestValue = $field->getRequestValue();
            return data_set($item, $field->getColumn(), $newValue);
        };
    }
}
```

Register in service provider:

```php
$applies->for(ModelResource::class)->fields()->add(File::class, FileModelApply::class);
```
