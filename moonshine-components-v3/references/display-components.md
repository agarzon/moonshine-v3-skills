# Display Components

## Badge

**Class:** `MoonShine\UI\Components\Badge`
**Blade:** `<x-moonshine::badge>`

Colored label/tag element.

```php
Badge::make(
    string $value = '',
    string|Color $color = Color::PURPLE
)
```

```php
use MoonShine\Support\Enums\Color;

Badge::make('New', Color::SUCCESS)
Badge::make('Draft', Color::WARNING)
Badge::make('Custom', 'pink')
```

Available colors (Enum `MoonShine\Support\Enums\Color`):
`PRIMARY`, `SECONDARY`, `SUCCESS`, `INFO`, `WARNING`, `ERROR`, `PURPLE`, `PINK`, `BLUE`, `GREEN`, `YELLOW`, `RED`, `GRAY`

Or as strings: `'primary'`, `'secondary'`, `'success'`, `'info'`, `'warning'`, `'error'`, `'purple'`, `'pink'`, `'blue'`, `'green'`, `'yellow'`, `'red'`, `'gray'`

**Blade:**
```blade
<x-moonshine::badge color="success">Active</x-moonshine::badge>
<x-moonshine::badge color="error">Deleted</x-moonshine::badge>
```

---

## Boolean

**Class:** `MoonShine\UI\Components\Boolean`
**Blade:** `<x-moonshine::boolean>`

Visual true/false indicator (green check / red cross).

```php
Boolean::make(true)
Boolean::make(false)
```

**Blade:**
```blade
<x-moonshine::boolean :value="true" />
<x-moonshine::boolean :value="false" />
```

---

## Color

**Class:** `MoonShine\UI\Components\Color`
**Blade:** `<x-moonshine::color>`

Displays a colored `<div>` block.

```php
use MoonShine\Support\Enums\Color as ColorEnum;

Color::make(ColorEnum::PURPLE)
Color::make('red')
```

**Blade:**
```blade
<x-moonshine::color :color="'red'" />
```

Uses the same `Color` enum as Badge.

---

## Alert

**Class:** `MoonShine\UI\Components\Alert`
**Blade:** `<x-moonshine::alert>`

Page notification banner.

```php
Alert::make(type: 'success', icon: 'check-circle', removable: true)
    ->content('Operation completed successfully!')
```

Types: `'primary'`, `'secondary'`, `'success'`, `'warning'`, `'error'`, `'info'`

```php
Alert::make(type: 'primary')->content('Primary alert')
Alert::make(type: 'error')->content('Something went wrong')
Alert::make(icon: 'academic-cap')->content('With custom icon')
Alert::make(removable: true)->content('Auto-dismissible')
```

**Blade:**
```blade
<x-moonshine::alert type="success">Record saved!</x-moonshine::alert>
<x-moonshine::alert type="warning" removable="true">Warning message</x-moonshine::alert>
<x-moonshine::alert type="error" icon="exclamation-triangle">Error occurred</x-moonshine::alert>
```

---

## Files

**Class:** `MoonShine\UI\Components\Files`
**Blade:** `<x-moonshine::files>`

Displays a list of downloadable files.

```php
Files::make(
    array $files = [],
    bool $download = true,
)
```

```php
Files::make([
    '/uploads/document.pdf',
    '/uploads/spreadsheet.xlsx',
])

// Without download links
Files::make($files, download: false)
```

**Blade:**
```blade
<x-moonshine::files
    :files="['/uploads/doc.pdf', '/uploads/sheet.xlsx']"
    :download="true"
/>
```

---

## Thumbnails

**Class:** `MoonShine\UI\Components\Thumbnails`
**Blade:** `<x-moonshine::thumbnails>`

Image thumbnail(s) display.

```php
Thumbnails::make(FileItem|string|array|null $items)
```

```php
// Multiple thumbnails
Thumbnails::make([
    '/images/photo1.jpg',
    '/images/photo2.jpg',
])

// Single thumbnail
Thumbnails::make('/images/photo.jpg')
```

**Blade:**
```blade
<x-moonshine::thumbnails
    :items="['/images/img1.jpg', '/images/img2.jpg']"
/>

<x-moonshine::thumbnails
    :items="'/images/single.jpg'"
    alt="Photo description"
/>
```

---

## ProgressBar

**Class:** `MoonShine\UI\Components\ProgressBar`
**Blade:** `<x-moonshine::progress-bar>`

Progress indicator bar.

```php
ProgressBar::make(
    float|int $value,
    string $size = 'sm',
    string|Color $color = '',
    bool $radial = false,
)
```

```php
ProgressBar::make(75)
ProgressBar::make(50, size: 'lg', color: 'success')
ProgressBar::make(30, radial: true)
```

Sizes: `'sm'`, `'md'`, `'lg'`, `'xl'`
Colors: `'primary'`, `'secondary'`, `'success'`, `'warning'`, `'error'`, `'info'`

**Blade:**
```blade
<x-moonshine::progress-bar color="primary" :value="75">75%</x-moonshine::progress-bar>
```

---

## Rating

**Class:** `MoonShine\UI\Components\Rating`
**Blade:** `<x-moonshine::rating>`

Star-style rating display.

```php
Rating::make(
    int $value,
    int $min = 1,
    int $max = 5,
)
```

```php
Rating::make(4)              // 4 out of 5
Rating::make(8, min: 1, max: 10)  // 8 out of 10
```

**Blade:**
```blade
<x-moonshine::rating value="4" min="1" max="5" />
```

---

## Carousel

**Class:** `MoonShine\UI\Components\Carousel`
**Blade:** `<x-moonshine::carousel>`

Image carousel/slider.

```php
Carousel::make(
    Closure|array $items = [],
    Closure|bool $portrait = false,
    Closure|string $alt = ''
)
```

```php
Carousel::make(
    items: ['/images/slide1.jpg', '/images/slide2.jpg', '/images/slide3.jpg'],
    alt: 'Gallery'
)

// Portrait orientation
Carousel::make(items: $images, portrait: true)

// Set items via fluent method
Carousel::make(alt: 'Photos')->items(['/img/1.jpg', '/img/2.jpg'])
```

**Blade:**
```blade
<x-moonshine::carousel
    :items="['/images/1.jpg', '/images/2.jpg']"
    :alt="'Photo gallery'"
/>
```

---

## Link

**Class:** `MoonShine\UI\Components\Link`
**Blade:** `<x-moonshine::link-button>`, `<x-moonshine::link-native>`

Styled link component.

```php
Link::make(
    Closure|string $href,
    Closure|string $label = ''
)
```

```php
Link::make('https://example.com', 'Visit Site')
    ->icon('arrow-top-right-on-square')
    ->tooltip('Opens in browser')
    ->filled()
```

**Blade:**
```blade
<x-moonshine::link-button href="/url" :filled="true">
    Button-style Link
</x-moonshine::link-button>

<x-moonshine::link-native href="/url">
    Native-style Link
</x-moonshine::link-native>
```

---

## Url

**Class:** `MoonShine\UI\Components\Url`

URL display component (used internally by URL fields).

---

## Icon

**Class:** `MoonShine\UI\Components\Icon`
**Blade:** `<x-moonshine::icon>`

Renders an icon from the icon set.

```php
Icon::make(
    string $icon,
    int $size = 5,
    Color|string $color = '',
    ?string $path = null,
)
```

```php
Icon::make('users')
Icon::make('pencil', size: 8, color: Color::PRIMARY)

// Custom SVG icon
Icon::make(svg('path-to-icon-pack')->toHtml())->custom()
```

**Blade:**
```blade
<x-moonshine::icon icon="users" />
```

MoonShine uses Heroicons by default. Icon names are the Heroicons outline names without prefix (e.g., `'users'`, `'pencil'`, `'trash'`, `'shopping-bag'`, `'check-circle'`).

---

## Breadcrumbs

**Class:** `MoonShine\UI\Components\Breadcrumbs`
**Blade:** `<x-moonshine::breadcrumbs>`

Navigation breadcrumb trail.

```php
Breadcrumbs::make(array $items = [])
```

`$items` is an associative array: `URL => Label`.

```php
Breadcrumbs::make([
    '/' => 'Home',
    '/articles' => 'Articles',
    '/articles/42' => 'My Article',
])
```

**Blade:**
```blade
<x-moonshine::breadcrumbs
    :items="['/' => 'Home', '/articles' => 'Articles']"
/>
```

---

## Spinner

**Class:** `MoonShine\UI\Components\Spinner`
**Blade:** `<x-moonshine::spinner>`

Loading indicator (animated spinner).

```php
Spinner::make(
    string $size = 'sm',
    string|Color $color = '',
    bool $fixed = false,
    bool $absolute = false,
)
```

```php
Spinner::make()
Spinner::make(size: 'lg', color: 'primary')
Spinner::make(fixed: true)    // fixed position on screen
Spinner::make(absolute: true) // absolute position in container
```

Sizes: `'sm'`, `'md'`, `'lg'`, `'xl'`
Colors: `'primary'`, `'secondary'`, `'success'`, `'warning'`, `'error'`, `'info'`

**Blade:**
```blade
<x-moonshine::spinner size="md" color="primary" />
<x-moonshine::spinner :fixed="true" />
<x-moonshine::spinner :absolute="true" />
```

---

## Loader

**Class:** `MoonShine\UI\Components\Loader`
**Blade:** `<x-moonshine::loader>`

Styled loading indicator (different visual from Spinner).

```php
Loader::make()
```

### Custom View

Globally change the loader view:

```php
// In ServiceProvider
Loader::changeView('my-custom-loader-view');
```

**Blade:**
```blade
<x-moonshine::loader />
```

---

## FlexibleRender

**Class:** `MoonShine\UI\Components\FlexibleRender`

Renders raw text, HTML, or Blade views inline as a component.

```php
FlexibleRender::make(
    Closure|Renderable|string $content,
    Closure|array $additionalData = []
)
```

```php
// Plain HTML
FlexibleRender::make('<p>Hello World</p>')

// Blade view
FlexibleRender::make(view('partials.info'))

// View with data
FlexibleRender::make(view('partials.info', ['key' => 'value']))
FlexibleRender::make(view('partials.info'), ['key' => 'value'])

// Closure with data
FlexibleRender::make(
    fn($data) => view('partials.info', $data),
    fn() => ['key' => 'value']
)
```

---

## Metrics

### ValueMetric

**Class:** `MoonShine\UI\Components\Metrics\Wrapped\ValueMetric`
**Blade:** `<x-moonshine::metrics.value>`

Displays a numeric value, optionally with progress bar and icon.

```php
ValueMetric::make(Closure|string $label)
```

```php
// Basic value
ValueMetric::make('Total Users')
    ->value(fn() => User::count())

// With icon
ValueMetric::make('Orders')
    ->value(fn() => Order::count())
    ->icon('shopping-bag')

// With progress (shows percentage bar)
ValueMetric::make('Completed Tasks')
    ->value(fn() => Task::completed()->count())
    ->progress(fn() => Task::count())

// Formatted value
ValueMetric::make('Revenue')
    ->value(fn() => Order::sum('total'))
    ->valueFormat(fn(int $value) => '$' . number_format($value))

// Column width in grid
ValueMetric::make('Users')
    ->value(fn() => User::count())
    ->columnSpan(4, adaptiveColumnSpan: 12)
```

**Blade:**
```blade
<x-moonshine::metrics.value
    title="Total Orders"
    icon="shopping-bag"
    :value="$orderCount"
    :progress="false"
/>
```

### Dashboard Metrics Layout

```php
Grid::make([
    Column::make([
        ValueMetric::make('Orders')
            ->value(fn() => Order::count())
            ->icon('shopping-bag')
            ->columnSpan(12)
    ], colSpan: 4),

    Column::make([
        ValueMetric::make('Revenue')
            ->value(fn() => Order::sum('price'))
            ->valueFormat(fn($v) => Number::forHumans($v))
            ->icon('currency-dollar')
            ->columnSpan(12)
    ], colSpan: 4),

    Column::make([
        ValueMetric::make('Completion Rate')
            ->value(fn() => Order::where('status', 'done')->count())
            ->progress(fn() => Order::count())
            ->icon('chart-bar')
            ->columnSpan(12)
    ], colSpan: 4),
])
```

### DonutChartMetric / LineChartMetric

These require the separate `moonshine/apexcharts` package based on [ApexCharts](https://apexcharts.com/).

```shell
composer require moonshine/apexcharts
```

See the [official ApexCharts repository](https://github.com/moonshine-software/apexcharts) for API documentation.

---

## ActionGroup

**Class:** `MoonShine\UI\Components\ActionGroup`

Groups multiple `ActionButton` instances with visibility control.

```php
ActionGroup::make(iterable $actions = [])
```

```php
ActionGroup::make([
    ActionButton::make('Edit', '/edit')->showInLine(),
    ActionButton::make('More', '#')->showInDropdown(),
    ActionButton::make('Secret', '/secret')->canSee(fn() => false),
])

// Fill all buttons with data
ActionGroup::make($buttons)->fill($dataWrapper)

// Add buttons
ActionGroup::make($buttons)
    ->add(ActionButton::make('Extra'))
    ->prepend(ActionButton::make('First'))
    ->addMany([ActionButton::make('A'), ActionButton::make('B')])
```

Display modes per button:
- `->showInLine()` -- always visible inline
- `->showInDropdown()` -- placed in "..." dropdown menu
