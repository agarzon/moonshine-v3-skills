# Layout Components

## Box

**Class:** `MoonShine\UI\Components\Layout\Box`
**Blade:** `<x-moonshine::layout.box>`

A styled container for highlighting content areas.

```php
Box::make(
    Closure|string|iterable $labelOrComponents = [],
    iterable $components = []
)
```

```php
// Without heading
Box::make([
    Alert::make()->content('Important notice'),
    Text::make('Name'),
])

// With heading
Box::make('User Details', [
    Text::make('Name'),
    Email::make('Email'),
])

// Dark style
Box::make(['Content'])->dark()

// With icon
Box::make('Settings', $fields)->icon('cog')
```

**Blade:**
```blade
<x-moonshine::layout.box title="Title" dark>
    Content
</x-moonshine::layout.box>
```

---

## Grid

**Class:** `MoonShine\UI\Components\Layout\Grid`
**Blade:** `<x-moonshine::layout.grid>`

12-column grid system for positioning elements.

```php
Grid::make(
    iterable $components = [],
    int $gap = 6,
)
```

```php
Grid::make([
    Column::make([Text::make('Left')], colSpan: 6),
    Column::make([Text::make('Right')], colSpan: 6),
])

// Custom gap
Grid::make($components, gap: 4)
```

---

## Column

**Class:** `MoonShine\UI\Components\Layout\Column`
**Blade:** `<x-moonshine::layout.column>`

A column within a Grid. The grid is 12 columns wide.

```php
Column::make(
    iterable $components = [],
    int $colSpan = 12,          // desktop (>= 1280px)
    int $adaptiveColSpan = 12,  // mobile (< 1280px)
)
```

```php
Grid::make([
    Column::make([Box::make('A')], colSpan: 4, adaptiveColSpan: 12),
    Column::make([Box::make('B')], colSpan: 4, adaptiveColSpan: 6),
    Column::make([Box::make('C')], colSpan: 4, adaptiveColSpan: 6),
])
```

**Blade:**
```blade
<x-moonshine::layout.grid>
    <x-moonshine::layout.column adaptiveColSpan="6" colSpan="6">
        Left content
    </x-moonshine::layout.column>
    <x-moonshine::layout.column adaptiveColSpan="6" colSpan="6">
        Right content
    </x-moonshine::layout.column>
</x-moonshine::layout.grid>
```

---

## Flex

**Class:** `MoonShine\UI\Components\Layout\Flex`
**Blade:** `<x-moonshine::layout.flex>`

Flexbox layout for inline positioning.

```php
Flex::make(
    iterable $components = [],
    int $colSpan = 12,
    int $adaptiveColSpan = 12,
    string $itemsAlign = 'center',     // Tailwind items-{value}
    string $justifyAlign = 'center',   // Tailwind justify-{value}
    bool $withoutSpace = false
)
```

```php
Flex::make([
    Text::make('First'),
    Text::make('Second'),
])
    ->justifyAlign('between')
    ->itemsAlign('start')
    ->wrap()  // enable flex-wrap
```

**Blade:**
```blade
<x-moonshine::layout.flex :justifyAlign="'end'">
    <div>Item 1</div>
    <div>Item 2</div>
</x-moonshine::layout.flex>
```

---

## Div

**Class:** `MoonShine\UI\Components\Layout\Div`
**Blade:** `<x-moonshine::layout.div>`

Simple `<div>` wrapper with nested components and attribute support.

```php
Div::make(iterable $components)
```

```php
Div::make([Text::make('Content')])
    ->class('my-wrapper')
    ->xData(['visible' => true])
```

---

## Divider

**Class:** `MoonShine\UI\Components\Layout\Divider`
**Blade:** `<x-moonshine::layout.divider>`

Horizontal separator line between content areas.

```php
Divider::make()                    // plain line
Divider::make('Section Title')     // with label
Divider::make('Center')->centered() // centered label
```

**Blade:**
```blade
<x-moonshine::layout.divider />
```

---

## LineBreak

**Class:** `MoonShine\UI\Components\Layout\LineBreak`
**Blade:** `<x-moonshine::layout.line-break>`

Vertical spacing between elements.

```php
LineBreak::make()
```

---

## Tabs / Tab

**Class:** `MoonShine\UI\Components\Tabs`, `MoonShine\UI\Components\Tabs\Tab`
**Blade:** `<x-moonshine::tabs>`

### Creating Tabs

```php
Tabs::make(iterable $components = [])
Tab::make(Closure|string|iterable $labelOrComponents = [], iterable $components = [])
```

```php
Tabs::make([
    Tab::make('General', [
        Text::make('Title'),
        Textarea::make('Description'),
    ]),
    Tab::make('Settings', [
        Switcher::make('Active'),
        Number::make('Sort Order'),
    ]),
])
```

### Active Tab

```php
Tabs::make([
    Tab::make('Tab 1', [...])->active(),
    Tab::make('Tab 2', [...]),
])

// Conditional active
Tab::make('Tab 1', [...])->active(session()->has('key'))
```

### Vertical Mode

```php
Tabs::make([...])->vertical()
```

### Label Attributes

```php
Tab::make('Tab 1', [...])->labelAttributes(['x-show' => '!flag'])
```

**Blade:**
```blade
<x-moonshine::tabs
    :items="['general' => 'General', 'settings' => 'Settings']"
    active="settings"
>
    <x-slot:general>General content</x-slot:general>
    <x-slot:settings>Settings content</x-slot:settings>
</x-moonshine::tabs>
```

> Use `snake_case` for tab keys in Blade.

---

## Collapse

**Class:** `MoonShine\UI\Components\Collapse`
**Blade:** `<x-moonshine::collapse>`

Collapsible content block. Component state is preserved when collapsed.

```php
Collapse::make(
    Closure|string $label = '',
    iterable $components = [],
    bool $open = false,
    bool $persist = true   // remember open/closed state
)
```

```php
Collapse::make('Advanced Settings', [
    Text::make('Slug'),
    Text::make('Meta Title'),
])

// Start expanded
Collapse::make('Details', $fields)->open()

// Disable state persistence
Collapse::make('Temporary', $fields)->persist(false)

// With icon
Collapse::make('Settings', $fields)->icon('cog')
```

---

## Card

**Class:** `MoonShine\UI\Components\Card`
**Blade:** `<x-moonshine::card>`

A styled card element for displaying content.

```php
Card::make(
    Closure|string $title = '',
    Closure|array|string $thumbnail = '',
    Closure|string $url = '#',
    Closure|array $values = [],
    Closure|string|null $subtitle = null,
    bool $overlay = false,
)
```

```php
Card::make(
    title: 'Article Title',
    thumbnail: '/images/cover.jpg',
    url: fn() => route('article.show', $id),
    values: ['Author' => 'John', 'Date' => '2024-01-15'],
    subtitle: 'Published',
)

// Fluent methods
Card::make(title: 'Title')
    ->header(static fn() => Badge::make('new', 'success'))
    ->subtitle('Subtitle text')
    ->thumbnail(['/img/1.jpg', '/img/2.jpg'])  // carousel
    ->values(['Key' => 'Value'])
    ->url('/link')
    ->actions(static fn() => ActionButton::make('Edit', '/edit'))
```

**Blade:**
```blade
<x-moonshine::card
    title="Title"
    thumbnail="/images/cover.jpg"
    url="/link"
    subtitle="Subtitle"
    :values="['ID' => 1, 'Author' => 'John']"
>
    Card body content
    <x-slot:header>
        <x-moonshine::badge color="green">new</x-moonshine::badge>
    </x-slot:header>
    <x-slot:actions>
        <x-moonshine::link-button href="#">Action</x-moonshine::link-button>
    </x-slot:actions>
</x-moonshine::card>
```

---

## Heading

**Class:** `MoonShine\UI\Components\Heading`
**Blade:** `<x-moonshine::heading>`

Content heading with configurable gradation (h1-h6).

```php
Heading::make(
    Closure|string $label = '',
    ?int $h = null,
    bool $asClass = true,  // true = <div class="h1">, false = <h1>
)
```

```php
Heading::make('Page Title', 1)            // <div class="h1">
Heading::make('Section')->h(2)            // <div class="h2">
Heading::make('Real H3')->h(3, false)     // <h3>
Heading::make('Title', 1)->tag('p')       // <p class="h1">
```

**Blade:**
```blade
<x-moonshine::heading h="2">Section Title</x-moonshine::heading>
```

---

## Title

**Class:** `MoonShine\UI\Components\Title`
**Blade:** `<x-moonshine::title>`

Main page header component.

```php
Title::make(Closure|string|null $value, int $h = 1)
```

```php
Title::make('Dashboard')
Title::make($page->getTitle())
```

---

## Wrapper

**Class:** `MoonShine\UI\Components\Layout\Wrapper`
**Blade:** `<x-moonshine::layout.wrapper>`

Wraps sidebar and content to ensure correct layout. Used immediately after `Body`.

```php
Body::make([
    Wrapper::make([
        Sidebar::make([...]),
        Content::make([...]),
    ])
])
```

---

## Content

**Class:** `MoonShine\UI\Components\Layout\Content`
**Blade:** `<x-moonshine::layout.content>`

The main content area of the page.

```php
Content::make([
    Title::make($page->getTitle()),
    Components::make($page->getComponents()),
])
```

---

## Flash

**Class:** `MoonShine\UI\Components\Layout\Flash`
**Blade:** (part of layout)

Displays session-based notifications (alerts and toasts).

```php
Flash::make(
    string $key = 'alert',
    string|FlashType $type = FlashType::INFO,
    bool $withToast = true,
    bool $removable = true
)
```

```php
Flash::make()  // displays session('alert') and session('toast')
```

Setting toast in controller:
```php
session()->flash('toast', [
    'type' => FlashType::INFO->value,
    'message' => 'Operation complete',
]);
```

Async toast via events:
```php
AlpineJs::event(
    JsEvent::TOAST,
    params: ToastEventParams::make(ToastType::SUCCESS, 'Done!')
)
```

---

## Fragment

**Class:** `MoonShine\Laravel\Components\Fragment`

Wraps a page area for partial async updates using Blade Fragments.

```php
Fragment::make(iterable $components = [])
```

```php
// Define a fragment
Fragment::make([
    TableBuilder::make()->fields($fields)->items($items)
])->name('my-fragment')

// Update fragment on form success
FormBuilder::make()->async(events: AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'my-fragment'))

// Pass extra params with update request
Fragment::make($components)
    ->name('my-fragment')
    ->updateWith(params: ['resourceItem' => request('resourceItem')])

// Pass field values via selectors
Fragment::make($components)
    ->withSelectorsParams(['start_date' => '#start_date', 'end_date' => '#end_date'])
    ->name('my-fragment')

// Chain fragment updates
Fragment::make([FlexibleRender::make('<p>Step 1</p>')])
    ->updateWith(events: [AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'step-2')])
    ->name('step-1'),

Fragment::make([FlexibleRender::make('<p>Step 2</p>')])
    ->name('step-2'),

ActionButton::make('Start')
    ->dispatchEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'step-1'))

// Include current URL query params
Fragment::make($components)->name('frag')->withQueryParams()

// Response callback
Fragment::make($components)
    ->updateWith(callback: AsyncCallback::with(afterResponse: 'myJsFunction'))
    ->name('frag')
```

---

## Components

**Class:** `MoonShine\UI\Components\Components`

No visual representation; outputs a set of components.

```php
Components::make([
    Box::make([...]),
    Alert::make()->content('Notice'),
])
```

---

## When

**Class:** `MoonShine\UI\Components\When`

Conditionally displays components based on a runtime condition.

```php
When::make(
    Closure $condition,
    Closure $components,
    ?Closure $default = null
)
```

```php
When::make(
    static fn() => config('moonshine.auth.enabled', true),
    static fn() => [Profile::make(withBorder: true)],
    static fn() => [FlexibleRender::make('Auth disabled')],
)
```

---

## FieldsGroup

**Class:** `MoonShine\UI\Components\FieldsGroup`

Groups fields for batch operations (fill, preview mode, etc.).

```php
FieldsGroup::make([Text::make('Title'), Email::make('Email')])
    ->fill($data)
    ->previewMode()
    ->withoutWrappers()
    ->mapFields(fn(FieldContract $field, int $index) => $field)
```
