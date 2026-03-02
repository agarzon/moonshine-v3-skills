# Icon System Reference

MoonShine v3 icon system using Heroicons and custom SVGs.

---

## Icon system

MoonShine uses the [Heroicons](https://heroicons.com) icon set by default. Custom SVGs are also supported.

### Heroicons sets

| Prefix | Variant | Example |
|--------|---------|---------|
| (none) | Outline (24x24, stroke) | `->icon('academic-cap')` |
| `s.` | Solid (24x24, filled) | `->icon('s.academic-cap')` |
| `m.` | Mini (20x20) | `->icon('m.academic-cap')` |
| `c.` | Compact (16x16) | `->icon('c.academic-cap')` |

### icon() method API

Available on MenuItems, MenuGroups, Resources, Pages, Components, Fields, and ActionButtons:

```php
icon(
    string $icon,
    bool $custom = false,
    ?string $path = null,
)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `$icon` | `string` | Icon name or raw HTML SVG (if `$custom = true`) |
| `$custom` | `bool` | Whether `$icon` contains raw HTML |
| `$path` | `?string` | Directory path for custom Blade icon templates |

### Custom icons

#### From a Blade file

```php
->icon('my-icon', path: 'icons')
// Resolves to resources/views/icons/my-icon.blade.php
```

#### From Blade Icons package

```php
->icon(svg('heroicon-o-star')->toHtml(), custom: true)
```

#### Raw SVG HTML

```php
->icon('<svg xmlns="http://www.w3.org/2000/svg" ...>...</svg>', custom: true)
```

### Icon attribute

Set a default icon on a Resource or Page class:

```php
use MoonShine\Support\Attributes\Icon;

#[Icon('users')]
class UserResource extends ModelResource {}
```

This icon is used automatically by MenuItem when no explicit icon is set on the menu item.
