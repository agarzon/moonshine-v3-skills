# Menu System Reference

Complete API reference for MoonShine v3 menu system.

---

## Table of Contents

- [Overview](#overview)
- [MenuItem](#menuitem)
- [MenuGroup](#menugroup)
- [MenuDivider](#menudivider)
- [MenuElement (base class)](#menuelement)
- [MenuManager](#menumanager)
- [Menu autoloading](#menu-autoloading)
- [Menu autoload attributes](#menu-autoload-attributes)
- [Menu authorization](#menu-authorization)
- [Icon assignment](#icon-assignment)
- [Badges](#badges)
- [Translation](#translation)
- [Active state](#active-state)
- [Custom views](#custom-views)
- [Change button](#change-button)
- [Custom attributes](#custom-attributes)

---

## Overview

The menu system consists of three element types and a manager:

- `MoonShine\MenuManager\MenuItem` -- a single navigable link
- `MoonShine\MenuManager\MenuGroup` -- a collapsible group of items
- `MoonShine\MenuManager\MenuDivider` -- a visual separator
- `MoonShine\MenuManager\MenuManager` -- programmatic menu manipulation

All elements extend `MoonShine\MenuManager\MenuElement` (abstract).

The menu is typically defined in the layout's `menu()` method and automatically injected into the `Menu` component.

## MenuItem

### Constructor / make()

```php
MenuItem::make(
    Closure|string $label,
    Closure|MenuFillerContract|string $filler,
    ?string $icon = null,
    Closure|bool $blank = false,
)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `$label` | `Closure\|string` | Display name of the menu item |
| `$filler` | `Closure\|MenuFillerContract\|string` | URL source: Resource class, Page class, URL string, or Closure |
| `$icon` | `?string` | Icon name (Heroicons or custom) |
| `$blank` | `Closure\|bool` | Open link in new tab |

### Accepted filler types

```php
// ModelResource -- generates URL to resource's first page
MenuItem::make('Users', MoonShineUserResource::class)

// Page -- generates URL to the page
MenuItem::make('Dashboard', DashboardPage::class)

// URL string
MenuItem::make('Docs', 'https://moonshine-laravel.com/docs')

// Closure returning URL
MenuItem::make('Home', fn() => route('home'))
```

### Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `icon()` | `icon(string $icon, bool $custom = false, ?string $path = null)` | Set icon |
| `badge()` | `badge(Closure\|string\|int\|float\|null $value)` | Add badge |
| `blank()` | `blank(Closure\|bool $blankCondition = true)` | Open in new tab |
| `translatable()` | `translatable(string $key = '')` | Enable translation |
| `canSee()` | `canSee(Closure $callback)` | Conditional display |
| `whenActive()` | `whenActive(Closure $when)` | Custom active-state logic |
| `setUrl()` | `setUrl(string\|Closure\|null $url, Closure\|bool $blank = false)` | Set URL directly |
| `getUrl()` | `getUrl(): string` | Resolve and return URL |
| `isActive()` | `isActive(): bool` | Check if item matches current URL |
| `isBlank()` | `isBlank(): bool` | Check if item opens in new tab |
| `getFiller()` | `getFiller(): MenuFillerContract\|Closure\|string` | Get the filler object |
| `changeButton()` | `changeButton(Closure $callback)` | Modify the underlying ActionButton |
| `customView()` | `customView(string $path)` | Use a custom Blade view |
| `customAttributes()` | `customAttributes(array $attributes)` | Set HTML attributes |

### Examples

```php
// Full-featured menu item
MenuItem::make('Comments', CommentResource::class, 'chat-bubble-left')
    ->badge(fn() => Comment::where('status', 'new')->count())
    ->canSee(fn() => auth()->user()->can('viewAny', Comment::class))
    ->whenActive(fn() => request()->fullUrlIs('*admin/comment*'))

// External link in new tab
MenuItem::make('Laravel', 'https://laravel.com', blank: true)

// With closure-based URL
MenuItem::make('Reports', fn() => route('admin.reports'))
```

## MenuGroup

### Constructor / make()

```php
MenuGroup::make(
    Closure|string $label,
    iterable $items = [],
    ?string $icon = null,
)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `$label` | `Closure\|string` | Group label |
| `$items` | `iterable` | Array of MenuItem, MenuGroup, or MenuDivider |
| `$icon` | `?string` | Icon name |

### Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `setItems()` | `setItems(iterable $items): static` | Replace group items |
| `getItems()` | `getItems(): MenuElementsContract` | Get items as collection |
| `isActive()` | `isActive(): bool` | True if any child is active |
| `icon()` | `icon(string $icon, bool $custom = false, ?string $path = null)` | Set icon |
| `canSee()` | `canSee(Closure $callback)` | Conditional display |
| `translatable()` | `translatable(string $key = '')` | Enable translation |
| `customView()` | `customView(string $path)` | Use a custom Blade view |
| `customAttributes()` | `customAttributes(array $attributes)` | Set HTML attributes |

### Examples

```php
// Basic group
MenuGroup::make('System', [
    MenuItem::make('Admins', MoonShineUserResource::class),
    MenuItem::make('Roles', MoonShineUserRoleResource::class),
], 'cog')

// Group with setItems (alternative syntax)
MenuGroup::make('Content')->setItems([
    MenuItem::make('Articles', ArticleResource::class),
    MenuItem::make('Categories', CategoryResource::class),
])->icon('newspaper')

// Nested groups (multi-level menu)
MenuGroup::make('Admin', [
    MenuGroup::make('Users', [
        MenuItem::make('All Users', UserResource::class),
        MenuItem::make('Roles', RoleResource::class),
    ]),
    MenuGroup::make('Settings', [
        MenuItem::make('General', SettingsPage::class),
    ]),
])

// Conditional group (only for admin users)
MenuGroup::make('System', [
    MenuItem::make('Admins', MoonShineUserResource::class),
])->canSee(fn() => request()->user('moonshine')?->id === 1)
```

## MenuDivider

### Constructor / make()

```php
MenuDivider::make(Closure|string $label = '')
```

A visual separator line. Optionally displays a label.

### Examples

```php
// Simple divider
MenuDivider::make()

// Labeled divider
MenuDivider::make('Settings')

// Conditional divider
MenuDivider::make()->canSee(fn() => true)
```

## MenuElement

Abstract base class for all menu elements. Provides shared traits:

### Traits included

- `Makeable` -- static `make()` factory
- `Macroable` -- Laravel macro support
- `WithCore` -- access to MoonShine core
- `WithComponentAttributes` -- HTML attribute management
- `WithIcon` -- icon support
- `HasCanSee` -- conditional visibility
- `WithLabel` -- label management
- `WithViewRenderer` -- Blade rendering
- `Conditionable` -- `when()` / `unless()` support

### Key methods from traits

```php
// Label
$item->setLabel('New Label');
$item->getLabel(): string;

// Icon
$item->icon('users');
$item->icon(svg('custom-icon')->toHtml(), custom: true);
$item->icon('cog', path: 'icons'); // resources/views/icons/cog.blade.php
$item->getIconValue(): string;

// Visibility
$item->canSee(fn() => true);
$item->isSee(): bool;

// Attributes
$item->customAttributes(['class' => 'highlight']);
$item->setAttribute('data-id', '123');
$item->class('my-class');

// Top mode (set automatically by Menu::top())
$item->topMode(true);
$item->isTopMode(): bool;
```

## MenuManager

Programmatic menu manipulation, typically used in ServiceProvider or packages.

Access via DI: `MoonShine\Contracts\MenuManager\MenuManagerContract`

### Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `add()` | `add(array\|MenuElementContract $data): static` | Add items to the menu |
| `remove()` | `remove(Closure $condition): static` | Remove items matching condition |
| `addBefore()` | `addBefore(Closure $before, ...$data): static` | Insert items before a matching element |
| `addAfter()` | `addAfter(Closure $after, ...$data): static` | Insert items after a matching element |
| `topMode()` | `topMode(?Closure $condition = null): static` | Enable top navigation mode |
| `all()` | `all(?iterable $items = null): MenuElementsContract` | Get resolved menu collection |
| `flushState()` | `flushState(): void` | Clear all items |

### Examples

```php
// In a package service provider
use MoonShine\Contracts\MenuManager\MenuManagerContract;

public function boot(MenuManagerContract $menuManager): void
{
    // Add item
    $menuManager->add(
        MenuItem::make('Package Page', PackagePage::class)
    );

    // Add before a specific element
    $menuManager->addBefore(
        fn(MenuElementContract $el) => $el instanceof MenuItem && $el->getLabel() === 'Settings',
        MenuItem::make('Before Settings', SomePage::class)
    );

    // Add after
    $menuManager->addAfter(
        fn(MenuElementContract $el) => $el instanceof MenuItem && $el->getLabel() === 'Dashboard',
        MenuDivider::make(),
        MenuItem::make('Reports', ReportsPage::class)
    );

    // Remove items
    $menuManager->remove(
        fn(MenuElementContract $el) => $el instanceof MenuItem && $el->getLabel() === 'Old Item'
    );
}
```

## Menu autoloading

Replace manual `menu()` array with auto-discovery:

```php
protected function menu(): array
{
    return $this->autoloadMenu();
}
```

Autoloading scans all registered resources and pages, creating menu items automatically based on their `Icon` attributes and labels.

## Menu autoload attributes

Control autoload behavior with PHP 8 attributes:

### SkipMenu

Exclude a resource or page from the auto-generated menu:

```php
use MoonShine\MenuManager\Attributes\SkipMenu;

#[SkipMenu]
class ProfilePage extends Page {}
```

### Group

Group resources/pages by label. Items with the same group label are placed together:

```php
use MoonShine\MenuManager\Attributes\Group;

#[Group('Content', 'newspaper', translatable: true)]
class ArticleResource extends ModelResource {}
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `$label` | `string` | Group label (or translation key if translatable) |
| `$icon` | `?string` | Group icon |
| `$translatable` | `bool` | Whether label is a translation key |

### Order

Control menu item ordering:

```php
use MoonShine\MenuManager\Attributes\Order;

#[Order(1)]
class ArticleResource extends ModelResource {}

#[Order(2)]
class CategoryResource extends ModelResource {}
```

### CanSee

Conditional display via a method on the resource/page:

```php
use MoonShine\MenuManager\Attributes\CanSee;

#[CanSee(method: 'canViewInMenu')]
class SecretResource extends ModelResource
{
    public function canViewInMenu(): bool
    {
        return auth()->user()->hasRole('admin');
    }
}
```

## Menu authorization

Use `canSee()` on any menu element for authorization:

```php
protected function menu(): array
{
    return [
        MenuGroup::make('System', [
            MenuItem::make('Admins', MoonShineUserResource::class),
            MenuItem::make('Roles', MoonShineUserRoleResource::class)
                ->canSee(fn() => false), // hidden
        ])->canSee(fn() => request()->user('moonshine')?->id === 1),

        MenuItem::make('Reports', ReportsPage::class)
            ->canSee(fn() => auth()->user()->can('view-reports')),

        MenuDivider::make()
            ->canSee(fn() => auth()->user()->isAdmin()),
    ];
}
```

Groups without visible items are automatically hidden (via `onlyVisible()` in `MenuElements`).

## Icon assignment

Icons can be set at three levels (in order of precedence):

### 1. Directly on menu item

```php
MenuItem::make('Users', UserResource::class, 'users')
// or
MenuItem::make('Users', UserResource::class)->icon('users')
```

### 2. Via resource/page Icon attribute

```php
use MoonShine\Support\Attributes\Icon;

#[Icon('users')]
class UserResource extends ModelResource {}
```

If no icon is set on the MenuItem, the resource's `#[Icon]` attribute is used automatically.

### 3. Custom icon with path

```php
MenuItem::make('Custom', SomePage::class)
    ->icon('my-icon', path: 'icons')
// Looks for resources/views/icons/my-icon.blade.php
```

### 4. Custom HTML icon

```php
MenuItem::make('Custom', SomePage::class)
    ->icon(svg('custom-icon-pack.star')->toHtml(), custom: true)
```

### Icon sets

MoonShine ships with Heroicons in four variants:

| Prefix | Set | Example |
|--------|-----|---------|
| (none) | Outline | `->icon('academic-cap')` |
| `s.` | Solid | `->icon('s.academic-cap')` |
| `m.` | Mini | `->icon('m.academic-cap')` |
| `c.` | Compact | `->icon('c.academic-cap')` |

## Badges

Add count badges to menu items:

```php
MenuItem::make('Comments', CommentResource::class)
    ->badge(fn() => Comment::where('status', 'new')->count())

// Static badge
MenuItem::make('New', SomePage::class)
    ->badge(42)

// Badge with translation
MenuItem::make('Items', ItemResource::class)
    ->badge(fn() => __('menu.badge.new'))
```

Resources with a `getBadge()` method automatically get badges when used as fillers.

## Translation

### Translate label

```php
// Using dot notation (file.key)
MenuItem::make('menu.Comments', CommentResource::class)
    ->translatable()

// With explicit key prefix
MenuItem::make('Comments', CommentResource::class)
    ->translatable('menu')
// Resolves to __('menu.Comments')
```

Translation file (`lang/en/menu.php`):

```php
return [
    'Comments' => 'Comments',
];
```

### Translate groups

```php
MenuGroup::make('menu.system', [/* ... */])
    ->translatable()
```

## Active state

By default, a menu item is active when the current URL matches. Override with `whenActive()`:

```php
MenuItem::make('Label', '/endpoint')
    ->whenActive(fn(string $path, string $host) => request()->fullUrlIs('*admin/endpoint/*'))
```

The closure receives:

| Parameter | Type | Description |
|-----------|------|-------------|
| `$path` | `string` | Parsed URL path of the menu item |
| `$host` | `string` | Parsed URL host of the menu item |
| `$context` | `MenuItem` | The menu item instance |

For Resource/Page fillers, `isActive()` delegates to the filler's own `isActive()` method unless `whenActive()` is set.

Groups are automatically active if any child item is active.

## Custom views

Replace the default Blade view for rendering:

```php
MenuItem::make('Label', '/endpoint')
    ->customView('admin.custom-menu-item')

MenuGroup::make('Group', [/* ... */])
    ->customView('admin.custom-menu-group')
```

## Change button

MenuItem renders via an internal `ActionButton`. Modify it:

```php
MenuItem::make('Label', '/endpoint')
    ->changeButton(fn(ActionButton $button) => $button->class('new-item'))
```

> `url`, `badge`, `icon` are overridden by the system. Use the corresponding MenuItem methods to change those.

## Custom attributes

```php
MenuItem::make('Roles', RoleResource::class)
    ->customAttributes(['class' => 'highlight'])

MenuGroup::make('System', [/* ... */])
    ->setAttribute('data-id', '123')
    ->class('group-custom')
```
