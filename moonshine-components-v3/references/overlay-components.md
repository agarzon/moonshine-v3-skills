# Overlay Components

## Modal

**Class:** `MoonShine\UI\Components\Modal`
**Blade:** `<x-moonshine::modal>`

### Constructor

```php
Modal::make(
    Closure|string $title = '',
    Closure|Renderable|string $content = '',
    Closure|Renderable|ActionButtonContract|string $outer = '',
    Closure|string|null $asyncUrl = null,
    iterable $components = [],
)
```

- `$title` -- modal title
- `$content` -- modal body content
- `$outer` -- external trigger block (e.g., a button)
- `$asyncUrl` -- URL for async content loading
- `$components` -- nested components

### Basic Usage

```php
Modal::make(
    title: 'Confirm Action',
    content: 'Are you sure you want to proceed?',
)
```

### Naming and Events

A unique `name()` is required for event-based control.

```php
Modal::make('Title', 'Content')
    ->name('my-modal'),

// Open via ActionButton
ActionButton::make('Open')->toggleModal('my-modal')

// Open via async ActionButton (fetches content, then opens)
ActionButton::make('Open', '/endpoint')
    ->async(events: [AlpineJs::event(JsEvent::MODAL_TOGGLED, 'my-modal')])
```

**JavaScript triggers:**
```js
// Native JS
document.dispatchEvent(new CustomEvent("modal_toggled:my-modal"))

// Alpine.js
this.$dispatch('modal_toggled:my-modal')

// MoonShine global
MoonShine.ui.toggleModal('my-modal')
```

### Open/Close Events

Fire additional events when modal opens or closes.

```php
Modal::make('Title', asyncUrl: '/')
    ->name('my-modal')
    ->toggleEvents(
        [AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Hello'])],
        onlyOpening: false,
        onlyClosing: true,   // only fire on close
    )
```

### Default State

```php
// Open on page load
Modal::make('Title', 'Content')->open()

// Conditional
Modal::make('Title', 'Content')->open(fn() => session()->has('show_modal'))
```

### Close Behavior

```php
// Prevent closing when clicking outside
Modal::make('Title', 'Content')->closeOutside(false)

// Prevent auto-close after successful async request
Modal::make('Demo', fn() => FormBuilder::make(route('save'))
    ->fields([Text::make('Name')])
    ->async()
)->name('demo-modal')->autoClose(false)
```

### Width

```php
// Maximum width
Modal::make('Title', 'Content')->wide()

// Auto width (based on content)
Modal::make('Title', 'Content')->auto()
```

### Async Content

```php
// Load content from URL
Modal::make('Title', '', ActionButton::make('Open', '#'), asyncUrl: '/endpoint')

// Always reload on open (not cached)
Modal::make('Title', '', asyncUrl: '/endpoint')->alwaysLoad()
```

### Outer Attributes

```php
Modal::make('Title', 'Content', ActionButton::make('Open', '#'))
    ->outerAttributes(['class' => 'mt-2'])
```

### Form inside Modal

```php
Modal::make(
    'Create Item',
    static fn() => FormBuilder::make(route('items.store'))
        ->fields([
            Text::make('Name'),
            Email::make('Email'),
        ])
        ->submit('Create', ['class' => 'btn-primary'])
        ->async(),
)->name('create-modal'),

ActionButton::make('New Item')->toggleModal('create-modal')
```

### ActionButton Modal Shortcut

```php
ActionButton::make('Edit')
    ->inModal(
        title: 'Edit Item',
        content: 'Modal content here',
        name: 'edit-modal',
        builder: fn(Modal $modal, ActionButton $ctx) => $modal->wide(),
        components: [],
    )

// Unique name per table row
ActionButton::make('Delete')
    ->inModal(
        name: static fn(mixed $item, ActionButtonContract $ctx): string =>
            "delete-{$ctx->getData()?->getKey()}"
    )

// Confirmation dialog
ActionButton::make('Delete', '/delete')
    ->withConfirm(
        title: 'Confirm Delete',
        content: 'This cannot be undone.',
        button: 'Delete Now',
        fields: [Password::make('Password')],  // optional extra fields
        method: HttpMethod::DELETE,
        formBuilder: fn(FormBuilder $form) => $form,
        modalBuilder: fn(Modal $modal) => $modal->wide(),
        name: 'confirm-delete',
    )
```

### Event with Parameters

```php
Modal::make('Modal', fn() => FormBuilder::make()->fields([
    Text::make('Title')->class('title'),
    Div::make()->class('div-content'),
])),

ActionButton::make('Open')
    ->dispatchEvent(
        AlpineJs::event(
            JsEvent::MODAL_TOGGLED, 'default',
            EventParams::make()
                ->selectors(['.div-content' => 'test'])
                ->fieldsValues(['.title' => 'test-value'])
        )
    )
```

### Blade

```blade
<x-moonshine::modal title="Title">
    <div>Content</div>
    <x-slot name="outerHtml">
        <x-moonshine::link-button @click.prevent="toggleModal">
            Open modal
        </x-moonshine::link-button>
    </x-slot>
</x-moonshine::modal>

<!-- Wide -->
<x-moonshine::modal wide title="Title">...</x-moonshine::modal>

<!-- Auto width -->
<x-moonshine::modal auto title="Title">...</x-moonshine::modal>

<!-- No close on outside click -->
<x-moonshine::modal :closeOutside="false" title="Title">...</x-moonshine::modal>

<!-- Async content -->
<x-moonshine::modal async :asyncUrl="route('async')" title="Title">
    <x-slot name="outerHtml">
        <x-moonshine::link-button @click.prevent="toggleModal">Open</x-moonshine::link-button>
    </x-slot>
</x-moonshine::modal>
```

---

## OffCanvas

**Class:** `MoonShine\UI\Components\OffCanvas`
**Blade:** `<x-moonshine::off-canvas>`

Side panel that slides in from the edge of the screen.

### Constructor

```php
OffCanvas::make(
    Closure|string $title = '',
    Closure|Renderable|string $content = '',
    Closure|string $toggler = '',
    Closure|string|null $asyncUrl = null,
    iterable $components = [],
)
```

### Basic Usage

```php
OffCanvas::make(
    'Filters',
    static fn() => FormBuilder::make(route('filter'))
        ->fields([Select::make('Status')->options($opts)])
        ->submit('Apply'),
    'Show Filters'
)
```

### Naming and Events

```php
OffCanvas::make('Title', 'Content')
    ->name('my-canvas'),

// Open via ActionButton
ActionButton::make('Open')->toggleOffCanvas('my-canvas')

// Async open
ActionButton::make('Open', '/endpoint')
    ->async(events: [AlpineJs::event(JsEvent::OFF_CANVAS_TOGGLED, 'my-canvas')])
```

**JavaScript triggers:**
```js
document.dispatchEvent(new CustomEvent("off_canvas_toggled:my-canvas"))
this.$dispatch('off_canvas_toggled:my-canvas')
MoonShine.ui.toggleOffCanvas('my-canvas')
```

### Open/Close Events

```php
OffCanvas::make('Panel', asyncUrl: '/')
    ->name('my-panel')
    ->left()
    ->toggleEvents(
        [AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Panel opened'])],
        onlyOpening: true,
        onlyClosing: false,
    )
```

### Position

```php
->left()   // left side (default is right)
```

### Width

```php
->wide()   // wider than default
->full()   // maximum width (full screen)
```

### Default State

```php
->open()              // show on page load
->open(fn() => true)  // conditional
```

### Async Content

```php
OffCanvas::make('Title', '', 'Open', asyncUrl: '/endpoint')

// Always reload
OffCanvas::make(...)->alwaysLoad()
```

### Toggler Attributes

```php
OffCanvas::make('Title', 'Content', 'Open')
    ->togglerAttributes(['class' => 'mt-2'])
```

### ActionButton Shortcut

```php
ActionButton::make('Open')
    ->inOffCanvas(
        title: fn() => 'Panel Title',
        content: fn() => 'Panel Content',
        name: 'my-panel',
        builder: fn(OffCanvas $oc, ActionButton $ctx) => $oc->left()->wide(),
        components: [],
    )
```

### Blade

```blade
<x-moonshine::off-canvas title="Offcanvas" :left="false">
    <x-slot:toggler>Open</x-slot:toggler>
    Panel content here
</x-moonshine::off-canvas>
```

---

## Dropdown

**Class:** `MoonShine\UI\Components\Dropdown`
**Blade:** `<x-moonshine::dropdown>`

Dropdown block with optional search, items list, and footer.

### Constructor

```php
Dropdown::make(
    ?string $title = null,
    Closure|string $toggler = '',
    Closure|Renderable|string $content = '',
    Closure|array $items = [],
    bool $searchable = false,
    Closure|string $searchPlaceholder = '',
    string $placement = 'bottom-start',
    Closure|string $footer = '',
)
```

```php
Dropdown::make(
    'Menu',
    'Click Me',
    'Content inside dropdown',
    ['Item 1', 'Item 2', 'Item 3']
)

// Searchable dropdown
Dropdown::make(title: 'Select', searchable: true, items: $items)

// Custom placement (uses popper.js positions)
Dropdown::make(placement: 'left')
```

### Blade

```blade
<x-moonshine::dropdown title="Actions" placement="bottom-start">
    <div class="m-4">Content</div>
    <x-slot:toggler>Click me</x-slot:toggler>
    <x-slot:footer>Footer text</x-slot:footer>
</x-moonshine::dropdown>
```

Available placements: `top`, `top-start`, `top-end`, `bottom`, `bottom-start`, `bottom-end`, `left`, `left-start`, `left-end`, `right`, `right-start`, `right-end`. See [popper.js docs](https://popper.js.org/docs/v2/constructors/#options).

---

## Popover

**Class:** `MoonShine\UI\Components\Popover`
**Blade:** `<x-moonshine::popover>`

Tooltip-like popup that appears on hover.

### Constructor

```php
Popover::make(
    string $title,
    string $trigger,
    string $placement = 'right',
)
```

```php
Popover::make('Help Text', 'Hover here')
    ->content('Detailed HTML explanation')
```

### Blade

```blade
<x-moonshine::popover title="Help" placement="right">
    <x-slot:trigger>
        <button class="btn">Hover me</button>
    </x-slot:trigger>
    <p>Popover content with HTML support.</p>
</x-moonshine::popover>
```

### Without Component (Inline)

```html
<span x-data="popover" data-content="HTML content here">
    <a>Hover text</a>
</span>

<span x-data="popover({placement: 'top'})" title="Title" data-content="Content">
    <a>Hover text</a>
</span>
```

Placement options use [tippy.js positions](https://atomiks.github.io/tippyjs/v6/all-props/#placement).

---

## Toast (via Events)

Toasts are triggered via JS events, not as standalone components. They require the `Flash` component in the layout.

### From Controller (session)

```php
session()->flash('toast', [
    'type' => FlashType::SUCCESS->value,  // info, success, warning, error
    'message' => 'Record saved!',
]);
```

### From Async Response

```php
return MoonShineJsonResponse::make()->toast('Success!', ToastType::SUCCESS);
```

### Via JS Event

```php
AlpineJs::event(JsEvent::TOAST, params: ToastEventParams::make(ToastType::SUCCESS, 'Done!'))

// Or inline params
AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Hello', 'type' => 'success'])
```

### In Async Event Chains

```php
FormBuilder::make('/save')
    ->name('my-form')
    ->async(events: [
        AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Saved', 'type' => 'success']),
    ])

TableBuilder::make()
    ->name('my-table')
    ->async(events: [
        AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Loaded', 'type' => 'info']),
    ])
```
