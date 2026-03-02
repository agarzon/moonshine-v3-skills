# CRUD Pages Reference

## Page Types

MoonShine v3 uses three CRUD page types, identified by `PageType` enum:

```php
use MoonShine\Support\Enums\PageType;

PageType::INDEX;   // Index listing page
PageType::FORM;    // Create/edit form page
PageType::DETAIL;  // Detail view page
```

## Creating a Pages-Based Resource

When generating a resource, choose "Model resource with pages":

```bash
php artisan moonshine:resource Post
```

This creates page classes in `app/MoonShine/Pages/Post/`:

```php
use App\MoonShine\Pages\Post\PostIndexPage;
use App\MoonShine\Pages\Post\PostFormPage;
use App\MoonShine\Pages\Post\PostDetailPage;

class PostResource extends ModelResource
{
    protected function pages(): array
    {
        return [
            PostIndexPage::class,
            PostFormPage::class,
            PostDetailPage::class,
        ];
    }
}
```

## Page Layers

All CRUD pages are divided into three layers:

- **TopLayer** -- metrics on index page, action buttons on form page
- **MainLayer** -- primary content (TableBuilder on index, FormBuilder on form, detail table on detail)
- **BottomLayer** -- extra content, relation fields, modals

Override layer methods to customize:

```php
use MoonShine\Laravel\Pages\Crud\IndexPage;
use MoonShine\UI\Components\Heading;

class PostIndexPage extends IndexPage
{
    protected function topLayer(): array
    {
        return [
            Heading::make('Custom top'),
            ...parent::topLayer()
        ];
    }

    protected function mainLayer(): array
    {
        return [
            Heading::make('Custom main'),
            ...parent::mainLayer()
        ];
    }

    protected function bottomLayer(): array
    {
        return [
            Heading::make('Custom bottom'),
            ...parent::bottomLayer()
        ];
    }
}
```

Access layer components from resource:

```php
use MoonShine\Support\Enums\Layer;

// From a resource
$this->getFormPage()->getLayerComponents(Layer::BOTTOM);

// From a page
$this->getLayerComponents(Layer::BOTTOM);
```

Push a component to a page layer from the resource's `onLoad()`:

```php
use MoonShine\Support\Enums\Layer;

protected function onLoad(): void
{
    $this->getFormPage()->pushToLayer(
        layer: Layer::BOTTOM,
        component: Permissions::make('Permissions', $this)
    );
}
```

---

## IndexPage Customization

### Defining Fields

```php
use MoonShine\Laravel\Pages\Crud\IndexPage;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

class PostIndexPage extends IndexPage
{
    protected function fields(): iterable
    {
        return [
            ID::make(),
            Text::make('Title'),
        ];
    }
}
```

### Replacing the Main Component (getItemsComponent)

Replace the TableBuilder with a custom component:

```php
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Contracts\UI\TableBuilderContract;
use MoonShine\Laravel\Collections\Fields;
use MoonShine\UI\Components\Table\TableBuilder;

class PostIndexPage extends IndexPage
{
    protected function getItemsComponent(iterable $items, Fields $fields): ComponentContract
    {
        return TableBuilder::make(items: $items)
            ->name($this->getListComponentName())
            ->fields($fields)
            ->cast($this->getResource()->getCaster())
            ->withNotFound()
            ->when(
                ! is_null($head = $this->getResource()->getHeadRows()),
                fn(TableBuilderContract $table) => $table->headRows($head)
            )
            ->when(
                ! is_null($body = $this->getResource()->getRows()),
                fn(TableBuilderContract $table) => $table->rows($body)
            )
            ->when(
                ! is_null($foot = $this->getResource()->getFootRows()),
                fn(TableBuilderContract $table) => $table->footRows($foot)
            )
            ->when(
                ! is_null($this->getResource()->getTrAttributes()),
                fn(TableBuilderContract $table) => $table->trAttributes(
                    $this->getResource()->getTrAttributes()
                )
            )
            ->when(
                ! is_null($this->getResource()->getTdAttributes()),
                fn(TableBuilderContract $table) => $table->tdAttributes(
                    $this->getResource()->getTdAttributes()
                )
            )
            ->buttons($this->getResource()->getIndexButtons())
            ->clickAction($this->getResource()->getClickAction())
            ->when($this->getResource()->isAsync(), function(TableBuilderContract $table): void {
                $table->async()->pushState();
            })
            ->when($this->getResource()->isStickyTable(), function(TableBuilderContract $table): void {
                $table->sticky();
            })
            ->when($this->getResource()->isColumnSelection(), function(TableBuilderContract $table): void {
                $table->columnSelection();
            });
    }
}
```

### Using List Component Outside Resource

```php
app(MoonShineUserResource::class)->getIndexPage()->getListComponent(withoutFragment: false);
```

---

## FormPage Customization

### Replacing the Form Component (getFormComponent)

```php
use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Contracts\UI\FormBuilderContract;
use MoonShine\Laravel\Collections\Fields;
use MoonShine\Laravel\Pages\Crud\FormPage;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Fields\Hidden;

class PostFormPage extends FormPage
{
    protected function getFormComponent(
        string $action,
        ?DataWrapperContract $item,
        Fields $fields,
        bool $isAsync = true,
    ): ComponentContract {
        $resource = $this->getResource();

        return FormBuilder::make($action)
            ->cast($this->getResource()->getCaster())
            ->fill($item)
            ->fields([
                ...$fields
                    ->when(
                        ! is_null($item),
                        static fn(Fields $fields) => $fields->push(
                            Hidden::make('_method')->setValue('PUT')
                        )
                    )
                    ->when(
                        ! $resource->isItemExists() && ! $resource->isCreateInModal(),
                        static fn(Fields $fields) => $fields->push(
                            Hidden::make('_force_redirect')->setValue(true)
                        )
                    )
                    ->toArray(),
            ])
            ->when(
                ! $resource->hasErrorsAbove(),
                fn(FormBuilderContract $form) => $form->errorsAbove($resource->hasErrorsAbove())
            )
            ->when(
                $isAsync,
                static fn(FormBuilderContract $formBuilder) => $formBuilder
                    ->async(events: array_filter([
                        $resource->getListEventName(
                            request()->input('_component_name', 'default'),
                            $isAsync && $resource->isItemExists() ? array_filter([
                                'page' => request()->input('page'),
                                'sort' => request()->input('sort'),
                            ]) : []
                        ),
                        ! $resource->isItemExists() && $resource->isCreateInModal()
                            ? AlpineJs::event(JsEvent::FORM_RESET, $resource->getUriKey())
                            : null,
                    ]))
            )
            ->when(
                $resource->isPrecognitive(),
                static fn(FormBuilderContract $form) => $form->precognitive()
            )
            ->when(
                $resource->isSubmitShowWhen(),
                static fn(FormBuilderContract $form) => $form->submitShowWhenAttribute()
            )
            ->name($resource->getUriKey())
            ->submit(__('moonshine::ui.save'), ['class' => 'btn-primary btn-lg'])
            ->buttons($resource->getFormBuilderButtons());
    }
}
```

---

## DetailPage Customization

### Replacing the Detail Component (getDetailComponent)

```php
use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Laravel\Collections\Fields;
use MoonShine\UI\Components\Table\TableBuilder;

class PostDetailPage extends DetailPage
{
    protected function getDetailComponent(?DataWrapperContract $item, Fields $fields): ComponentContract
    {
        return TableBuilder::make($fields)
            ->cast($this->getResource()->getCaster())
            ->items([$item])
            ->vertical()
            ->simple()
            ->preview();
    }
}
```

---

## Modifiers from the Resource

Instead of publishing page classes, you can modify the main component from the resource:

```php
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\UI\Components\FlexibleRender;

// Modify the index table
public function modifyListComponent(ComponentContract $component): ComponentContract
{
    return parent::modifyListComponent($component)->customAttributes([
        'data-my-attr' => 'value'
    ]);
}

// Modify the form
public function modifyFormComponent(ComponentContract $component): ComponentContract
{
    return parent::modifyFormComponent($component)->fields([
        FlexibleRender::make('Top'),
        ...parent::modifyFormComponent($component)->getFields()->toArray(),
        FlexibleRender::make('Bottom'),
    ])->submit('Go');
}

// Modify the detail view
public function modifyDetailComponent(ComponentContract $component): ComponentContract
{
    return parent::modifyDetailComponent($component)->customAttributes([
        'data-my-attr' => 'value'
    ]);
}
```

---

## Redirects and Response Modifiers

### Redirect After Save

```php
use MoonShine\Support\Enums\PageType;

// Via property
protected ?PageType $redirectAfterSave = PageType::FORM;

// Via method
public function getRedirectAfterSave(): string
{
    return '/';
}

public function getRedirectAfterDelete(): string
{
    return $this->getIndexPageUrl();
}
```

### Response Modifiers (Async Mode)

When the resource operates in async mode, you can modify JSON responses:

```php
use Symfony\Component\HttpFoundation\Response;
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

public function modifyErrorResponse(Response $response, Throwable $exception): Response
{
    return $response;
}
```

---

## Simulate Route

Use CRUD pages on non-standard routes by emulating the route context:

```php
class HomeController extends Controller
{
    public function __invoke(FormArticlePage $page, ArticleResource $resource)
    {
        return $page->simulateRoute($page, $resource);
    }
}
```

---

## Resource Routes Reference

```php
$resource->getUrl();                                  // first page
$resource->getIndexPageUrl();                         // index page
$resource->getIndexPageUrl(['query-tag' => $tag->uri()]);  // with query tag
$resource->getFormPageUrl();                          // create form
$resource->getFormPageUrl(1);                         // edit form by ID
$resource->getFormPageUrl($item);                     // edit form by Model
$resource->getDetailPageUrl(1);                       // detail by ID
$resource->getDetailPageUrl($item);                   // detail by Model
$resource->getAsyncMethodUrl('updateSomething');      // async method
$resource->getFragmentLoadUrl('table-index', $page); // fragment load

// CRUD routes
$resource->getRoute('crud.update', $data->getKey()); // PUT
$resource->getRoute('crud.store');                    // POST
$resource->getRoute('crud.destroy', $data->getKey()); // DELETE
$resource->getRoute('crud.massDelete');               // DELETE (mass)

// Handlers
$resource->getRoute('handler', query: ['handlerUri' => $export->getUriKey()]);
```

### Active Page

```php
$resource->getActivePage();      // ?PageContract
$resource->isIndexPage();        // bool
$resource->isFormPage();         // bool
$resource->isDetailPage();       // bool
$resource->isCreateFormPage();   // bool
$resource->isUpdateFormPage();   // bool
```
