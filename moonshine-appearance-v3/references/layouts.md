# Layouts Reference

Complete API reference for MoonShine v3 layout system.

---

## Table of Contents

- [Layout class hierarchy](#layout-class-hierarchy)
- [BaseLayout methods](#baselayout-methods)
- [AppLayout](#applayout)
- [CompactLayout](#compactlayout)
- [BlankLayout](#blanklayout)
- [LoginLayout](#loginlayout)
- [Building a layout from scratch](#building-from-scratch)
- [Layout slots](#layout-slots)
- [Page-specific layout assignment](#page-layout-assignment)
- [Modifying layouts from pages](#modifying-layouts)
- [Layout components reference](#layout-components)
- [Dark mode in layouts](#dark-mode)
- [Blade layout components](#blade-components)

---

## Layout class hierarchy

```
MoonShine\UI\AbstractLayout
  └── MoonShine\Laravel\Layouts\BaseLayout (abstract)
        ├── MoonShine\Laravel\Layouts\AppLayout
        │     └── MoonShine\Laravel\Layouts\CompactLayout
        ├── MoonShine\Laravel\Layouts\BlankLayout (final)
        └── MoonShine\Laravel\Layouts\LoginLayout (final)
```

All layout classes live in `MoonShine\Laravel\Layouts\`.

## BaseLayout methods

`BaseLayout` is the abstract base providing all overridable component methods.

### Component getters

| Method | Returns | Purpose |
|--------|---------|---------|
| `getHeadComponent()` | `Head` | `<head>` with CSRF meta, favicon, assets |
| `getLogoComponent()` | `Logo` | Logo with home URL, large and small variants |
| `getSidebarComponent()` | `Sidebar` | Full sidebar: logo, theme switcher, burger, menu, profile |
| `getTopBarComponent()` | `TopBar` | Top navigation bar: logo, menu (top mode), profile, theme switcher |
| `getHeaderComponent()` | `Header` | Breadcrumbs, search, notifications, locales |
| `getFooterComponent()` | `Footer` | Copyright and footer menu links |
| `getProfileComponent(bool $sidebar = false)` | `Profile` | User profile widget |
| `getFaviconComponent()` | `Favicon` | Favicon link tags |
| `getSearchComponent()` | `ComponentContract` | Search input |
| `getContentComponents()` | `array` | Page components wrapped with optional Title/Heading |

### Configuration getters

| Method | Returns | Purpose |
|--------|---------|---------|
| `getLogo(bool $small = false)` | `string` | Path to logo image |
| `getHomeUrl()` | `string` | Home page URL |
| `getHeadLang()` | `string` | HTML lang attribute value |
| `getFooterMenu()` | `array` | Footer links as `['url' => 'label']` |
| `getFooterCopyright()` | `string` | Copyright HTML string |

### Boolean flags

| Method | Default | Purpose |
|--------|---------|---------|
| `isAlwaysDark()` | `false` | Force dark theme |
| `isAuthEnabled()` | from config | Whether authentication is enabled |
| `isProfileEnabled()` | from config | Whether to show profile widget |
| `isUseNotifications()` | from config | Whether to show notifications |
| `withTitle()` | `true` | Whether to render page title above content |
| `withSubTitle()` | `true` | Whether to render page subtitle |

### Overridable hooks

| Method | Signature | Purpose |
|--------|-----------|---------|
| `menu()` | `protected function menu(): array` | Return array of MenuItem/MenuGroup/MenuDivider |
| `assets()` | `protected function assets(): array` | Return array of asset elements (Css, Js, etc.) |
| `colors()` | `protected function colors(ColorManagerContract $colorManager): void` | Configure color scheme |
| `build()` | `public function build(): Layout` | Construct the full component tree |

### Constants

```php
BaseLayout::CONTENT_FRAGMENT_NAME = '_content'
BaseLayout::CONTENT_ID = '_moonshine-content'
```

## AppLayout

Standard layout with sidebar navigation. Extends `BaseLayout`.

### Default menu

```php
protected function menu(): array
{
    return [
        MenuGroup::make(static fn () => __('moonshine::ui.resource.system'), [
            MenuItem::make(
                static fn () => __('moonshine::ui.resource.admins_title'),
                MoonShineUserResource::class
            ),
            MenuItem::make(
                static fn () => __('moonshine::ui.resource.role_title'),
                MoonShineUserRoleResource::class
            ),
        ]),
    ];
}
```

### build() structure

```
Layout
  Html [lang=..., withAlpineJs, withThemes]
    Head [csrf-token meta, favicon, assets]
    Body
      Wrapper
        Sidebar [collapsed]
          menu-heading (logo, theme switcher, burger)
          menu (menu items, profile)
        Div.flex.grow.overflow-auto
          Fragment._content.layout-page
            Flash
            Header (breadcrumbs, search, notifications, locales)
            Content (page components)
            Footer (copyright, menu)
```

### Customization pattern

```php
final class MoonShineLayout extends AppLayout
{
    protected function menu(): array
    {
        return [
            ...parent::menu(),
            MenuItem::make('Articles', ArticleResource::class),
        ];
    }

    protected function getFooterCopyright(): string
    {
        return 'My App';
    }

    public function build(): Layout
    {
        return parent::build();
    }
}
```

## CompactLayout

Minimalistic theme extending `AppLayout`. Key differences:

- Adds `theme-minimalistic` class to `Body`
- Loads compact theme CSS via `$this->getCompactThemeCss()`
- Overrides light and dark color palettes
- Sets `withTitle()` and `withSubTitle()` to `false`

### Default colors (light)

```php
primary: '#1E96FC'
secondary: '#1D8A99'
body: '249, 250, 251'
dark.DEFAULT: '30, 31, 67'
dark.50-900: light gray gradient
successBg: '209, 255, 209'
```

### Default colors (dark)

```php
body: '27, 37, 59'
dark.50-900: blue-gray gradient
successBg: '17, 157, 17'
```

### Rounded radius customization

```php
protected function assets(): array
{
    return [
        ...parent::assets(),
        InlineCss::make(<<<'Style'
            :root {
              --radius: 0.15rem;
              --radius-sm: 0.075rem;
              --radius-md: 0.275rem;
              --radius-lg: 0.3rem;
              --radius-xl: 0.4rem;
              --radius-2xl: 0.5rem;
              --radius-3xl: 1rem;
              --radius-full: 9999px;
            }
        Style),
    ];
}
```

## BlankLayout

Minimal layout with only `Head` and `Body`. No sidebar, header, footer, or wrapper. Useful for fully custom or embedded pages.

```php
// Source structure
Layout
  Html [lang=..., withAlpineJs, withThemes]
    Head
    Body
      Components (raw page components)
```

Usage:

```php
class EmbeddedPage extends Page
{
    protected ?string $layout = BlankLayout::class;
}
```

## LoginLayout

Authentication page layout. Provides `title()` and `description()` methods.

### API

```php
$layout->title(string $title): self
$layout->description(string $description): self
$layout->getTitle(): string    // defaults to moonshine::ui.login.title
$layout->getDescription(): string  // defaults to moonshine::ui.login.description
```

### Structure

```
Layout
  Html
    Head
    Body
      Div.authentication
        Div.authentication-logo (Logo)
        Div.authentication-content
          Div.authentication-header (Heading + description)
          Components (login form, etc.)
        ...pushed components
```

### Modifying from a page

```php
protected function modifyLayout(LayoutContract $layout): LayoutContract
{
    return $layout->title('Two-Factor Auth')->description('Enter your code');
}
```

## Building a layout from scratch

Everything in MoonShine is a component, including HTML tags. A full custom layout:

```php
namespace App\MoonShine\Layouts;

use MoonShine\Laravel\Layouts\BaseLayout;
use MoonShine\UI\Components\Components;
use MoonShine\UI\Components\Layout\{
    Assets, Body, Content, Div, Favicon, Flash,
    Footer, Head, Header, Html, Layout, Logo,
    Menu, Meta, Sidebar, ThemeSwitcher, Wrapper
};

final class CustomLayout extends BaseLayout
{
    protected function menu(): array
    {
        return [
            // your menu items
        ];
    }

    public function build(): Layout
    {
        return Layout::make([
            Html::make([
                $this->getHeadComponent(),
                Body::make([
                    Wrapper::make([
                        $this->getSidebarComponent(),
                        Div::make([
                            Flash::make(),
                            $this->getHeaderComponent(),
                            Content::make($this->getContentComponents()),
                            $this->getFooterComponent(),
                        ])->class('layout-page'),
                    ]),
                ]),
            ])
                ->customAttributes(['lang' => $this->getHeadLang()])
                ->withAlpineJs()
                ->withThemes($this->isAlwaysDark()),
        ]);
    }
}
```

## Layout slots

Slots allow injecting components into predefined areas without overriding `build()`:

| Slot method | Injection point |
|------------|----------------|
| `sidebarSlot()` | Before `Menu` inside sidebar menu area |
| `sidebarTopSlot()` | After `ThemeSwitcher` in sidebar heading actions |
| `topBarSlot()` | Before `Profile` in top bar actions area |

```php
protected function sidebarSlot(): array
{
    return [Search::make()->enabled()];
}

protected function sidebarTopSlot(): array
{
    return [Notifications::make()];
}

protected function topBarSlot(): array
{
    return [MyCustomComponent::make()];
}
```

## Page-specific layout assignment

### Via property

```php
class CustomPage extends Page
{
    protected ?string $layout = MyLayout::class;
}
```

### Via attribute

```php
use MoonShine\Core\Attributes\Layout;

#[Layout(MyLayout::class)]
class CustomPage extends Page {}
```

### Via configuration

```php
// config/moonshine.php
'layout' => \App\MoonShine\Layouts\MoonShineLayout::class,
```

## Modifying layouts from pages

The `modifyLayout()` method on a Page gives access to the layout instance after creation:

```php
use MoonShine\Contracts\UI\LayoutContract;

protected function modifyLayout(LayoutContract $layout): LayoutContract
{
    // Modify the layout before rendering
    return $layout;
}
```

This is particularly useful for LoginLayout modifications:

```php
/**
 * @param LoginLayout $layout
 */
protected function modifyLayout(LayoutContract $layout): LayoutContract
{
    return $layout
        ->title(__('Two-Factor Authentication'))
        ->description(__('Please enter your verification code'));
}
```

## Layout components reference

Key UI components used in layouts (all under `MoonShine\UI\Components\Layout\`):

| Component | Purpose |
|-----------|---------|
| `Layout` | Root layout wrapper |
| `Html` | `<html>` tag with `withAlpineJs()`, `withThemes()` |
| `Head` | `<head>` tag with `bodyColor()`, `title()` |
| `Meta` | `<meta>` tag |
| `Body` | `<body>` tag |
| `Wrapper` | Main content wrapper |
| `Sidebar` | Sidebar with `collapsed()` support |
| `TopBar` | Top navigation bar |
| `Header` | Page header area |
| `Content` | Main content area |
| `Footer` | Page footer with `copyright()`, `menu()` |
| `Menu` | Navigation menu, supports `top()` mode |
| `Logo` | Logo with `minimized()` support |
| `ThemeSwitcher` | Dark/light toggle |
| `Burger` | Mobile menu toggle |
| `Flash` | Flash message display |
| `Favicon` | Favicon links with `customAssets()`, `bodyColor()` |
| `Assets` | CSS/JS asset inclusion |
| `Div` | Generic `<div>` wrapper |

Laravel-specific components (under `MoonShine\Laravel\Components\Layout\`):

| Component | Purpose |
|-----------|---------|
| `Search` | Global search input |
| `Notifications` | Notification bell |
| `Profile` | User profile widget |
| `Locales` | Language switcher |

## Dark mode in layouts

### Always dark

```php
protected function isAlwaysDark(): bool
{
    return true;
}
```

This passes `true` to `Html::withThemes($alwaysDark)`.

### Force dark on specific components

Components inside Sidebar/TopBar that change with theme should be forced dark:

```php
$this->getSidebarComponent()->class('dark'),
$this->getTopBarComponent()->class('dark'),
```

## Blade layout components

All layout components have Blade equivalents:

```blade
<x-moonshine::layout />
<x-moonshine::layout.html :with-alpine-js="true" :with-themes="true" />
<x-moonshine::layout.head />
<x-moonshine::layout.meta name="csrf-token" :content="csrf_token()" />
<x-moonshine::layout.favicon />
<x-moonshine::layout.assets />
<x-moonshine::layout.body />
<x-moonshine::layout.wrapper />
<x-moonshine::layout.sidebar :collapsed="true" />
<x-moonshine::layout.div class="menu-heading" />
<x-moonshine::layout.logo href="/" logo="/logo.png" :minimized="true" />
<x-moonshine::layout.theme-switcher />
<x-moonshine::layout.burger />
<x-moonshine::layout.menu :elements="$menuArray" />
<x-moonshine::layout.header />
<x-moonshine::layout.content />
<x-moonshine::layout.search placeholder="Search" />
<x-moonshine::layout.locales :locales="collect()" />
<x-moonshine::breadcrumbs :items="['#' => 'Home']" />
```

### Blade assets for themes

Default theme:

```blade
<x-moonshine::layout.assets>
    @vite([
        'resources/css/main.css',
        'resources/js/app.js',
    ], 'vendor/moonshine')
</x-moonshine::layout.assets>
```

Compact theme:

```blade
<x-moonshine::layout.assets>
    @vite([
        'resources/css/main.css',
        'resources/css/minimalistic.css',
        'resources/js/app.js',
    ], 'vendor/moonshine')
</x-moonshine::layout.assets>
```
