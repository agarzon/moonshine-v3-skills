# Events, Buttons, Import/Export, and Metrics Reference

## Resource Events

MoonShine provides lifecycle hooks that run within the resource's CRUD operations. Override these methods in your resource class.

### All Available Events

```php
class PostResource extends ModelResource
{
    // -- Create lifecycle --

    protected function beforeCreating(mixed $item): mixed
    {
        // Runs before a new record is saved
        if (auth()->user()->moonshine_user_role_id !== 1) {
            request()->merge([
                'author_id' => auth()->id(),
            ]);
        }
        return $item;
    }

    protected function afterCreated(mixed $item): mixed
    {
        // Runs after a new record is saved
        return $item;
    }

    // -- Update lifecycle --

    protected function beforeUpdating(mixed $item): mixed
    {
        // Runs before an existing record is updated
        if (auth()->user()->moonshine_user_role_id !== 1) {
            request()->merge([
                'author_id' => auth()->id(),
            ]);
        }
        return $item;
    }

    protected function afterUpdated(mixed $item): mixed
    {
        // Runs after an existing record is updated
        return $item;
    }

    // -- Delete lifecycle --

    protected function beforeDeleting(mixed $item): mixed
    {
        // Runs before a record is deleted
        return $item;
    }

    protected function afterDeleted(mixed $item): mixed
    {
        // Runs after a record is deleted
        return $item;
    }

    // -- Mass delete lifecycle --

    protected function beforeMassDeleting(array $ids): void
    {
        // Runs before mass deletion with the array of IDs
    }

    protected function afterMassDeleted(array $ids): void
    {
        // Runs after mass deletion
    }
}
```

Standard Laravel Eloquent events (creating, created, updating, updated, deleting, deleted) also work since MoonShine uses standard Eloquent save/delete methods under the hood.

---

## Button Customization

Buttons are `ActionButton` components displayed on resource pages. MoonShine provides methods for every button location.

### Modifying Standard CRUD Buttons

Each standard button can be modified or replaced via `modify*Button()` methods:

```php
use MoonShine\Contracts\UI\ActionButtonContract;
use MoonShine\UI\Components\ActionButton;

// Create button (top of index page)
protected function modifyCreateButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->error();
    // Or replace entirely:
    // return ActionButton::make('Create');
}

// Detail button (per row)
protected function modifyDetailButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->warning();
}

// Edit button (per row)
protected function modifyEditButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->icon('pencil-square');
}

// Delete button (per row)
protected function modifyDeleteButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->icon('x-mark');
}

// Mass delete button
protected function modifyMassDeleteButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->icon('x-mark');
}

// Filters button
protected function modifyFiltersButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->error();
}
```

Standard button names for reference:
- `resource-detail-button`
- `resource-edit-button`
- `resource-delete-button`
- `mass-delete-button`

### Index Page Top Buttons

Buttons displayed above the table, next to the Create button:

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;

protected function topButtons(): ListOf
{
    return parent::topButtons()->add(
        ActionButton::make('Refresh', '#')
            ->dispatchEvent(
                AlpineJs::event(JsEvent::TABLE_UPDATED, $this->getListComponentName())
            )
    );
}
```

### Index Table Row Buttons

Per-row action buttons in the table:

```php
use Illuminate\Database\Eloquent\Model;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;

protected function indexButtons(): ListOf
{
    return parent::indexButtons()->prepend(
        ActionButton::make(
            'Link',
            fn(Model $item) => '/endpoint?id=' . $item->getKey()
        )
    );
}
```

Remove a specific button:

```php
protected function indexButtons(): ListOf
{
    return parent::indexButtons()
        ->except(fn(ActionButton $btn) => $btn->getName() === 'resource-delete-button');
}
```

Clear all and add custom:

```php
protected function indexButtons(): ListOf
{
    return parent::indexButtons()
        ->empty()
        ->add(ActionButton::make('Link', '/endpoint'));
}
```

### Bulk Action Buttons

Add `bulk()` to make a button operate on selected rows:

```php
protected function indexButtons(): ListOf
{
    return parent::indexButtons()->add(
        ActionButton::make('Bulk Export', '/endpoint')->bulk()
    );
}
```

### Button Display Mode

Display buttons in a dropdown menu instead of inline:

```php
class PostResource extends ModelResource
{
    protected bool $indexButtonsInDropdown = true;
}
```

Or control per-button:

```php
protected function indexButtons(): ListOf
{
    return parent::indexButtons()->prepend(
        ActionButton::make('Button 1', '/')->showInLine(),
        ActionButton::make('Button 2', '/')->showInDropdown(),
    );
}
```

### Form Page Buttons

Buttons displayed on the form page (above the form):

```php
protected function formButtons(): ListOf
{
    return parent::formButtons()->add(
        ActionButton::make('Link')->method('updateSomething')
    );
}
```

### Form Builder Buttons

Buttons inside the form (next to Save):

```php
protected function formBuilderButtons(): ListOf
{
    return parent::formBuilderButtons()->add(
        ActionButton::make('Back', fn() => $this->getIndexPageUrl())->class('btn-lg')
    );
}
```

### Detail Page Buttons

```php
protected function detailButtons(): ListOf
{
    return parent::detailButtons()->add(
        ActionButton::make('Link', '/endpoint')
    );
}
```

---

## Async Row Update

Trigger an asynchronous update of a single table row:

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Fields\Switcher;

protected function indexFields(): iterable
{
    return [
        ID::make(),
        Text::make('Title'),
        Switcher::make('Active')
            ->updateOnPreview(
                events: [AlpineJs::event(JsEvent::TABLE_ROW_UPDATED, $this->getListComponentNameWithRow())]
            ),
    ];
}
```

Simplified with `withUpdateRow()`:

```php
Switcher::make('Active')
    ->withUpdateRow($this->getListComponentName())
```

Triggering from a response:

```php
use MoonShine\Laravel\Http\Responses\MoonShineJsonResponse;

return MoonShineJsonResponse::make()
    ->events([
        AlpineJs::event(JsEvent::TABLE_ROW_UPDATED, $this->getListComponentNameWithRow($item->getKey()))
    ])
    ->toast('Success');
```

---

## Import / Export

Requires the `moonshine/import-export` package:

```bash
composer require moonshine/import-export
```

### Setup

Add the trait and interface to your resource:

```php
use MoonShine\ImportExport\Contracts\HasImportExportContract;
use MoonShine\ImportExport\Traits\ImportExportConcern;

class CategoryResource extends ModelResource implements HasImportExportContract
{
    use ImportExportConcern;
}
```

### Import Fields

```php
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

protected function importFields(): iterable
{
    return [
        ID::make(),       // Required for upsert behavior
        Text::make('Name'),
    ];
}
```

Transform values during import with `fromRaw()`:

```php
use App\Enums\StatusEnum;
use MoonShine\UI\Fields\Enum;

protected function importFields(): iterable
{
    return [
        ID::make(),
        Enum::make('Status')
            ->attach(StatusEnum::class)
            ->fromRaw(static fn(string $raw, Enum $ctx) => StatusEnum::tryFrom($raw)),
    ];
}
```

### Import Settings

```php
use MoonShine\ImportExport\ImportHandler;
use MoonShine\Laravel\Handlers\Handler;
use MoonShine\UI\Components\ActionButton;

protected function import(): ?Handler
{
    return ImportHandler::make(__('moonshine::ui.import'))
        ->notifyUsers(fn(ImportHandler $ctx) => [auth()->id()])
        ->disk('public')
        ->dir('/imports')
        ->deleteAfter()
        ->delimiter(',')
        ->queue()                                              // run in background
        ->modifyButton(fn(ActionButton $btn) => $btn->class('my-class'));
}
```

Return `null` from `import()` to hide the import button.

### Import Events

```php
public function beforeImportFilling(array $data): array
{
    return $data;
}

public function beforeImported(mixed $item): mixed
{
    return $item;
}

public function afterImported(mixed $item): mixed
{
    return $item;
}
```

### Export Fields

```php
protected function exportFields(): iterable
{
    return [
        ID::make(),
        Text::make('Name'),
    ];
}
```

Transform values during export with `modifyRawValue()`:

```php
use App\Enums\StatusEnum;

protected function exportFields(): iterable
{
    return [
        ID::make(),
        Enum::make('Status')
            ->attach(StatusEnum::class)
            ->modifyRawValue(static fn(StatusEnum $raw, $data, $ctx) => $raw->value),
    ];
}
```

### Export Settings

```php
use MoonShine\ImportExport\ExportHandler;
use MoonShine\Laravel\Handlers\Handler;
use MoonShine\UI\Components\ActionButton;

protected function export(): ?Handler
{
    return ExportHandler::make(__('moonshine::ui.export'))
        ->notifyUsers(fn() => [auth()->id()])
        ->disk('public')
        ->filename(sprintf('export_%s', date('Ymd-His')))
        ->dir('/exports')
        ->csv()                                                 // export as CSV instead of xlsx
        ->delimiter(',')
        ->withConfirm()                                         // ask for confirmation
        ->queue()                                               // run in background
        ->modifyButton(fn(ActionButton $btn) => $btn->class('my-class'));
}
```

Return `null` from `export()` to hide the export button.

### Custom Import/Export Handler

Generate a handler class:

```bash
php artisan moonshine:handler
```

Change the parent from `Handler` to `ImportHandler` or `ExportHandler` and implement your custom logic.

---

## Metrics

Display statistical blocks at the top of the index page:

```php
use App\Models\Post;
use App\Models\Comment;
use MoonShine\UI\Components\Metrics\Wrapped\ValueMetric;

protected function metrics(): array
{
    return [
        ValueMetric::make('Articles')
            ->value(fn() => Post::count())
            ->columnSpan(6),
        ValueMetric::make('Comments')
            ->value(fn() => Comment::count())
            ->columnSpan(6),
    ];
}
```

### Wrap Metrics in a Fragment

To allow async refresh of metrics:

```php
use Closure;
use MoonShine\Laravel\Components\Fragment;

protected function fragmentMetrics(): ?Closure
{
    return static fn(array $components): Fragment => Fragment::make($components)->name('metrics');
}
```

> For advanced metric types (DonutChartMetric, LineChartMetric), see the `moonshine-components` skill.
