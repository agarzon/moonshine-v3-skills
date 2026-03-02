# Field Display & Attributes Reference

Field creation, display methods, view modes, and attribute configuration for MoonShine v3 fields.

## Field Creation

```php
make(Closure|string|null $label = null, ?string $column = null, ?Closure $formatted = null)
```

- `$label` - field title (HTML allowed, not escaped)
- `$column` - database column / request `name` attribute (auto-detected from label if English)
- `$formatted` - closure for formatting value in preview mode

```php
Text::make('Full Name', 'first_name', fn($item) => $item->first_name . ' ' . $item->last_name)
```

> Fields that do NOT support `$formatted`: `Json`, `File`, `Range`, `RangeSlider`, `DateRange`, `Select`, `Enum`, `HasOne`, `HasMany`.

---

## Display Methods

### Label

```php
->setLabel(Closure|string $label)
->translatable(string $key = '')    // translate via Lang
->insideLabel()                      // wrap field in <label>
->beforeLabel()                      // display label after field
```

```php
Slug::make('Slug')->setLabel(fn(Field $field) => $field->getData()?->exists ? 'Slug (do not change)' : 'Slug')

Text::make('ui.Title')->translatable()  // uses __('ui.Title')
Text::make('Title')->translatable('ui') // uses __('ui.Title')
```

### Hint

```php
->hint(string $hint)
```

```php
Number::make('Rating')->hint('From 0 to 5')->min(0)->max(5)
```

### Link

```php
->link(string|Closure $link, string|Closure $name = '', ?string $icon = null, bool $withoutIcon = false, bool $blank = false)
```

```php
Text::make('Docs')->link('https://example.com', 'Documentation', blank: true)
```

### Badge

```php
->badge(string|Color|Closure|null $color = null)
```

Colors: `primary`, `secondary`, `success`, `warning`, `error`, `info`, `purple`, `pink`, `blue`, `green`, `yellow`, `red`, `gray`.

```php
Text::make('Status')->badge(fn($value, Field $field) => $value === 'active' ? 'green' : 'gray')
```

### Horizontal Layout

```php
->horizontal()    // label and field side by side
```

### Wrapper

```php
->withoutWrapper(mixed $condition = null)  // remove wrapper div
```

### Text Wrap

```php
->textWrap(TextWrap::CLAMP)     // clamp overflow
->textWrap(TextWrap::ELLIPSIS)  // ellipsis overflow
->withoutTextWrap()             // disable text wrapping
```

> Default: `Text` uses ELLIPSIS, `Textarea` uses CLAMP.

### Sorting

```php
->sortable(Closure|string|null $callback = null)
```

```php
Text::make('Title')->sortable()

BelongsTo::make('Author')->sortable('author_id')

Text::make('Title')->sortable(function(Builder $query, string $column, string $direction) {
    $query->orderBy($column, $direction);
})
```

### Column Selection

```php
->columnSelection(bool $active = true, bool $hideOnInit = false)
->sticky()  // sticky column in table
```

---

## View Modes

```php
->defaultMode()   // always render as form element
->previewMode()   // always render as preview
->rawMode()       // always render raw value
```

---

## Attribute Methods

### Required / Disabled / Readonly

```php
->required(Closure|bool|null $condition = null)
->disabled(Closure|bool|null $condition = null)
->readonly(Closure|bool|null $condition = null)
```

### Custom Attributes

```php
->customAttributes(array $attributes, bool $override = false)
```

```php
Password::make('Password')->customAttributes(['autocomplete' => 'off'])
Textarea::make('Text')->customAttributes(['rows' => 6])
```

### Wrapper Attributes

```php
->customWrapperAttributes(array $attributes)
```

```php
Text::make('Title')->customWrapperAttributes(['class' => 'mt-8'])
```

### Name Attribute

```php
->setNameAttribute('custom_name')   // override name
->wrapName('options')               // result: name="options[field_name]"
->virtualColumn('image_1')          // virtual name for conditional fields
```
