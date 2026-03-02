# Basic Fields Reference

## Text

`MoonShine\UI\Fields\Text` - Basic text input (`<input type="text">`).

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')
Text::make('Title', 'title')
```

### Key Methods

| Method | Description |
|--------|-------------|
| `placeholder(string $value)` | Set placeholder text |
| `mask(string $mask)` | Apply input mask (e.g., `'+7 (999) 999-99-99'`) |
| `tags(?int $limit = null)` | Transform into tag input |
| `unescape()` | Disable HTML escaping in preview |
| `copy()` | Add copy-to-clipboard button |
| `eye()` | Add show/hide toggle (password-style) |
| `locked()` | Add lock icon |
| `suffix(string $ext)` | Add suffix (e.g., `.com`) |
| `updateOnPreview()` | Enable inline editing in tables |

```php
Text::make('Phone', 'phone')
    ->mask('+7 (999) 999-99-99')
    ->placeholder('Enter phone')

Text::make('Domain', 'domain')
    ->suffix('.com')
    ->copy()

Text::make('Tags', 'tags')
    ->tags(5)

Text::make('Name')
    ->updateOnPreview()
    ->locked()
```

---

## Textarea

`MoonShine\UI\Fields\Textarea` - Multi-line text (`<textarea>`).

```php
use MoonShine\UI\Fields\Textarea;

Textarea::make('Description')
```

### Key Methods

| Method | Description |
|--------|-------------|
| `unescape()` | Disable HTML escaping |
| `customAttributes(['rows' => 6])` | Set height via rows attribute |

```php
Textarea::make('Description')
    ->customAttributes(['rows' => 6])
    ->unescape()
```

---

## Number

`MoonShine\UI\Fields\Number` - Numeric input (`<input type="number">`).

```php
use MoonShine\UI\Fields\Number;

Number::make('Sort')
```

### Key Methods

| Method | Description |
|--------|-------------|
| `min(int\|float $min)` | Minimum value |
| `max(int\|float $max)` | Maximum value |
| `step(int\|float $step)` | Step increment |
| `buttons()` | Add +/- buttons |
| `stars()` | Display as stars in preview (for ratings) |
| `default(mixed $default)` | Default value |
| `placeholder(string $value)` | Placeholder text |
| `copy()`, `eye()`, `locked()`, `suffix()` | Extensions |
| `updateOnPreview()` | Inline editing |

```php
Number::make('Price')
    ->min(0)
    ->max(100000)
    ->step(5)
    ->buttons()

Number::make('Rating')
    ->stars()
    ->min(1)
    ->max(10)
    ->default(5)
```

---

## Password

`MoonShine\UI\Fields\Password` - Password input. Inherits from Text. Auto-hashes value on apply using Laravel's `Hash` facade.

```php
use MoonShine\UI\Fields\Password;
use MoonShine\UI\Fields\PasswordRepeat;

Password::make('Password', 'password'),
PasswordRepeat::make('Password repeat', 'password_repeat')
```

- Displays as `***` in preview mode
- `PasswordRepeat` does NOT modify data on apply (confirmation only)

---

## Email

`MoonShine\UI\Fields\Email` - Inherits from Text. Sets `type=email`.

```php
use MoonShine\UI\Fields\Email;

Email::make('Email')
```

---

## Phone

`MoonShine\UI\Fields\Phone` - Inherits from Text. Sets `type=tel`.

```php
use MoonShine\UI\Fields\Phone;

Phone::make('Phone')
    ->mask('7 999 999-99-99')
```

---

## Url

`MoonShine\UI\Fields\Url` - Inherits from Text. Sets `type=url`.

```php
use MoonShine\UI\Fields\Url;

Url::make('Link')
    ->title(fn(string $url, Url $ctx) => str($url)->limit(3))
    ->blank()  // opens in new tab in preview
```

### Key Methods

| Method | Description |
|--------|-------------|
| `title(Closure $callback)` | Custom link title in preview |
| `blank()` | Add `target="_blank"` to preview link |

---

## Color

`MoonShine\UI\Fields\Color` - Color picker input.

```php
use MoonShine\UI\Fields\Color;

Color::make('Color')
```

---

## Date

`MoonShine\UI\Fields\Date` - Date input (`<input type="date">`).

```php
use MoonShine\UI\Fields\Date;

Date::make('Created at', 'created_at')
```

### Key Methods

| Method | Description |
|--------|-------------|
| `withTime()` | Enable datetime input |
| `format(string $format)` | Preview format (e.g., `'d.m.Y'`) |
| `inputFormat(string $format)` | Edit format (e.g., `'H:i'`) |
| `updateOnPreview()` | Inline editing |

```php
Date::make('Created at', 'created_at')
    ->withTime()
    ->format('d.m.Y H:i')
```

For time-only input:
```php
Text::make('Time')->setAttribute('type', 'time')
```

---

## DateRange

`MoonShine\UI\Fields\DateRange` - Select a date range with two fields.

```php
use MoonShine\UI\Fields\DateRange;

DateRange::make('Dates')
    ->fromTo('date_from', 'date_to')
```

### Key Methods

| Method | Description |
|--------|-------------|
| `fromTo(string $from, string $to)` | Required: specify the two column names |
| `withTime()` | Enable datetime |
| `format(string $format)` | Preview format |
| `min(string $min)` / `max(string $max)` | Date bounds |
| `fromAttributes(array)` / `toAttributes(array)` | Custom attributes per input |

```php
DateRange::make('Dates')
    ->fromTo('date_from', 'date_to')
    ->withTime()
    ->format('d.m.Y')
    ->min('2024-01-01')
    ->max('2024-12-31')
```

> When used as a filter, do NOT use `fromTo()` -- filtering occurs on a single DB column:
> `DateRange::make('Created', 'created_at')`

---

## Range

`MoonShine\UI\Fields\Range` - Numeric range with two inputs.

```php
use MoonShine\UI\Fields\Range;

Range::make('Price')
    ->fromTo('price_from', 'price_to')
    ->min(0)
    ->max(10000)
    ->step(5)
```

### Key Methods

Same as DateRange: `fromTo()`, `min()`, `max()`, `step()`, `stars()`, `fromAttributes()`, `toAttributes()`.

> When used as a filter: `Range::make('Age', 'age')` (no `fromTo`).

---

## RangeSlider

`MoonShine\UI\Fields\RangeSlider` - Inherits from Range. Adds slider UI.

```php
use MoonShine\UI\Fields\RangeSlider;

RangeSlider::make('Age')
    ->fromTo('age_from', 'age_to')
    ->min(18)
    ->max(100)
```

---

## Slug

`MoonShine\Laravel\Fields\Slug` - Auto-generates slug. Inherits from Text. **Depends on Eloquent model.**

```php
use MoonShine\Laravel\Fields\Slug;

Slug::make('Slug')
    ->from('title')      // source field for generation
    ->separator('_')     // word separator (default: '-')
    ->locale('ru')       // locale for slug generation
    ->unique()           // ensure uniqueness in DB
    ->live()             // dynamic slug (updates as user types)
```

Dynamic slug requires the source field to be reactive:
```php
Text::make('Title')->reactive(),
Slug::make('Slug')->from('title')->live()
```

---

## ID

`MoonShine\UI\Fields\ID` - Primary key field. Inherits from Hidden. Displayed only in preview mode.

```php
use MoonShine\UI\Fields\ID;

ID::make()                         // defaults to label='ID', column='id'
ID::make(column: 'primary_key')    // custom primary key name
```

---

## Hidden

`MoonShine\UI\Fields\Hidden` - Hidden form input (`type=hidden`).

```php
use MoonShine\UI\Fields\Hidden;

Hidden::make('category_id')
Hidden::make('category_id')->showValue()  // show value while keeping hidden behavior
```

---

## HiddenIds

`MoonShine\UI\Fields\HiddenIds` - Collects primary keys of selected elements (for bulk actions).

```php
use MoonShine\UI\Fields\HiddenIds;

HiddenIds::make('index-table')  // component name with the list
```

---

## Position

`MoonShine\UI\Fields\Position` - Row numbering for repeating elements (Json, tables).

```php
use MoonShine\UI\Fields\Position;

Json::make('Options', 'options')->fields([
    Position::make(),
    Text::make('Title'),
    Text::make('Value'),
])
```

---

## Enum

`MoonShine\UI\Fields\Enum` - Select field powered by PHP Enum. Inherits from Select.

```php
use MoonShine\UI\Fields\Enum;

Enum::make('Status')
    ->attach(StatusEnum::class)
```

**Requires Enum Cast on the model attribute.**

### Display Methods on Enum

Implement `toString()` for display labels:
```php
enum StatusEnum: string
{
    case NEW = 'new';
    case DRAFT = 'draft';

    public function toString(): ?string
    {
        return match ($this) {
            self::NEW => 'New',
            self::DRAFT => 'Draft',
        };
    }
}
```

Implement `getColor()` for colored badges in preview:
```php
public function getColor(): ?string
{
    return match ($this) {
        self::NEW => 'info',
        self::DRAFT => 'gray',
    };
}
```

Available colors: `primary`, `secondary`, `success`, `warning`, `error`, `info`, `purple`, `pink`, `blue`, `green`, `yellow`, `red`, `gray`.

---

## Preview

`MoonShine\UI\Fields\Preview` - Read-only display field. NOT for data input.

```php
use MoonShine\UI\Fields\Preview;

Preview::make('Preview', 'preview', fn() => fake()->realText())
Preview::make('Status')->badge(fn($status) => $status === 1 ? 'green' : 'gray')
Preview::make('Active')->boolean()
Preview::make('Link')->link('https://example.com', blank: true)
Preview::make('Thumb')->image()
```

---

## Template

`MoonShine\UI\Fields\Template` - Empty field for building custom fields via fluent interface.

```php
use MoonShine\UI\Fields\Template;

Template::make()
    ->setLabel('My Field')
    ->fields([
        Text::make('Title')
    ])
```
