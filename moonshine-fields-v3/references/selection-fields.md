# Selection Fields Reference

## Select

`MoonShine\UI\Fields\Select` - Dropdown selection field. Uses Choices.js library by default.

### Basic Usage

```php
use MoonShine\UI\Fields\Select;

Select::make('Country', 'country_id')
    ->options([
        'value 1' => 'Option Label 1',
        'value 2' => 'Option Label 2',
    ])
```

### Constructor

```php
Select::make(Closure|string|null $label = null, ?string $column = null, ?Closure $formatted = null)
```

### Options via Object API

```php
use MoonShine\UI\Fields\Select;

Select::make('Select')
    ->options(
        new Options([
            new Option(
                label: 'Option 1',
                value: '1',
                selected: true,
                properties: new OptionProperty(image: 'https://example.com/img.png'),
            ),
            new Option(label: 'Option 2', value: '2'),
        ])
    )
```

### All Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `options()` | `options(array\|Options $options)` | Set available options |
| `default()` | `default(mixed $default)` | Default selected value |
| `nullable()` | `nullable(Closure\|bool\|null $condition = null)` | Allow NULL (adds empty option) |
| `placeholder()` | `placeholder(string $value)` | Placeholder text |
| `multiple()` | `multiple(Closure\|bool\|null $condition = null)` | Enable multi-select |
| `searchable()` | `searchable()` | Enable search in dropdown |
| `native()` | `native()` | Disable Choices.js, use native `<select>` |
| `async()` | `async(Closure\|string\|null $url, ...)` | Async search from URL |
| `asyncOnInit()` | `asyncOnInit(bool $whenOpen = true)` | Load values on page load or on open |
| `onChangeEvent()` | `onChangeEvent(array\|string $events, ...)` | Dispatch events on change |
| `updateOnPreview()` | `updateOnPreview(...)` | Inline editing in preview |
| `optionProperties()` | `optionProperties(Closure\|array $data)` | Add images to options |

### Option Groups

```php
Select::make('City', 'city_id')
    ->options([
        'Italy' => [
            1 => 'Rome',
            2 => 'Milan',
        ],
        'France' => [
            3 => 'Paris',
            4 => 'Marseille',
        ]
    ])
```

Or via `OptionGroup` objects:
```php
Select::make('City')
    ->options(
        new Options([
            new OptionGroup('Italy', new Options([
                new Option('Rome', '1'),
                new Option('Milan', '2'),
            ])),
            new OptionGroup('France', new Options([
                new Option('Paris', '3'),
                new Option('Marseille', '4'),
            ])),
        ])
    )
```

### Multiple Selection

```php
Select::make('Countries', 'country_ids')
    ->options([1 => 'USA', 2 => 'UK', 3 => 'France'])
    ->multiple()
```

> When using `multiple()` with a model, add a cast to the model attribute: `'country_ids' => 'array'` or `'json'`.

### Searchable

```php
Select::make('Country', 'country_id')
    ->options([...])
    ->searchable()
```

### Async Search

The endpoint must return JSON array of `{value, label}` objects. The request includes a `query` parameter.

```php
Select::make('Country', 'country_id')
    ->options([...])       // initial options (optional)
    ->async('/api/search') // URL for async search
```

Expected JSON response:
```json
[
    {"value": 1, "label": "Option 1"},
    {"value": 2, "label": "Option 2"}
]
```

Load values immediately on page display:
```php
Select::make('Country', 'country_id')
    ->async('/search')
    ->asyncOnInit(whenOpen: false) // load immediately; true/empty = load on click
```

### Change Events

```php
Select::make('Country', 'country_id')
    ->options([...])
    ->onChangeEvent(
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'selects'),
        exclude: ['text', 'description'],  // optional: exclude form fields from payload
    )
```

### Values with Images

```php
Select::make('Country', 'country_id')
    ->options([1 => 'Andorra', 2 => 'UAE'])
    ->optionProperties(fn() => [
        1 => ['image' => 'https://example.com/ad.png'],
        2 => ['image' => 'https://example.com/ae.png'],
    ])
```

Custom image sizing:
```php
new OptionProperty(
    new OptionImage(
        src: 'https://example.com/img.png',
        height: 6,   // h-{x} class, 1-10
        width: 6,     // w-{x} class, 1-10
        objectFit: ObjectFit::CONTAIN
    )
)
```

### Inline Editing

```php
Select::make('Country')
    ->updateOnPreview()
```

### Native Mode

```php
Select::make('Type')->native()
```

### Choices.js Custom Options

```php
Select::make('Country', 'country_id')
    ->options([...])
    ->customAttributes([
        'data-max-item-count' => 2
    ])
```

---

## Checkbox

`MoonShine\UI\Fields\Checkbox` - Boolean checkbox field.

### Basic Usage

```php
use MoonShine\UI\Fields\Checkbox;

Checkbox::make('Publish', 'is_publish')
```

### On/Off Values

Default values are `1` (checked) and `0` (unchecked). Override with:

```php
Checkbox::make('Publish', 'is_publish')
    ->onValue('yes')
    ->offValue('no')
```

### Methods

| Method | Description |
|--------|-------------|
| `onValue(int\|string $value)` | Value when checked (default: `1`) |
| `offValue(int\|string $value)` | Value when unchecked (default: `0`) |
| `updateOnPreview()` | Enable inline toggle in tables |

```php
Checkbox::make('Active', 'is_active')
    ->updateOnPreview()
```

---

## Switcher

`MoonShine\UI\Fields\Switcher` - Toggle switch. Inherits from Checkbox with different visual styling.

```php
use MoonShine\UI\Fields\Switcher;

Switcher::make('Publish', 'is_publish')
```

Has all the same methods as Checkbox (`onValue()`, `offValue()`, `updateOnPreview()`).

```php
Switcher::make('Active')
    ->onChangeUrl(fn() => '/endpoint')          // async call on toggle
    ->onChangeMethod('someMethod')              // call resource/page method
```

---

## Radio

Radio buttons are typically implemented using `Select` with `native()` mode or by creating a custom field. MoonShine v3 does not have a standalone Radio field class -- use `Select` or `Enum` for single-choice scenarios.

---

## Common Patterns for Selection Fields

### Using Enum with Select behavior

```php
Enum::make('Status')
    ->attach(StatusEnum::class)
    ->default(StatusEnum::DRAFT->value)
```

### Conditional options (reactive)

```php
Select::make('Category', 'category_id')
    ->options([...])
    ->reactive(function(Fields $fields, ?string $value, Field $field, array $values): Fields {
        $field->setValue($value);
        return tap($fields, fn($f) =>
            $f->findByColumn('article_id')
              ?->options(Article::where('category_id', $value)->pluck('title', 'id')->toArray())
        );
    }),

Select::make('Article', 'article_id')
    ->options([])
    ->reactive()
```

### Select with badge in preview

```php
Select::make('Status', 'status')
    ->options(['active' => 'Active', 'inactive' => 'Inactive'])
    ->badge(fn($value) => $value === 'active' ? 'success' : 'gray')
```
