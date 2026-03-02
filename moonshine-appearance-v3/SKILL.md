---
name: moonshine-appearance-v3
description: Use when customizing MoonShine v3 layouts, menus, colors, icons, assets, pages, dark mode, branding, or overall admin panel design.
---

# MoonShine v3 Appearance

Covers layout templates, menu configuration, color theming, icons, asset management, dark mode, branding, and custom pages.

## Layout overview

MoonShine provides two built-in layout templates:

- **AppLayout** -- standard sidebar layout with full header, footer, sidebar, breadcrumbs
- **CompactLayout** -- minimalistic theme extending AppLayout, adds `theme-minimalistic` class and overrides colors for a lighter look

The selected layout is published at `app/MoonShine/Layouts/MoonShineLayout.php` and configured in `moonshine.layout`.

### Hierarchy

```
BaseLayout (abstract)
  -> AppLayout (standard)
       -> CompactLayout (compact/minimalistic)
  -> BlankLayout (bare bones -- Head + Body only)
  -> LoginLayout (authentication page)
```

### Choosing a layout

```php
// config/moonshine.php
'layout' => \MoonShine\Laravel\Layouts\AppLayout::class,

// Or via MoonShineServiceProvider
$config->layout(\App\MoonShine\Layouts\CustomLayout::class);
```

## Creating a custom layout

### Generate via artisan

```shell
php artisan moonshine:layout MyLayout
```

This creates `app/MoonShine/Layouts/MyLayout.php`.

### Extending an existing layout

```php
namespace App\MoonShine\Layouts;

use MoonShine\Laravel\Layouts\CompactLayout;
use MoonShine\UI\Components\Layout\Layout;

final class MoonShineLayout extends CompactLayout
{
    protected function getFooterMenu(): array
    {
        return [
            'https://example.com' => 'Custom link',
        ];
    }

    protected function getFooterCopyright(): string
    {
        return 'My Company';
    }

    public function build(): Layout
    {
        return parent::build();
    }
}
```

### Override component methods (quick customization)

Instead of rewriting `build()`, override individual component methods:

```php
protected function getHeadComponent(): Head { /* ... */ }
protected function getLogoComponent(): Logo { /* ... */ }
protected function getSidebarComponent(): Sidebar { /* ... */ }
protected function getHeaderComponent(): Header { /* ... */ }
protected function getTopBarComponent(): Topbar { /* ... */ }
protected function getFooterComponent(): Footer { /* ... */ }
protected function getProfileComponent(bool $sidebar = false): Profile { /* ... */ }
protected function getContentComponents(): array { /* ... */ }
protected function getLogo(bool $small = false): string { /* ... */ }
protected function getHomeUrl(): string { /* ... */ }
```

### Layout slots

Quickly inject components into Sidebar or TopBar without overriding `build()`:

```php
protected function sidebarSlot(): array
{
    return [
        Search::make()->enabled(),
    ];
}

protected function sidebarTopSlot(): array
{
    return [
        Notifications::make(),
    ];
}

protected function topBarSlot(): array
{
    return [
        // custom components in top bar
    ];
}
```

### Using TopBar instead of Sidebar

```php
public function build(): Layout
{
    return Layout::make([
        Html::make([
            $this->getHeadComponent(),
            Body::make([
                Wrapper::make([
                    $this->getTopBarComponent(),
                    // $this->getSidebarComponent(), // removed
                    Div::make([
                        Flash::make(),
                        $this->getHeaderComponent(),
                        Content::make([
                            Components::make($this->getPage()->getComponents()),
                        ]),
                        $this->getFooterComponent(),
                    ])->class('layout-page'),
                ]),
            ])->class('theme-minimalistic'),
        ])
            ->customAttributes(['lang' => $this->getHeadLang()])
            ->withAlpineJs()
            ->withThemes(),
    ]);
}
```

> If using both Sidebar and TopBar, TopBar must come first in the Wrapper.

See [references/layouts.md](references/layouts.md) for the full layout API.

## Menu setup quick start

The menu is declared in the layout's `menu()` method:

```php
use MoonShine\MenuManager\MenuItem;
use MoonShine\MenuManager\MenuGroup;
use MoonShine\MenuManager\MenuDivider;

protected function menu(): array
{
    return [
        MenuGroup::make('Content', [
            MenuItem::make('Articles', ArticleResource::class, 'document-text'),
            MenuItem::make('Categories', CategoryResource::class, 'tag'),
        ])->icon('newspaper'),

        MenuDivider::make(),

        MenuItem::make('Dashboard', DashboardPage::class, 'home'),

        MenuItem::make('External', 'https://example.com', blank: true),
    ];
}
```

### Key menu features

- **MenuItem** -- links to a Resource, Page, URL, or Closure
- **MenuGroup** -- groups items with an optional icon
- **MenuDivider** -- visual separator with optional label
- **badge()** -- show counts: `->badge(fn() => Comment::count())`
- **canSee()** -- conditional display: `->canSee(fn() => auth()->user()->isAdmin())`
- **translatable()** -- i18n support: `->translatable('menu')`
- **whenActive()** -- custom active state logic
- **icon()** -- Heroicons or custom SVGs

### Autoloaded menu

Replace manual menu declaration with automatic discovery:

```php
protected function menu(): array
{
    return $this->autoloadMenu();
}
```

Control autoload behavior with attributes on resources/pages:

```php
use MoonShine\MenuManager\Attributes\SkipMenu;
use MoonShine\MenuManager\Attributes\Group;
use MoonShine\MenuManager\Attributes\Order;
use MoonShine\MenuManager\Attributes\CanSee;

#[SkipMenu]
class ProfilePage extends Page {}

#[Group('Content', 'newspaper', translatable: true)]
#[Order(1)]
class ArticleResource extends ModelResource {}

#[CanSee(method: 'canViewInMenu')]
class SecretResource extends ModelResource
{
    public function canViewInMenu(): bool
    {
        return auth()->user()->isAdmin();
    }
}
```

See [references/menu-system.md](references/menu-system.md) for the full menu API.

## Color customization quick start

Override colors in the layout's `colors()` method:

```php
use MoonShine\Contracts\ColorManager\ColorManagerContract;

protected function colors(ColorManagerContract $colorManager): void
{
    $colorManager
        ->primary('#1E96FC')
        ->secondary('#1D8A99')
        ->body('249, 250, 251')
        ->dark('30, 31, 67', 'DEFAULT')
        ->dark('55, 65, 81', 700)
        ->dark('31, 41, 55', 800)
        ->dark('17, 24, 39', 900)
        ->successBg('209, 255, 209')
        ->successText('15, 99, 15')
        ->errorBg('255, 224, 224')
        ->errorText('81, 20, 20');

    // Dark theme overrides
    $colorManager
        ->body('27, 37, 59', dark: true)
        ->dark('39, 45, 69', 700, dark: true)
        ->dark('27, 37, 59', 800, dark: true)
        ->dark('15, 23, 42', 900, dark: true);
}
```

### Global color override (ServiceProvider)

```php
public function boot(ColorManagerContract $colors): void
{
    $colors->primary('#7843e9');
}
```

> Layout `colors()` loads after ServiceProvider and takes precedence.

### Semantic helpers

```php
$colorManager->background('27, 37, 59');   // body + dark.800 + dark body
$colorManager->content('39, 45, 69');      // dark.700 + dark.900
$colorManager->tableRow('40, 51, 78');     // dark.600
$colorManager->borders('53, 69, 103');     // dark.300
$colorManager->dropdowns('48, 61, 93');    // dark.400
$colorManager->buttons('83, 103, 132');    // dark.50 + dark.500 + dark.400
$colorManager->dividers('74, 90, 121');    // dark.100 + dark.200
```

See [references/colors.md](references/colors.md), [references/icons.md](references/icons.md), and [references/assets-branding.md](references/assets-branding.md) for the full color, icon, and asset API.

## Creating custom pages

### Generate a page

```shell
php artisan moonshine:page CustomPage
```

### Page structure

```php
namespace App\MoonShine\Pages;

use MoonShine\Laravel\Pages\Page;
use MoonShine\UI\Components\Layout\Box;
use MoonShine\UI\Components\Layout\Grid;
use MoonShine\UI\Components\Layout\Column;

class CustomPage extends Page
{
    protected string $title = 'Dashboard';
    protected string $subtitle = 'Overview';

    protected function components(): iterable
    {
        return [
            Grid::make([
                Column::make([
                    Box::make([/* ... */])
                ])->columnSpan(6),
                Column::make([
                    Box::make([/* ... */])
                ])->columnSpan(6),
            ])
        ];
    }
}
```

### Assign a layout to a specific page

```php
class CustomPage extends Page
{
    protected ?string $layout = MyLayout::class;
}

// Or via attribute
use MoonShine\Core\Attributes\Layout;

#[Layout(MyLayout::class)]
class CustomPage extends Page {}
```

### Modify layout dynamically from a page

```php
use MoonShine\Contracts\UI\LayoutContract;

protected function modifyLayout(LayoutContract $layout): LayoutContract
{
    return $layout->title('Custom Title')->description('Custom description');
}
```

### Page lifecycle hooks

- `onLoad()` -- runs when the page is the active route (add assets, authorize, etc.)
- `booted()` -- runs when MoonShine creates the page instance (early initialization)

## Dark mode setup

### Always-dark theme

```php
final class MoonShineLayout extends AppLayout
{
    protected function isAlwaysDark(): bool
    {
        return true;
    }
}
```

### Theme switcher

The `ThemeSwitcher` component is included by default in `getSidebarComponent()` and `getTopBarComponent()`. It toggles the `.dark` class on the root `<html>` element.

### Forcing dark mode on components inside light-themed areas

Sidebar, TopBar, and MobileBar are styled with dark colors by default. If you add custom components to them, force dark mode with:

```php
$this->getSidebarComponent()->class('dark'),
$this->getTopBarComponent()->class('dark'),
```

## Assets quick start

Add custom CSS/JS to a layout:

```php
use MoonShine\AssetManager\Css;
use MoonShine\AssetManager\Js;
use MoonShine\AssetManager\InlineCss;

protected function assets(): array
{
    return [
        ...parent::assets(),
        Css::make('/css/custom.css'),
        Js::make('/js/custom.js')->defer(),
        InlineCss::make(':root { --radius: 0.15rem; }'),
    ];
}
```

### Vite integration

```php
use Illuminate\Support\Facades\Vite;
use MoonShine\AssetManager\Js;

protected function assets(): array
{
    return [
        Js::make(Vite::asset('resources/js/app.js')),
    ];
}
```

## Branding

### Logo

```php
protected function getLogo(bool $small = false): string
{
    return $small ? '/images/logo-small.png' : '/images/logo.png';
}
```

Or via config:

```php
// config/moonshine.php
'logo' => '/images/logo.png',
'logo_small' => '/images/logo-small.png',
```

### Favicon

```php
protected function getFaviconComponent(): Favicon
{
    return parent::getFaviconComponent()->customAssets([
        'apple-touch' => '/favicon/apple-touch-icon.png',
        '32' => '/favicon/favicon-32x32.png',
        '16' => '/favicon/favicon-16x16.png',
        'safari-pinned-tab' => '/favicon/safari-pinned-tab.svg',
        'web-manifest' => '/favicon/site.webmanifest',
    ]);
}
```

### Footer

```php
protected function getFooterMenu(): array
{
    return ['https://docs.example.com' => 'Docs'];
}

protected function getFooterCopyright(): string
{
    return sprintf('&copy; %d My Company', now()->year);
}
```

## Blade templates

Layouts can also be written in pure Blade using `<x-moonshine::layout.*>` components. Key components: `layout`, `layout.html`, `layout.head`, `layout.body`, `layout.wrapper`, `layout.sidebar`, `layout.header`, `layout.content`, `layout.menu`, `layout.logo`, `layout.theme-switcher`, `layout.burger`, `layout.assets`, `layout.favicon`.

See [references/layouts.md](references/layouts.md) for the full Blade component reference.

## Cross-references

- **moonshine-setup-v3** -- Installation, configuration, routing, and initial bootstrap
- **moonshine-components-v3** -- UI components used inside layouts (Box, Grid, Column, Card, etc.)
- **moonshine-resources-v3** -- ModelResource classes that populate menu items and pages
- **moonshine-fields-v3** -- Field types used within page components
