---
name: moonshine-resources-v3
description: Use when creating ModelResource, configuring CRUD operations, working with tables/forms/detail pages, adding filters, search, pagination, events, buttons, query modification, import/export, or metrics in MoonShine v3.
---

# MoonShine v3 -- ModelResource

## Overview

`ModelResource` is the primary building block for admin panel sections backed by Eloquent models. It extends `CrudResource` and provides full CRUD functionality: index listing, create/edit forms, detail views, filtering, search, pagination, events, authorization, import/export, and metrics.

## Creating a Resource

```bash
php artisan moonshine:resource Post
```

This generates a resource class in `app/MoonShine/Resources/` and registers it in `MoonShineServiceProvider`.

## Core Properties

```php
namespace App\MoonShine\Resources;

use App\Models\Post;
use MoonShine\Laravel\Resources\ModelResource;

/**
 * @extends ModelResource<Post>
 */
class PostResource extends ModelResource
{
    protected string $model = Post::class;
    protected string $title = 'Posts';
    protected array $with = ['category'];          // eager load
    protected string $column = 'id';               // display column for breadcrumbs/relations
    protected ?string $alias = null;               // custom URL alias
}
```

## Declaring Fields per Page

Use `indexFields()`, `formFields()`, and `detailFields()` to define fields for each CRUD page.

> For field types, configuration, and relationship fields, see the `moonshine-fields-v3` skill.

```php
use MoonShine\UI\Components\Layout\Box;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

protected function indexFields(): iterable
{
    return [
        ID::make(),
        Text::make('Title'),
    ];
}

protected function formFields(): iterable
{
    return [
        Box::make([
            ID::make(),
            Text::make('Title'),
            Text::make('Subtitle'),
        ]),
    ];
}

protected function detailFields(): iterable
{
    return [
        Text::make('Title', 'title'),
        Text::make('Subtitle'),
    ];
}
```

### Pages-Based Resource (Alternative)

Instead of defining fields in the resource, you can generate separate page classes:

```php
use App\MoonShine\Pages\Post\PostIndexPage;
use App\MoonShine\Pages\Post\PostFormPage;
use App\MoonShine\Pages\Post\PostDetailPage;

protected function pages(): array
{
    return [
        PostIndexPage::class,
        PostFormPage::class,
        PostDetailPage::class,
    ];
}
```

Each page class extends `IndexPage`, `FormPage`, or `DetailPage` and has its own `fields()` method. See `references/crud-pages.md` for full customization details.

## Table Configuration

```php
use MoonShine\Support\Enums\SortDirection;
use MoonShine\Support\Enums\ClickAction;

class PostResource extends ModelResource
{
    protected string $sortColumn = 'created_at';
    protected SortDirection $sortDirection = SortDirection::DESC;

    // Pagination
    protected int $itemsPerPage = 25;
    protected bool $simplePaginate = false;    // use simple pagination
    protected bool $cursorPaginate = false;    // use cursor pagination
    protected bool $usePagination = true;      // set false to disable

    // Async mode (enabled by default)
    protected bool $isAsync = true;
    protected bool $isLazy = false;            // lazy-load table data

    // Display
    protected bool $stickyTable = false;       // sticky table header
    protected bool $stickyButtons = false;     // sticky action buttons column
    protected bool $columnSelection = false;   // let users toggle columns

    // Click action on table row
    protected ?ClickAction $clickAction = null;
    // Options: ClickAction::SELECT, ClickAction::DETAIL, ClickAction::EDIT
}
```

### Table Attributes

```php
use Closure;
use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;

protected function trAttributes(): Closure
{
    return fn(?DataWrapperContract $data, int $row) => [
        'data-tr' => $row,
    ];
}

protected function tdAttributes(): Closure
{
    return fn(?DataWrapperContract $data, int $row, int $cell) => [
        'width' => '20%',
    ];
}
```

### Custom thead/tbody/tfoot

```php
use MoonShine\UI\Collections\TableCells;
use MoonShine\UI\Collections\TableRows;
use MoonShine\UI\Components\Table\TableRow;

protected function tfoot(): null|TableRowsContract|Closure
{
    return static function(?TableRowContract $default, TableBuilder $table) {
        $cells = TableCells::make();
        $cells->pushCell('Balance:');
        $cells->pushCell('$1000');
        return TableRows::make([TableRow::make($cells), $default]);
    };
}
```

## Search

```php
protected function search(): array
{
    return ['id', 'title', 'text'];
}
```

### Full-Text Search

```php
use MoonShine\Support\Attributes\SearchUsingFullText;

#[SearchUsingFullText(['title', 'text'])]
protected function search(): array
{
    return ['id'];
}
```

### Relation Search

```php
protected function search(): array
{
    return ['category.title'];
}
```

See `references/filters-search.md` for filters, query tags, JSON search, global search, and query modification.

## Filters

Filters use the same field classes displayed on the index page:

```php
use MoonShine\UI\Fields\Text;

protected function filters(): iterable
{
    return [
        Text::make('Title', 'title'),
    ];
}
```

Cache filter state with:

```php
protected bool $saveQueryState = true;
```

## Modal CRUD

Open create, edit, or detail views in modal windows on the index page:

```php
class PostResource extends ModelResource
{
    protected bool $createInModal = true;
    protected bool $editInModal = true;
    protected bool $detailInModal = true;
}
```

## Active Actions

Control which CRUD operations are available globally:

```php
use MoonShine\Support\ListOf;
use MoonShine\Laravel\Enums\Action;

protected function activeActions(): ListOf
{
    return parent::activeActions()
        ->except(Action::VIEW, Action::MASS_DELETE);
    // or: ->only(Action::VIEW)
}
```

Available actions: `Action::CREATE`, `Action::VIEW`, `Action::UPDATE`, `Action::DELETE`, `Action::MASS_DELETE`.

## Redirects

```php
use MoonShine\Support\Enums\PageType;

// Via property (redirect to form after save)
protected ?PageType $redirectAfterSave = PageType::FORM;

// Via method (custom URL)
public function getRedirectAfterSave(): string
{
    return '/';
}

public function getRedirectAfterDelete(): string
{
    return $this->getIndexPageUrl();
}
```

## Validation

```php
protected function rules(mixed $item): array
{
    return [
        'title' => ['required', 'string', 'min:5'],
    ];
}

public function validationMessages(): array
{
    return [
        'email.required' => 'Email is required',
    ];
}

public function prepareForValidation(): void
{
    request()?->merge([
        'email' => request()?->string('email')->lower()->value(),
    ]);
}

// Enable precognitive validation
protected bool $isPrecognitive = true;
```

## Events Lifecycle

Override these methods in your resource to hook into CRUD operations:

```php
protected function beforeCreating(mixed $item): mixed  { return $item; }
protected function afterCreated(mixed $item): mixed     { return $item; }
protected function beforeUpdating(mixed $item): mixed   { return $item; }
protected function afterUpdated(mixed $item): mixed     { return $item; }
protected function beforeDeleting(mixed $item): mixed   { return $item; }
protected function afterDeleted(mixed $item): mixed     { return $item; }
protected function beforeMassDeleting(array $ids): void {}
protected function afterMassDeleted(array $ids): void   {}
```

See `references/events-buttons.md` for event examples, button customization, import/export, and metrics.

## Buttons Overview

```php
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;

// Add buttons above the index table (next to Create)
protected function topButtons(): ListOf
{
    return parent::topButtons()->add(
        ActionButton::make('Refresh', '#')
            ->dispatchEvent(AlpineJs::event(JsEvent::TABLE_UPDATED, $this->getListComponentName()))
    );
}

// Per-row buttons in the index table
protected function indexButtons(): ListOf
{
    return parent::indexButtons()->add(
        ActionButton::make('Link', '/endpoint')
    );
}

// Bulk action button
protected function indexButtons(): ListOf
{
    return parent::indexButtons()->add(
        ActionButton::make('Bulk Action', '/endpoint')->bulk()
    );
}
```

## Query Modification

```php
use Illuminate\Contracts\Database\Eloquent\Builder;

// Modify all queries for this resource
protected function modifyQueryBuilder(Builder $builder): Builder
{
    return $builder->where('active', true);
}

// Modify query for single-record retrieval
protected function modifyItemQueryBuilder(Builder $builder): Builder
{
    return $builder->withTrashed();
}
```

## Component Modifiers

Override the main component on any CRUD page from the resource:

```php
use MoonShine\Contracts\UI\ComponentContract;

public function modifyListComponent(ComponentContract $component): ComponentContract
{
    return parent::modifyListComponent($component)->customAttributes(['data-my-attr' => 'value']);
}

public function modifyFormComponent(ComponentContract $component): ComponentContract
{
    return parent::modifyFormComponent($component)->customAttributes(['data-my-attr' => 'value']);
}

public function modifyDetailComponent(ComponentContract $component): ComponentContract
{
    return parent::modifyDetailComponent($component)->customAttributes(['data-my-attr' => 'value']);
}
```

## Page Components (Quick Add)

Add extra components to pages without publishing page classes:

```php
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Components\Modal;
use MoonShine\UI\Fields\Text;

// Added to bottomLayer of all pages
protected function pageComponents(): array
{
    return [
        Modal::make('My Modal', components: [
            FormBuilder::make()->fields([
                Text::make('Title'),
            ])
        ])->name('demo-modal'),
    ];
}

// Or target specific pages:
// indexPageComponents(), formPageComponents(), detailPageComponents()
```

## Resource Lifecycle Hooks

```php
// Fires when resource is loaded and active
protected function onLoad(): void
{
    // e.g., add assets, push components to layers
}

// Fires when MoonShine creates the resource instance
protected function onBoot(): void
{
    // e.g., early initialization
}
```

Traits can hook in via `load{TraitName}` / `boot{TraitName}` naming convention.

## Authorization

```php
protected bool $withPolicy = true; // enable Laravel Policy checks
```

Available Policy methods: `viewAny`, `view`, `create`, `update`, `delete`, `massDelete`, `restore`, `forceDelete`.

Generate a policy:
```bash
php artisan moonshine:policy PostPolicy
```

## Routes Helper

```php
$resource->getUrl();                    // first page
$resource->getIndexPageUrl();           // index page
$resource->getFormPageUrl();            // create page
$resource->getFormPageUrl(1);           // edit page by ID
$resource->getDetailPageUrl($item);     // detail page
$resource->getAsyncMethodUrl('method'); // custom async method
$resource->getRoute('crud.update', $id); // CRUD routes
```

## Response Modifiers (Async Mode)

```php
use MoonShine\Laravel\Http\Responses\MoonShineJsonResponse;

public function modifySaveResponse(MoonShineJsonResponse $response): MoonShineJsonResponse
{
    return $response;
}

public function modifyDestroyResponse(MoonShineJsonResponse $response): MoonShineJsonResponse
{
    return $response;
}

public function modifyMassDeleteResponse(MoonShineJsonResponse $response): MoonShineJsonResponse
{
    return $response;
}
```

## Cross-References

- For field types and relationship fields, see the `moonshine-fields-v3` skill.
- For ActionButton advanced usage, see the `moonshine-action-button` skill.
- For Layout and menu configuration, see the `moonshine-appearance-v3` skill.
- For TableBuilder and FormBuilder components, see the `moonshine-components-v3` skill.
- Detailed CRUD page customization: `references/crud-pages.md`
- Filters, search, and query details: `references/filters-search.md`
- Events, buttons, import/export, metrics: `references/events-buttons.md`
