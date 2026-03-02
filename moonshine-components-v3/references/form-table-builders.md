# FormBuilder, TableBuilder, CardsBuilder

## FormBuilder

**Class:** `MoonShine\UI\Components\FormBuilder`
**Blade:** `<x-moonshine::form>`

### Constructor

```php
FormBuilder::make(
    string $action = '',
    FormMethod $method = FormMethod::POST,
    FieldsContract|iterable $fields = [],
    mixed $values = [],
)
```

### Core Methods

```php
// Set/change action URL
->action('/crud/update')

// Set HTTP method
->method(FormMethod::POST)  // POST, GET, PUT, DELETE, PATCH

// Add fields
->fields([Text::make('Title'), Textarea::make('Body')])

// Fill fields with values
->fill(['title' => 'Hello'])

// Type casting
->cast(new ModelCaster(User::class))
->fillCast($values, new ModelCaster(User::class))
->fillCast(User::query()->first(), new ModelCaster(User::class))

// Name (REQUIRED for events and async)
->name('my-form')
```

### Submit Button

```php
// Customize submit label and attributes
->submit(label: 'Save', attributes: ['class' => 'btn-primary'])

// Custom submit button
->submit(button: ActionButton::make('Confirm')->primary())

// Hide submit button
->hideSubmit()

// Add extra buttons
->buttons([
    ActionButton::make('Cancel', route('cancel')),
    ActionButton::make('Delete', route('delete'))->error(),
])
```

### Async Mode

```php
// Basic async (submits via AJAX)
->async()

// Async with events on success
->async(events: [
    AlpineJs::event(JsEvent::TABLE_UPDATED, 'my-table'),
    AlpineJs::event(JsEvent::FORM_RESET, 'my-form'),
])

// Async with custom URL
->async(url: '/custom-handler')

// Async with JS callback
->async(callback: AsyncCallback::with(responseHandler: 'myHandler'))
```

> **IMPORTANT:** `async()` must come AFTER `name()`.

### Calling Resource Methods

```php
// Call method directly on resource (no extra controller needed)
->asyncMethod('updateSomething')

// With file download
->asyncMethod('exportZip')->download()
```

Resource method signatures:

```php
// Notification response
public function updateSomething(MoonShineRequest $request): MoonShineJsonResponse
{
    return MoonShineJsonResponse::make()->toast('Done', ToastType::SUCCESS);
}

// Redirect response
public function updateSomething(MoonShineRequest $request): MoonShineJsonResponse
{
    return MoonShineJsonResponse::make()->redirect('/');
}
```

### Async Selectors (Replace HTML areas)

```php
// Replace field values by selector
public function formAction(): MoonShineJsonResponse
{
    return MoonShineJsonResponse::make()
        ->fieldsValues(['.title' => 'New Value']);
}

// Replace HTML blocks by selector
->asyncMethod('formAction')
->asyncSelector(['.area-1', '.area-2'])
```

### Reactivity

```php
// For forms outside resources, specify reactive URL manually
->reactiveUrl(
    fn(FormBuilder $form) => $form->getCore()->getRouter()->getEndpoints()->reactive($page, $resource)
)
```

### Validation

```php
// Hide errors above form
->errorsAbove(false)

// Pre-cognitive validation (Laravel Precognition)
->precognitive()

// Multiple forms on same page (need matching errorBag names)
FormBuilder::make(route('form.one'))->name('formOne')
FormBuilder::make(route('form.two'))->name('formTwo')
// In FormRequest: protected $errorBag = 'formOne';
```

### Apply (Process Fields in Controller)

```php
// Simple save
$form->apply(fn(Model $item) => $item->save());

// With before/after hooks
$form->apply(
    static fn(Model $item) => $item->save(),
    before: function (Model $item) {
        // pre-processing
        return $item;
    },
    after: function (Model $item) {
        // post-processing
        return $item;
    },
    throw: true
);
```

### Dispatch Events

```php
// Dispatch JS event on submit (instead of normal form submission)
->dispatchEvent(
    AlpineJs::event(JsEvent::OFF_CANVAS_TOGGLED, 'panel-name'),
    exclude: ['large_field'],  // exclude fields from payload
    withoutPayload: false       // true = send no data
)

// Trigger form submit via event
AlpineJs::event(JsEvent::FORM_SUBMIT, 'form-name')
```

### Other Methods

```php
->redirect('/after-save')       // Set redirect after save
->withoutRedirect()             // Prevent redirect
->rawMode()                     // Raw mode (no hidden fields)
->excludeFields(['_method'])    // Exclude fields from apply()
->customAttributes(['class' => 'custom-form'])
->onBeforeFieldsRender(fn($fields, $ctx) => $fields)
->switchFormMode(isAsync: true, events: '')  // Toggle async/precognitive
```

---

## TableBuilder

**Class:** `MoonShine\UI\Components\Table\TableBuilder`
**Blade:** `<x-moonshine::table>`

### Constructor

```php
TableBuilder::make(iterable $fields = [], iterable $items = [])
```

### Core Methods

```php
// Fields (each field = table column)
->fields([
    ID::make()->sortable(),
    Text::make('Title'),
    Text::make('Status')->customWrapperAttributes(['class' => 'w-20']),
])

// Data items
->items($collection)
->items([['id' => 1, 'title' => 'Hello']])

// Paginator
->paginator(
    (new ModelCaster(Article::class))->paginatorCast(Article::query()->paginate())
)
->simple()  // simplified pagination

// Name (REQUIRED for async/events)
->name('my-table')

// Key field name (when NOT using cast)
->castKeyName('id')

// Type casting
->cast(new ModelCaster(User::class))
```

### Action Buttons

```php
->buttons([
    ActionButton::make('Edit', fn() => route('edit')),
    ActionButton::make('More', '#')->showInDropdown(),
    ActionButton::make('View', fn() => route('show'))->blank()->canSee(fn($data) => $data->active),
    ActionButton::make('Mass Delete', route('mass.delete'))->bulk(),
])
->stickyButtons()  // stick buttons column
```

### View Methods

```php
// Vertical layout (for detail pages)
->vertical()
->vertical(title: 2, value: 10)  // customize column widths
->vertical(
    title: fn(FieldContract $field, Column $default, TableBuilder $ctx) => $default->columnSpan(3),
    value: fn(FieldContract $field, Column $default, TableBuilder $ctx) => $default->columnSpan(9),
)

// Editable mode (fields in form mode, not preview)
->editable()

// Preview mode (no buttons, no sorting)
->preview()

// "Not found" message when empty
->withNotFound()
```

### Row Customization

```php
// Custom tbody rows
->rows(static fn(TableRowsContract $default) =>
    $default->pushRow(
        TableCells::make()->pushCell('Custom cell content')
    )
)

// Custom thead rows
->headRows(static fn(TableRowContract $default) =>
    TableRows::make([$default])->pushRow(
        TableCells::make()->pushCell('Extra header')
    )
)

// Custom tfoot rows
->footRows(static fn(?TableRowContract $default) =>
    TableRows::make([$default])->pushRow(
        TableCells::make()->pushCell('Footer content')
    )
)
```

### Additional Features

```php
// Add new rows dynamically
->creatable(reindex: true, limit: 5, label: 'Add', icon: 'plus')
->creatable(button: ActionButton::make('Custom Add', '#'))

// Reindex field names (title[0], title[1], ...)
->reindex()

// Drag-and-drop sorting
->reorderable(url: '/reorder-url', key: 'id', group: 'group-name')

// Sticky header
->sticky()

// Column selection (user can show/hide columns)
->columnSelection()

// Client-side search
->searchable()

// Row click action
->clickAction(ClickAction::EDIT)  // EDIT, SELECT, DETAIL
->clickAction(ClickAction::EDIT, '.edit-button')

// Save table state in URL
->pushState()

// Modify bulk action checkbox
->modifyRowCheckbox(
    fn(Checkbox $cb, DataWrapperContract $data, TableBuilder $ctx) =>
        $data->getKey() === 2 ? $cb->customAttributes(['checked' => true]) : $cb
)
```

### Slots (Content Above Table)

```php
->topLeft(fn(): array => [Div::make([/* left content */])])
->topRight(fn(): array => [Div::make([/* right content */])])
```

### HTML Attributes

```php
->trAttributes(fn(?DataWrapperContract $data, int $row): array =>
    ['class' => $row % 2 ? 'bg-gray-100' : '']
)
->tdAttributes(fn(?DataWrapperContract $data, int $row, int $cell): array =>
    ['class' => $cell === 0 ? 'font-bold' : '']
)
->headAttributes(['class' => 'bg-blue-500'])
->bodyAttributes(['class' => 'text-sm'])
->footAttributes(['class' => 'bg-gray-200'])
->customAttributes(['class' => 'custom-table'])
```

### Async Loading

```php
// Basic async (auto-detects URL from current page)
->name('my-table')->async()

// With events on success
->name('my-table')->async(events: [
    AlpineJs::event(JsEvent::FORM_RESET, 'my-form'),
    AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Loaded', 'type' => 'success']),
])

// Custom URL for async (when outside MoonShine pages)
->name('my-table')->async(route('custom.component', [
    '_namespace' => self::class,
    '_component_name' => 'my-table'
]))
```

> **IMPORTANT:** `async()` must come AFTER `name()`.

### Lazy Loading

```php
// Load table data on page load via async request
TableBuilder::make()
    ->name('dashboard-table')
    ->fields([ID::make(), Text::make('Title')])
    ->async()
    ->lazy()
    ->whenAsync(fn(TableBuilder $t) => $t->items(
        Http::get('https://api.example.com/data')->json()
    ))

// Load on button click
ActionButton::make('Load')
    ->async(events: [AlpineJs::event(JsEvent::TABLE_UPDATED, 'my-table')]),

TableBuilder::make()
    ->name('my-table')
    ->fields([ID::make(), Text::make('Title')])
    ->async()->lazy()
    ->whenAsync(fn(TableBuilder $t) => $t->items($data))
    ->withNotFound()
```

### Events

| Event | Description |
|---|---|
| `JsEvent::TABLE_UPDATED` | Refresh entire table |
| `JsEvent::TABLE_REINDEX` | Reindex table fields |
| `JsEvent::TABLE_ROW_UPDATED` | Update single row: `"{name}-{row-id}"` |

### Blade Usage

```blade
<x-moonshine::table
    :columns="['#', 'Name', 'Email']"
    :values="[
        [1, 'John', 'john@example.com'],
        [2, 'Jane', 'jane@example.com'],
    ]"
    :sticky="true"
    :notfound="true"
/>
```

Styling classes for `tr`/`td`: `bgc-primary`, `bgc-secondary`, `bgc-success`, `bgc-warning`, `bgc-error`, `bgc-info`, `bgc-purple`, `bgc-pink`, `bgc-blue`, `bgc-green`, `bgc-yellow`, `bgc-red`, `bgc-gray`.

---

## CardsBuilder

**Class:** `MoonShine\UI\Components\CardsBuilder`

### Constructor

```php
CardsBuilder::make(iterable $items = [], FieldsContract|iterable $fields = [])
```

### Core Methods

```php
->items($collection)
->fields([ID::make(), Text::make('Title')])
->paginator((new ModelCaster(Article::class))->paginatorCast(Article::query()->paginate()))
->cast(new ModelCaster(Article::class))
->name('my-cards')
```

### View Methods

```php
// Card title (from column name or closure)
->title('title')
->title(fn($data) => $data->name)

// URL for title link
->url(fn($data) => route('show', $data->id))

// Subtitle
->subtitle(fn() => 'Subtitle text')

// Thumbnail image
->thumbnail('image_column')
->thumbnail(fn() => 'https://example.com/image.jpg')

// Overlay mode (title/subtitle over image)
->overlay()

// Header (only with thumbnail + overlay)
->header(static fn() => Badge::make('new', 'success'))

// Content
->content('Custom HTML content')

// Action buttons
->buttons([
    ActionButton::make('Edit', route('edit')),
    ActionButton::make('Delete', route('delete')),
])

// Card width in grid (12-column system)
->columnSpan(4, adaptiveColumnSpan: 12)  // 3 cards per row on desktop
```

### Custom Component

```php
// Replace entire card rendering
->customComponent(function (Article $article, int $index, CardsBuilder $builder) {
    return Badge::make($article->title, 'green');
})
```

### Async Mode

```php
->name('my-cards')->async()
->name('my-cards')->async(events: [AlpineJs::event(JsEvent::CARDS_UPDATED, 'my-cards')])
```

### Blade Usage

```blade
<x-moonshine::layout.grid>
    <x-moonshine::layout.column colSpan="4" adaptiveColSpan="12">
        <x-moonshine::card
            url="#"
            thumbnail="/images/image.jpg"
            :title="'Card Title'"
            :subtitle="'2024-01-01'"
            :values="['ID' => 1, 'Author' => 'John']"
        >
            <x-slot:header>
                <x-moonshine::badge color="green">new</x-moonshine::badge>
            </x-slot:header>
            Card content here
            <x-slot:actions>
                <x-moonshine::link-button href="#">Read more</x-moonshine::link-button>
            </x-slot:actions>
        </x-moonshine::card>
    </x-moonshine::layout.column>
</x-moonshine::layout.grid>
```
