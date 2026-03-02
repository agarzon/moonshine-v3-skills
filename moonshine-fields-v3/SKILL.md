---
name: moonshine-fields-v3
description: Use when working with any MoonShine v3 field type, relationship fields, field validation, field lifecycle, custom apply logic, field modes, or creating custom fields.
---

# MoonShine v3 Fields

## Field Concept

Fields are the core building blocks of the MoonShine admin panel. They are used in:
- `FormBuilder` for building forms
- `TableBuilder` for creating tables
- `ModelResource` for filters
- Custom pages and Blade views

Every field is created via the static `make()` method:

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')           // label only, column auto-detected as "title"
Text::make('Title', 'title')  // explicit label + column
Text::make('Title', 'first_name', fn($item) => $item->first_name . ' ' . $item->last_name) // with formatted callback
```

Constructor signature for all fields:
```php
make(Closure|string|null $label = null, ?string $column = null, ?Closure $formatted = null)
```

> Fields in MoonShine are NOT tied to a model (except `Slug` and relationship fields).

## Field Modes

Every field has three visual states:

| Mode | Method | When Used | Description |
|------|--------|-----------|-------------|
| **Default** | `defaultMode()` | Forms | Renders as HTML form element (`<input>`, `<select>`, etc.) |
| **Preview** | `previewMode()` | Tables, detail views | Displays formatted value (images as thumbnails, dates formatted, etc.) |
| **Raw** | `rawMode()` | Export | Outputs the original unmodified value |

```php
Text::make('Title')->defaultMode()  // always form element
Text::make('Title')->previewMode()  // always preview display
Text::make('Title')->rawMode()      // always raw value
```

## Field Lifecycle

### In FormBuilder (saving)
1. Field declared in resource
2. Field enters FormBuilder
3. FormBuilder fills the field (`fill()`)
4. FormBuilder renders the field
5. On submit: `beforeApply()` -> `apply()` -> model `save()` -> `afterApply()`

### In TableBuilder (display)
1. Field enters TableBuilder in preview mode
2. TableBuilder iterates data, fills each field
3. TableBuilder renders rows with field previews

### In Export
1. Field enters Handler in raw mode
2. Handler iterates data, fills fields, generates table from raw values

## Apply Lifecycle

Fields modify incoming data during the "apply" phase. For model saving:

```php
Text::make('Thumbnail by link', 'thumbnail')
    ->onApply(function(Model $item, $value, Text $field) {
        if ($value) {
            $item->thumbnail = Storage::put('thumbnail.jpg', file_get_contents($value));
        }
        return $item;
    })
```

For filters (QueryBuilder):
```php
Text::make('Title')
    ->onApply(function(Builder $query, mixed $value, Text $field) {
        $query->where('title', $value);
    })
```

Related methods: `onBeforeApply()`, `onAfterApply()`, `canApply()`.

## Quick Reference: All Field Types

### Basic Input Fields
| Field | Namespace | Description |
|-------|-----------|-------------|
| `Text` | `MoonShine\UI\Fields\Text` | Text input (`<input type="text">`) |
| `Textarea` | `MoonShine\UI\Fields\Textarea` | Multi-line text |
| `Number` | `MoonShine\UI\Fields\Number` | Numeric input with min/max/step |
| `Password` | `MoonShine\UI\Fields\Password` | Password input (auto-hashes on apply) |
| `PasswordRepeat` | `MoonShine\UI\Fields\PasswordRepeat` | Confirmation field (no apply) |
| `Email` | `MoonShine\UI\Fields\Email` | Email input |
| `Phone` | `MoonShine\UI\Fields\Phone` | Phone input with mask support |
| `Url` | `MoonShine\UI\Fields\Url` | URL input with link preview |
| `Color` | `MoonShine\UI\Fields\Color` | Color picker |
| `Date` | `MoonShine\UI\Fields\Date` | Date/datetime input |
| `DateRange` | `MoonShine\UI\Fields\DateRange` | Date range (from/to) |
| `Range` | `MoonShine\UI\Fields\Range` | Numeric range (from/to) |
| `RangeSlider` | `MoonShine\UI\Fields\RangeSlider` | Range with slider UI |
| `Slug` | `MoonShine\Laravel\Fields\Slug` | Auto-generated slug (model-dependent) |
| `ID` | `MoonShine\UI\Fields\ID` | Primary key (hidden in forms) |
| `Hidden` | `MoonShine\UI\Fields\Hidden` | Hidden input |
| `HiddenIds` | `MoonShine\UI\Fields\HiddenIds` | Collects selected IDs for bulk actions |
| `Position` | `MoonShine\UI\Fields\Position` | Row numbering in repeating elements |
| `Preview` | `MoonShine\UI\Fields\Preview` | Read-only display (badge, boolean, link, image) |
| `Template` | `MoonShine\UI\Fields\Template` | Empty field built via fluent interface |

### Selection Fields
| Field | Namespace | Description |
|-------|-----------|-------------|
| `Select` | `MoonShine\UI\Fields\Select` | Dropdown (options, async, searchable, multiple) |
| `Enum` | `MoonShine\UI\Fields\Enum` | Select from PHP Enum |
| `Checkbox` | `MoonShine\UI\Fields\Checkbox` | Checkbox (on/off values) |
| `Switcher` | `MoonShine\UI\Fields\Switcher` | Toggle switch (extends Checkbox) |

### File & Editor Fields
| Field | Namespace | Description |
|-------|-----------|-------------|
| `File` | `MoonShine\UI\Fields\File` | File upload (disk, dir, extensions, multiple) |
| `Image` | `MoonShine\UI\Fields\Image` | Image upload with preview |
| `Code` | Package `moonshine/ace` | Code editor (Ace) |
| `Markdown` | Package `moonshine/easymde` | Markdown editor (EasyMDE) |
| `TinyMCE` | Package `moonshine/tinymce` | WYSIWYG editor |

### Structural Fields
| Field | Namespace | Description |
|-------|-----------|-------------|
| `Json` | `MoonShine\UI\Fields\Json` | JSON data (fields, keyValue, onlyValue, object) |
| `Fieldset` | `MoonShine\UI\Fields\Fieldset` | Groups fields visually |

### Relationship Fields
| Field | Namespace | Laravel Relation |
|-------|-----------|-----------------|
| `BelongsTo` | `MoonShine\Laravel\Fields\Relationships\BelongsTo` | `BelongsTo` |
| `BelongsToMany` | `MoonShine\Laravel\Fields\Relationships\BelongsToMany` | `BelongsToMany` |
| `HasOne` | `MoonShine\Laravel\Fields\Relationships\HasOne` | `HasOne` |
| `HasMany` | `MoonShine\Laravel\Fields\Relationships\HasMany` | `HasMany` |
| `HasOneThrough` | `MoonShine\Laravel\Fields\Relationships\HasOneThrough` | `HasOneThrough` |
| `HasManyThrough` | `MoonShine\Laravel\Fields\Relationships\HasManyThrough` | `HasManyThrough` |
| `MorphOne` | `MoonShine\Laravel\Fields\Relationships\MorphOne` | `MorphOne` |
| `MorphMany` | `MoonShine\Laravel\Fields\Relationships\MorphMany` | `MorphMany` |
| `MorphTo` | `MoonShine\Laravel\Fields\Relationships\MorphTo` | `MorphTo` |
| `MorphToMany` | `MoonShine\Laravel\Fields\Relationships\MorphToMany` | `MorphToMany` |
| `RelationRepeater` | `MoonShine\Laravel\Fields\Relationships\RelationRepeater` | `HasMany`/`HasOne` inline |

## Relationship Fields Overview

All relationship fields require a registered `ModelResource`. Constructor:

```php
BelongsTo::make(
    Closure|string $label,
    ?string $relationName = null,      // auto-detected from label via camelCase
    Closure|string|null $formatted = null, // display column or closure
    ModelResource|string|null $resource = null, // auto-detected from relation name
)
```

**Key patterns across relationship fields:**
- `asyncSearch()` - async search with custom query, formatted output, limit
- `creatable()` - create related object via modal
- `valuesQuery()` - modify the query that fetches dropdown values
- `withImage()` - add images to dropdown items
- `searchable()` - enable search in dropdown
- `selectMode()` - display BelongsToMany as dropdown instead of checkboxes
- `relatedLink()` - show as a link with count instead of full list
- `tabMode()` / `modalMode()` - display HasOne/HasMany in tabs or modals
- `disableOutside()` - render inside the main form instead of separately

## Common Field Methods (All Fields)

### Display & Attributes
```php
->setLabel('New Label')       // change label
->translatable('ui')          // translate label
->hint('Helper text')         // add hint text
->badge('success')            // show as colored badge in preview
->horizontal()                // label and field side by side
->withoutWrapper()            // remove wrapper div
->sortable()                  // enable column sorting in tables
->required()                  // mark as required
->disabled()                  // disable input
->readonly()                  // read-only
->customAttributes(['class' => 'my-class'])
->customWrapperAttributes(['class' => 'mt-8'])
```

### Value Modification
```php
->default('value')            // default value
->nullable()                  // allow NULL
->fill('value')               // manually fill
->changeFill(fn($data, $ctx) => ...)  // modify fill logic
->afterFill(fn($ctx) => ...)          // post-fill logic
->changePreview(fn($value, $ctx) => ...) // custom preview
->changeRender(fn($value, $ctx) => ...)  // replace entire render
->modifyRawValue(fn($raw, $data, $ctx) => ...) // modify raw/export value
->fromRaw(fn($raw, $ctx) => ...)      // convert raw to value (import)
```

### Apply Lifecycle
```php
->onApply(fn($item, $value, $field) => $item)
->onBeforeApply(fn($item, $value, $field) => $item)
->onAfterApply(fn($item, $value, $field) => $item)
->canApply(fn() => false)     // skip apply entirely
```

### Conditional Display
```php
->canSee(fn($field) => true)          // show/hide conditionally
->when(fn() => true, fn($field) => $field->locked())
->showWhen('other_field', '=', 'value') // dynamic JS-based show/hide
->showWhenDate('created_at', '>', '2024-01-01')
```

### Inline Editing (Preview Mode)
```php
->updateOnPreview()           // edit in table cells
->withUpdateRow('table-name') // update entire row on change
->updateInPopover('table-name') // edit in popover
```

### Reactivity
```php
->reactive(function(Fields $fields, ?string $value): Fields {
    return tap($fields, fn($f) => $f->findByColumn('slug')?->setValue(str($value)->slug()->value()));
})
```

## Creating a Custom Field

```shell
php artisan moonshine:field
```

This generates a field class with its own Blade view. Override `resolvePreview()`, `resolveOnApply()`, and the view to customize behavior.

## Cross-References

- For how fields are used in resources, see the `moonshine-resources-v3` skill
- Relationship fields require registered `ModelResource` instances
- Fields work with `FormBuilder` and `TableBuilder` components
- Filters use the same field classes with filter-specific apply logic

## Reference Files

- `references/basic-fields.md` - Text, Number, Date, Slug, ID, and other basic input fields
- `references/selection-fields.md` - Select, Enum, Checkbox, Switcher
- `references/file-fields.md` - File, Image, Code, Markdown, TinyMCE editors
- `references/relationship-fields.md` - All relationship field types with patterns
- `references/field-display-attributes.md` - Display methods, view modes, and attribute configuration
- `references/field-values-lifecycle.md` - Value methods, fill logic, and apply lifecycle
- `references/field-interactive.md` - Conditional display, showWhen, inline editing, onChange, reactivity, custom fields
