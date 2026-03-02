# TypeCasts, Apply Classes, Packages & Localization Reference

MoonShine v3 type casting, apply classes, artisan commands, package development, and localization.

## Type Casts

Fields work with primitives by default. TypeCasts provide typed data access for models, DTOs, or any data structure.

### Interfaces

```php
interface DataCasterContract
{
    public function cast(mixed $data): DataWrapperContract;
    public function paginatorCast(mixed $data): ?PaginatorContract;
}

interface DataWrapperContract
{
    public function getOriginal(): mixed;
    public function getKey(): int|string|null;
    public function toArray(): array;
}
```

### Built-in ModelCaster

```php
use MoonShine\Laravel\TypeCasts\ModelCaster;

// With TableBuilder
TableBuilder::make(items: User::paginate())
    ->fields([Text::make('Email')])
    ->cast(new ModelCaster(User::class));

// With FormBuilder
FormBuilder::make()
    ->fields([Text::make('Email')])
    ->fillCast(User::query()->first(), new ModelCaster(User::class));
```

### Custom TypeCast Example

```php
final readonly class ModelCaster implements DataCasterContract
{
    public function __construct(private string $class) {}

    public function getClass(): string { return $this->class; }

    public function cast(mixed $data): ModelDataWrapper
    {
        $model = new ($this->getClass());
        return new ModelDataWrapper($model->fill($data));
    }

    public function paginatorCast(mixed $data): ?PaginatorContract
    {
        if (! $data instanceof Paginator && ! $data instanceof CursorPaginator) {
            return null;
        }
        return (new PaginatorCaster(
            $data->appends(moonshine()->getRequest()->getExcept('page'))->toArray(),
            $data->items()
        ))->cast();
    }
}

final readonly class ModelDataWrapper implements DataWrapperContract
{
    public function __construct(private Model $model) {}

    public function getOriginal(): Model { return $this->model; }
    public function getKey(): int|string|null { return $this->model->getKey(); }
    public function toArray(): array { return $this->model->toArray(); }
}
```

### Generating a TypeCast

```shell
php artisan moonshine:type-cast {className?} {--base-dir=} {--base-namespace=}
```

Creates a file in `app/MoonShine/TypeCasts/`.

---

## Apply Classes

Apply classes let you override how a field's value is applied (saved) to the model.

### Generate

```shell
php artisan moonshine:apply {className?} {--base-dir=} {--base-namespace=}
```

Creates a class in `app/MoonShine/Applies/`. The class must be registered in a service provider.

---

## Artisan Commands Reference

| Command | Purpose |
|---------|---------|
| `moonshine:install` | Install MoonShine package |
| `moonshine:user` | Create superuser |
| `moonshine:resource` | Create resource (with `--test`/`--pest`/`--policy` flags) |
| `moonshine:page` | Create page (`--crud` for index/form/detail set) |
| `moonshine:layout` | Create layout (`--compact`/`--full`/`--default`) |
| `moonshine:component` | Create custom component |
| `moonshine:field` | Create custom field |
| `moonshine:controller` | Create controller |
| `moonshine:handler` | Create handler |
| `moonshine:policy` | Create policy for admin panel user |
| `moonshine:type-cast` | Create TypeCast class |
| `moonshine:apply` | Create apply class |
| `moonshine:publish` | Publish assets, resources, forms, or pages |

Common options for most commands: `--base-dir=`, `--base-namespace=` to customize output location.

---

## Package Development

### ServiceProvider Integration

```php
use MoonShine\Contracts\Core\DependencyInjection\CoreContract;
use MoonShine\Laravel\DependencyInjection\MoonShine;

class MyPackageServiceProvider extends ServiceProvider
{
    /** @param MoonShine $core */
    public function boot(CoreContract $core): void
    {
        $core
            ->resources([MyPackageResource::class])
            ->pages([MyPackagePage::class]);
    }
}
```

Add menu items:
```php
public function boot(CoreContract $core, MenuManagerContract $menu): void
{
    $menu->add([MenuItem::make('MyPage', MyPackagePage::class)]);
}
```

Add assets or colors:
```php
public function boot(CoreContract $core, AssetManagerContract $assets): void
{
    $assets->add([InlineCss::make('body {background: red;}')]);
}
```

Add authorization rules:
```php
/** @param MoonShineConfigurator $configurator */
public function boot(ConfiguratorContract $configurator): void
{
    $configurator->authorizationRules(
        static fn(ResourceContract $resource, Model $user, Ability $ability): bool => true
    );
}
```

Push components to existing pages:
```php
public function boot(): void
{
    ProfilePage::pushComponent(fn() => MyPackageComponent::make());
}
```

### Package Traits

Use `load{TraitName}()`/`boot{TraitName}()` magic methods in traits for resources/pages:

```php
trait HasMyPackageTrait
{
    public function loadHasMyPackageTrait(): void
    {
        $this->getFormPage()->addAssets([
            Js::make('vendor/my-package/js/app.js'),
        ]);
    }
}
```

### Auto-Discovery in composer.json

```json
"extra": {
    "laravel": {
        "providers": ["Author\\MoonShineMyPackage\\MyPackageServiceProvider"]
    }
}
```

---

## Localization

### Configuration

```php
// config/moonshine.php
'locale' => 'en',
'locales' => ['en', 'ru'],

// Or via MoonShineServiceProvider
$config->locale('en');
$config->locales(['en' => 'English', 'ru' => 'Russian']);
```

### Language Switching

Handled by `MoonShine\Laravel\Http\Middleware\ChangeLocale`. Replace with your own middleware to customize switching logic.

Translation files are in `lang/vendor/moonshine/`. Additional languages available via [laravel-lang/moonshine](https://laravel-lang.com/packages-moonshine.html).
