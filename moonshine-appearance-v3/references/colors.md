# Color Management Reference

Complete API reference for MoonShine v3 color management system.

---

## ColorManager

`MoonShine\ColorManager\ColorManager` manages the color scheme for both light and dark themes. It implements `ColorManagerContract`.

### Default colors

```php
ColorManager::DEFAULT = [
    'primary'      => '120, 67, 233',
    'secondary'    => '236, 65, 118',
    'body'         => '27, 37, 59',
    'dark' => [
        'DEFAULT' => '30, 31, 67',       // base dark color
        50  => '83, 103, 132',           // search, toasts, progress bars
        100 => '74, 90, 121',           // dividers
        200 => '65, 81, 114',           // dividers
        300 => '53, 69, 103',           // borders
        400 => '48, 61, 93',            // dropdowns, buttons, pagination
        500 => '41, 53, 82',            // buttons default background
        600 => '40, 51, 78',            // table rows
        700 => '39, 45, 69',            // content background
        800 => '27, 37, 59',            // sidebar background
        900 => '15, 23, 42',            // main background
    ],
    'success-bg'   => '0, 170, 0',
    'success-text' => '255, 255, 255',
    'warning-bg'   => '255, 220, 42',
    'warning-text' => '139, 116, 0',
    'error'        => '224, 45, 45',
    'error-bg'     => '224, 45, 45',
    'error-text'   => '255, 255, 255',
    'info-bg'      => '0, 121, 255',
    'info-text'    => '255, 255, 255',
];
```

### Dark theme defaults

```php
ColorManager::DARK = [
    'body'         => '27, 37, 59',
    'success-bg'   => '17, 157, 17',
    'success-text' => '178, 255, 178',
    'warning-bg'   => '225, 169, 0',
    'warning-text' => '255, 255, 199',
    'error'        => '185, 28, 28',
    'error-bg'     => '190, 10, 10',
    'error-text'   => '255, 197, 197',
    'info-bg'      => '38, 93, 205',
    'info-text'    => '179, 220, 255',
];
```

### Setting colors

```php
// Set a single color (light theme)
$colorManager->set('primary', '120, 67, 233');

// Set a single color (dark theme)
$colorManager->set('primary', '120, 67, 233', dark: true);

// Set a dark shade
$colorManager->set('dark.500', '41, 53, 82');

// Bulk assign
$colorManager->bulkAssign([
    'primary' => '120, 67, 233',
    'secondary' => '236, 65, 118',
]);

// Bulk assign for dark theme
$colorManager->bulkAssign([
    'body' => '27, 37, 59',
], dark: true);
```

### Getting colors

```php
// Get color as HEX (default)
$colorManager->get('primary');              // '#7843e9'

// Get color as RGB
$colorManager->get('primary', hex: false);  // '120, 67, 233'

// Get a specific shade
$colorManager->get('dark', 500);            // HEX of dark-500

// Get dark theme color
$colorManager->get('body', dark: true);

// Get all colors as ['name' => 'rgb_value'] array
$colorManager->getAll();                    // light theme
$colorManager->getAll(dark: true);          // dark theme
```

### Semantic helpers

High-level methods that set multiple related shades at once:

| Method | Sets shades | Description |
|--------|-------------|-------------|
| `background(string $value)` | body, dark.800, body(dark) | Page background |
| `content(string $value)` | dark.700, dark.900 | Content area background |
| `tableRow(string $value)` | dark.600 | Table row background |
| `borders(string $value)` | dark.300 | Border color |
| `dropdowns(string $value)` | dark.400 | Dropdown background |
| `buttons(string $value)` | dark.50, dark.500, dark.400 | Button backgrounds |
| `dividers(string $value)` | dark.100, dark.200 | Divider/separator color |

```php
$colorManager
    ->background('27, 37, 59')
    ->content('39, 45, 69')
    ->tableRow('40, 51, 78')
    ->borders('53, 69, 103')
    ->dropdowns('48, 61, 93')
    ->buttons('83, 103, 132')
    ->dividers('74, 90, 121');
```

### Dynamic methods

ColorManager uses `__call()` to support camelCase method names for all color keys:

```php
$colorManager->primary('120, 67, 233');
$colorManager->primary('#7843e9');            // HEX also accepted
$colorManager->secondary('236, 65, 118');
$colorManager->body('27, 37, 59');
$colorManager->dark('30, 31, 67', 'DEFAULT');
$colorManager->dark('83, 103, 132', 50);
$colorManager->dark('41, 53, 82', 500, dark: true);
$colorManager->successBg('0, 170, 0');
$colorManager->successText('255, 255, 255');
$colorManager->warningBg('255, 220, 42');
$colorManager->warningText('139, 116, 0');
$colorManager->errorBg('224, 45, 45');
$colorManager->errorText('255, 255, 255');
$colorManager->infoBg('0, 121, 255');
$colorManager->infoText('255, 255, 255');
```

Signature for all dynamic methods:

```php
$colorManager->{colorName}(string $value, int|string|null $shade = null, bool $dark = false)
```

The method name is converted to kebab-case internally (e.g., `successBg` becomes `success-bg`).

### Color conversion

`MoonShine\ColorManager\ColorMutator` converts between HEX and RGB:

```php
use MoonShine\ColorManager\ColorMutator;

// RGB to HEX
ColorMutator::toHEX('120, 67, 233');  // '#7843e9'
ColorMutator::toHEX('#7843e9');        // '#7843e9' (passthrough)

// HEX to RGB
ColorMutator::toRGB('#7843e9');        // '120,67,233'
ColorMutator::toRGB('120, 67, 233');   // '120, 67, 233' (passthrough)
ColorMutator::toRGB('rgb(120,67,233)'); // '120,67,233'
```

### HTML output

Generate CSS custom properties for both themes:

```php
$colorManager->toHtml();
```

Produces:

```html
<style>
    :root {
        --primary:120,67,233;
        --secondary:236,65,118;
        --body:27,37,59;
        --dark-DEFAULT:30,31,67;
        --dark-50:83,103,132;
        /* ... all light theme variables */
    }
    :root.dark {
        --body:27,37,59;
        --success-bg:17,157,17;
        /* ... all dark theme variables */
    }
</style>
```

### Global override

Override colors globally via `MoonShineServiceProvider`:

```php
use MoonShine\Contracts\ColorManager\ColorManagerContract;

public function boot(ColorManagerContract $colors): void
{
    $colors->primary('#7843e9');
    $colors->secondary('#E64176');
}
```

> Layout `colors()` method loads after ServiceProvider and takes precedence. Ensure your ServiceProvider colors are not overridden in the layout.

### Layout color configuration

Define colors per-layout in the `colors()` method:

```php
use MoonShine\Contracts\ColorManager\ColorManagerContract;

protected function colors(ColorManagerContract $colorManager): void
{
    // Light theme
    $colorManager
        ->primary('#1E96FC')
        ->secondary('#1D8A99')
        ->body('249, 250, 251')
        ->dark('30, 31, 67', 'DEFAULT')
        ->dark('249, 250, 251', 50)
        ->dark('243, 244, 246', 100)
        ->dark('229, 231, 235', 200)
        ->dark('209, 213, 219', 300)
        ->dark('156, 163, 175', 400)
        ->dark('107, 114, 128', 500)
        ->dark('75, 85, 99', 600)
        ->dark('55, 65, 81', 700)
        ->dark('31, 41, 55', 800)
        ->dark('17, 24, 39', 900)
        ->successBg('209, 255, 209')
        ->successText('15, 99, 15')
        ->warningBg('255, 246, 207')
        ->warningText('92, 77, 6')
        ->errorBg('255, 224, 224')
        ->errorText('81, 20, 20')
        ->infoBg('196, 224, 255')
        ->infoText('34, 65, 124');

    // Dark theme
    $colorManager
        ->body('27, 37, 59', dark: true)
        ->dark('83, 103, 132', 50, dark: true)
        ->dark('74, 90, 121', 100, dark: true)
        ->dark('65, 81, 114', 200, dark: true)
        ->dark('53, 69, 103', 300, dark: true)
        ->dark('48, 61, 93', 400, dark: true)
        ->dark('41, 53, 82', 500, dark: true)
        ->dark('40, 51, 78', 600, dark: true)
        ->dark('39, 45, 69', 700, dark: true)
        ->dark('27, 37, 59', 800, dark: true)
        ->dark('15, 23, 42', 900, dark: true)
        ->successBg('17, 157, 17', dark: true)
        ->successText('178, 255, 178', dark: true)
        ->warningBg('225, 169, 0', dark: true)
        ->warningText('255, 255, 199', dark: true)
        ->errorBg('190, 10, 10', dark: true)
        ->errorText('255, 197, 197', dark: true)
        ->infoBg('38, 93, 205', dark: true)
        ->infoText('179, 220, 255', dark: true);
}
```
