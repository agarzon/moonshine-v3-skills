# Assets & Branding Reference

Complete API reference for MoonShine v3 asset management, Vite integration, and branding customization.

---

## Asset management

`MoonShine\AssetManager\AssetManager` manages CSS and JavaScript assets. It implements `AssetManagerContract`.

### Asset types

| Class | Output | Description |
|-------|--------|-------------|
| `Css` | `<link rel="stylesheet" href="...">` | External CSS file |
| `Js` | `<script src="..."></script>` | External JavaScript file |
| `InlineCss` | `<style>...</style>` | Inline CSS content |
| `InlineJs` | `<script>...</script>` | Inline JavaScript content |
| `Raw` | Raw HTML in `<head>` | Arbitrary HTML content |

### Css

```php
use MoonShine\AssetManager\Css;

Css::make('/css/styles.css');
Css::make('/css/styles.css')->defer();
Css::make('/css/styles.css')->version('1.0.0');
Css::make('/css/styles.css')->customAttributes(['media' => 'print']);
```

Methods:

| Method | Returns | Description |
|--------|---------|-------------|
| `make(string $link)` | `Css` | Create instance |
| `defer()` | `self` | Add defer attribute |
| `version(string\|int\|null $version)` | `static` | Set version |
| `customAttributes(array $attrs)` | `static` | Add HTML attributes |
| `link(string $link)` | `static` | Update the link path |
| `getLink()` | `string` | Get link with version query param |
| `toHtml()` | `string` | Render as `<link>` tag |

### Js

```php
use MoonShine\AssetManager\Js;

Js::make('/js/app.js');
Js::make('/js/app.js')->defer();
Js::make('/js/app.js')->version('2.0.0');
Js::make('/js/app.js')->customAttributes(['data-module' => 'main']);
```

Methods: same as Css, but renders as `<script src="...">`.

### InlineCss

```php
use MoonShine\AssetManager\InlineCss;

InlineCss::make(<<<'CSS'
    .custom-class {
        color: red;
    }
CSS);

InlineCss::make(':root { --radius: 0.15rem; }');
```

Methods:

| Method | Returns | Description |
|--------|---------|-------------|
| `make(string $content)` | `InlineCss` | Create instance |
| `getContent()` | `string` | Get CSS content |
| `customAttributes(array $attrs)` | `static` | Add attributes to `<style>` tag |
| `toHtml()` | `string` | Render as `<style>` tag |

### InlineJs

```php
use MoonShine\AssetManager\InlineJs;

InlineJs::make(<<<'JS'
    document.addEventListener("DOMContentLoaded", function() {
        console.log("Loaded");
    });
JS);
```

Methods: same as InlineCss, but renders as `<script>` tag. Also supports `version()`.

### Raw

```php
use MoonShine\AssetManager\Raw;

Raw::make('<link rel="preconnect" href="https://fonts.googleapis.com">');
Raw::make('<meta property="og:title" content="Admin Panel">');
```

### AssetManager API

Access via DI: `MoonShine\Contracts\AssetManager\AssetManagerContract`
Or via helper: `moonshine()->getAssetManager()`

| Method | Signature | Description |
|--------|-----------|-------------|
| `add()` | `add(AssetElementContract\|array $assets): static` | Add assets in natural order |
| `prepend()` | `prepend(AssetElementContract\|array $assets): static` | Add assets to the beginning |
| `append()` | `append(AssetElementContract\|array $assets): static` | Add assets to the end |
| `modifyAssets()` | `modifyAssets(Closure $callback): static` | Modify asset collection |
| `getAssets()` | `getAssets(): AssetElementsContract` | Get all resolved assets |
| `getAsset()` | `getAsset(string $path): string` | Resolve asset path |
| `toHtml()` | `toHtml(): string` | Render all assets as HTML |
| `flushState()` | `flushState(): void` | Clear all assets |

**Loading order:**
- `prepend()` assets come first
- `add()` assets in the middle (order depends on lifecycle timing)
- `append()` assets come last

### Adding assets at different levels

#### Global (ServiceProvider)

```php
use MoonShine\Contracts\AssetManager\AssetManagerContract;

public function boot(AssetManagerContract $assets): void
{
    $assets->add(Js::make('/js/app.js'));
}
```

#### Layout

```php
protected function assets(): array
{
    return [
        ...parent::assets(),
        Css::make('/css/custom.css'),
        Js::make('/js/custom.js'),
    ];
}
```

#### Resource

```php
protected function onLoad(): void
{
    $this->getAssetManager()
        ->prepend(InlineJs::make('alert(1)'))
        ->append(Js::make('/js/app.js'));
}
```

#### Page

```php
protected function onLoad(): void
{
    parent::onLoad();

    $this->getAssetManager()
        ->add(Css::make('/css/app.css'))
        ->append(Js::make('/js/app.js'));
}
```

#### Component (on the fly)

```php
Box::make()->addAssets([
    Js::make('/js/custom.js'),
    Css::make('/css/styles.css'),
]);
```

#### Component (class definition)

```php
final class MyComponent extends MoonShineComponent
{
    protected function assets(): array
    {
        return [
            Js::make('/js/custom.js'),
            Css::make('/css/styles.css'),
        ];
    }
}
```

#### Component (via AssetManager in booted)

```php
final class MyComponent extends MoonShineComponent
{
    protected function booted(): void
    {
        parent::booted();

        $this->getAssetManager()
            ->add(Css::make('/css/app.css'))
            ->append(Js::make('/js/app.js'));
    }
}
```

### Versioning

Assets support cache-busting via version query parameters:

```php
Js::make('/js/app.js')->version('1.0.0');
// Renders: /js/app.js?v=1.0.0
```

By default, the MoonShine framework version is used. Override per-asset with `version()`.

If the URL already has query parameters, the version is appended with `&`:

```
/js/app.js?module=true -> /js/app.js?module=true&v=1.0.0
```

### Asset modification

Filter or transform the asset collection:

```php
$assetManager->modifyAssets(function(array $assets) {
    return array_filter($assets, function($asset) {
        return !str_contains((string) $asset, 'remove-this');
    });
});
```

### Vite integration

Use Laravel Vite for asset compilation:

```php
use Illuminate\Support\Facades\Vite;
use MoonShine\AssetManager\Js;
use MoonShine\AssetManager\Css;

protected function assets(): array
{
    return [
        Js::make(Vite::asset('resources/js/app.js')),
        Css::make(Vite::asset('resources/css/app.css')),
    ];
}
```

Blade integration with Vite:

```blade
<x-moonshine::layout.assets>
    @vite([
        'resources/css/main.css',
        'resources/js/app.js',
    ], 'vendor/moonshine')
</x-moonshine::layout.assets>
```

---

## Branding

### Logo

#### Via layout method

```php
protected function getLogo(bool $small = false): string
{
    return $small
        ? '/images/logo-small.png'
        : '/images/logo.png';
}
```

#### Via configuration

```php
// config/moonshine.php
'logo' => '/images/logo.png',
'logo_small' => '/images/logo-small.png',
```

#### Via ServiceProvider

```php
$config
    ->logo('/images/logo.png')
    ->logo('/images/logo-small.png', small: true);
```

The Logo component supports `minimized()` mode for collapsed sidebars:

```php
Logo::make($this->getHomeUrl(), $this->getLogo(), $this->getLogo(small: true))
    ->minimized()
```

### Favicon

Override `getFaviconComponent()` to customize favicons:

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

The `bodyColor()` method on Favicon sets the theme color meta tag:

```php
Favicon::make()->bodyColor($this->getColorManager()->get('body'))
```

### Footer

#### Copyright

```php
protected function getFooterCopyright(): string
{
    return sprintf('&copy; %d My Company', now()->year);
}
```

#### Footer menu links

```php
protected function getFooterMenu(): array
{
    return [
        'https://docs.example.com' => 'Docs',
        'https://support.example.com' => 'Support',
    ];
}
```

#### Full footer component override

```php
protected function getFooterComponent(): Footer
{
    return Footer::make()
        ->copyright($this->getFooterCopyright())
        ->menu($this->getFooterMenu());
}
```
