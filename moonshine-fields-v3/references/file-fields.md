# File & Editor Fields Reference

## File

`MoonShine\UI\Fields\File` - File upload field.

> Ensure a symbolic link is set up: `php artisan storage:link`
> Set `APP_URL` in `.env` for correct URL generation.

### Basic Usage

```php
use MoonShine\UI\Fields\File;

File::make('Document', 'file')
```

### All Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `disk()` | `disk(string $disk)` | Filesystem disk (default: `'public'`) |
| `dir()` | `dir(string $dir)` | Directory relative to disk root |
| `allowedExtensions()` | `allowedExtensions(array $ext)` | Restrict file types |
| `multiple()` | `multiple(Closure\|bool\|null $condition)` | Enable multiple file uploads |
| `removable()` | `removable(Closure\|bool\|null $condition, array $attributes)` | Allow file deletion |
| `disableDeleteFiles()` | `disableDeleteFiles()` | Delete DB record only, keep file |
| `enableDeleteDir()` | `enableDeleteDir()` | Delete directory if empty after removal |
| `disableDownload()` | `disableDownload(Closure\|bool\|null $condition)` | Disable download button |
| `keepOriginalFileName()` | `keepOriginalFileName()` | Preserve original filename |
| `customName()` | `customName(Closure $name)` | Custom file name on upload |
| `names()` | `names(Closure $closure)` | Display name without changing filename |
| `reorderable()` | `reorderable(Closure $url)` | Enable drag-and-drop sorting |
| `itemAttributes()` | `itemAttributes(Closure $closure)` | Custom attributes per file item |
| `dropzoneAttributes()` | `dropzoneAttributes(Closure $closure)` | Custom dropzone attributes |

### Complete Example

```php
File::make('Documents', 'documents')
    ->disk('public')
    ->dir('docs')
    ->allowedExtensions(['pdf', 'doc', 'txt'])
    ->multiple()
    ->removable()
    ->enableDeleteDir()
    ->keepOriginalFileName()
```

### Custom File Name

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Str;

File::make('File', 'file')
    ->customName(fn(UploadedFile $file, Field $field) => Str::random(10) . '.' . $file->extension())
```

### Display Names

```php
File::make('File', 'file')
    ->names(fn(string $filename, int $index = 0) => 'File ' . ($index + 1))
```

### Drag-and-Drop Sorting

```php
File::make('Files')
    ->reorderable(fn(File $ctx) => "/reorder/" . $ctx->getData()->getKey())
    ->multiple()
```

### Removable with Custom Behavior

```php
File::make('File')
    ->removable()
    ->disableDeleteFiles()  // remove from DB but keep file on disk
```

```php
File::make('File')
    ->dir('docs')
    ->removable()
    ->enableDeleteDir()     // delete the 'docs' directory if empty
```

### Helper Methods

- `getRemainingValues()` - Get values remaining after deletions
- `removeExcludedFiles()` - Physically delete files during process

> When using `multiple()` with a model, add a cast: `'documents' => 'array'` or `'json'`.

---

## Image

`MoonShine\UI\Fields\Image` - Image upload field. Inherits from File with image preview support.

### Basic Usage

```php
use MoonShine\UI\Fields\Image;

Image::make('Thumbnail')
```

Has all File methods plus image-specific preview features.

### All File Methods Apply

```php
Image::make('Photos', 'photos')
    ->disk('public')
    ->dir('images')
    ->multiple()
    ->removable()
    ->allowedExtensions(['jpg', 'png', 'webp'])
```

### Custom Modal Preview

```php
Image::make('Avatar')
    ->extraAttributes(
        fn(string $filename, int $index): ?FileItemExtra => new FileItemExtra(
            wide: false,   // XL modal size
            auto: true,    // auto-size modal to content
            styles: 'width: 250px;'  // custom image styles in modal
        )
    )
```

### Refresh After Apply

By default, Image fields refresh their preview after apply. To disable:

```php
Image::make('Avatar')
    ->disableRefreshAfterApply()
```

Or customize:
```php
Image::make('Avatar')
    ->refreshAfterApply(fn(Image $ctx) => $ctx)
```

---

## Code Editor (Ace)

Separate package for code editing.

### Installation

```shell
composer require moonshine/ace
```

See the [package repository](https://github.com/moonshine-software/ace) for full documentation.

---

## Markdown Editor (EasyMDE)

Separate package for Markdown editing.

### Installation

```shell
composer require moonshine/easymde
```

See the [package repository](https://github.com/moonshine-software/easymde) for full documentation.

---

## TinyMCE (WYSIWYG)

Separate package for rich text editing.

### Installation

```shell
composer require moonshine/tinymce
```

See the [package repository](https://github.com/moonshine-software/tinymce) for full documentation.

---

## Json Field

`MoonShine\UI\Fields\Json` - Work with JSON data in various modes.

> When used with a model, add a cast: `'options' => 'array'` or `'json'`.

### Field Set Mode (Array of Objects)

```php
use MoonShine\UI\Fields\Json;
use MoonShine\UI\Fields\Position;
use MoonShine\UI\Fields\Switcher;
use MoonShine\UI\Fields\Text;

Json::make('Product Options', 'options')
    ->fields([
        Position::make(),
        Text::make('Title'),
        Text::make('Value'),
        Switcher::make('Active'),
    ])
    ->creatable(limit: 6)
    ->removable()
```

### Key/Value Mode

```php
Json::make('Data')
    ->keyValue()  // default: Text key + Text value
```

With custom field types:
```php
Json::make('Data', 'data')
    ->keyValue(
        keyField: Select::make('Key')->options(['vk' => 'VK', 'email' => 'E-mail']),
        valueField: Text::make('Value'),
    )
```

### Only Value Mode

```php
Json::make('Tags')
    ->onlyValue()   // stores ["value1", "value2"]
```

### Object Mode (Single Object)

```php
Json::make('Settings', 'settings')
    ->fields([
        Text::make('Title'),
        Switcher::make('Active'),
    ])
    ->object()  // stores {"title": "...", "active": true}
```

### Nested Json

```php
Json::make('Products', 'products')
    ->fields([
        Text::make('Name', 'name'),
        Json::make('Prices', 'prices')
            ->fields([
                Number::make('Wholesale', 'wholesale_price'),
                Number::make('Retail', 'retail_price'),
            ])
            ->object(),
    ])
```

### All Methods

| Method | Description |
|--------|-------------|
| `fields(iterable $fields)` | Set fields for the JSON structure |
| `keyValue(...)` | Key/value mode |
| `onlyValue(...)` | Value-only mode |
| `object()` | Single object mode (not array of objects) |
| `creatable(?int $limit, ?ActionButton $button)` | Allow adding new rows |
| `removable(array $attributes)` | Allow removing rows |
| `vertical()` | Vertical layout |
| `reorderable(bool\|string $urlOrBool)` | Drag-and-drop sorting (enabled by default) |
| `filterMode()` | Adapt for filter usage |
| `default(array $default)` | Default values |
| `stopFilteringEmpty()` | Disable empty value filtering |
| `modifyTable(Closure $callback)` | Modify the TableBuilder |
| `modifyRemoveButton(Closure $callback)` | Modify remove button |
| `modifyCreateButton(Closure $callback)` | Modify create button |
| `buttons(array $buttons)` | Override row buttons |

### Default Values

```php
Json::make('Data')
    ->keyValue()
    ->default([
        ['key' => 'Default key', 'value' => 'Default value']
    ])
```
