# JavaScript Events & Alpine.js Reference

MoonShine v3 JavaScript events system, event parameters, fragment updates, Alpine.js integration, and async request flow.

## JavaScript Events Reference

MoonShine's frontend uses custom DOM events dispatched via `CustomEvent` to coordinate component updates. All events follow the naming convention `event_name:component_name`.

### JsEvent Enum (PHP)

```php
namespace MoonShine\Support\Enums;

enum JsEvent: string
{
    case FRAGMENT_UPDATED = 'fragment_updated';
    case TABLE_UPDATED = 'table_updated';
    case TABLE_REINDEX = 'table_reindex';
    case TABLE_ROW_UPDATED = 'table_row_updated';
    case CARDS_UPDATED = 'cards_updated';
    case FORM_RESET = 'form_reset';
    case FORM_SUBMIT = 'form_submit';
    case MODAL_TOGGLED = 'modal_toggled';
    case OFF_CANVAS_TOGGLED = 'off_canvas_toggled';
    case POPOVER_TOGGLED = 'popover_toggled';
    case TOAST = 'toast';
    case SHOW_WHEN_REFRESH = 'show_when_refresh';
}
```

### Event Details

#### FRAGMENT_UPDATED

Triggers a Fragment component to re-fetch its content from the server.

```php
// PHP
AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'my-fragment')
```

```js
// JS
dispatchEvent(new CustomEvent('fragment_updated:my-fragment'))
```

The Fragment component (`Fragment.js`) listens for this event and calls `fragmentUpdate()`, which:
1. Sends a GET request to `asyncUpdateRoute`.
2. Replaces its own `$root.outerHTML` with the response.
3. Optionally forwards additional events.

#### TABLE_UPDATED

Triggers an async TableBuilder to reload its data.

```php
AlpineJs::event(JsEvent::TABLE_UPDATED, 'index')
```

The TableBuilder component calls `asyncRequest()` -> `listComponentRequest()`, which:
1. Sends a GET request to `asyncUrl`.
2. Replaces the table's `$root.outerHTML` with the response.
3. Updates browser history state if `pushState` is enabled.

#### TABLE_ROW_UPDATED

Updates a single row in the table without full reload.

```php
AlpineJs::event(JsEvent::TABLE_ROW_UPDATED, 'index-{row-id}')
```

The `{row-id}` placeholder is resolved at dispatch time from the table row's `data-row-key` attribute.

#### TABLE_REINDEX

Triggers reindexing of form element names in an editable table after rows are reordered.

```php
AlpineJs::event(JsEvent::TABLE_REINDEX, 'my-table')
```

#### CARDS_UPDATED

Triggers a CardsBuilder component to reload its data (same pattern as TABLE_UPDATED).

```php
AlpineJs::event(JsEvent::CARDS_UPDATED, 'my-cards')
```

#### FORM_RESET

Resets all form fields to their initial values.

```php
AlpineJs::event(JsEvent::FORM_RESET, 'main-form')
```

The FormBuilder component calls `formReset()`, which:
1. Calls `this.$el.reset()`.
2. Dispatches a `reset` event on each form element.
3. Removes elements marked with `data-remove-on-form-reset`.

#### FORM_SUBMIT

Programmatically triggers form submission.

```php
AlpineJs::event(JsEvent::FORM_SUBMIT, 'main-form')
```

#### MODAL_TOGGLED

Opens or closes a Modal component.

```php
AlpineJs::event(JsEvent::MODAL_TOGGLED, 'my-modal')
```

The Modal component calls `toggleModal()`, which:
1. Toggles the `open` state.
2. If opening with `asyncUrl`, loads content via AJAX.
3. Dispatches opening/closing events from the modal's dataset.

Programmatic access from JS:
```js
MoonShine.ui.toggleModal('my-modal')
```

#### OFF_CANVAS_TOGGLED

Opens or closes an OffCanvas side panel.

```php
AlpineJs::event(JsEvent::OFF_CANVAS_TOGGLED, 'filter-panel')
```

```js
MoonShine.ui.toggleOffCanvas('filter-panel')
```

#### POPOVER_TOGGLED

Opens or closes a Popover.

```php
AlpineJs::event(JsEvent::POPOVER_TOGGLED, 'my-popover')
```

#### TOAST

Triggers a toast notification.

```php
AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Done!', 'type' => 'success'])
```

```js
MoonShine.ui.toast('Done!', 'success')
```

Toast types: `default`, `success`, `error`, `warning`, `info`.

#### SHOW_WHEN_REFRESH

Re-evaluates `showWhen` conditions on form fields after dynamic changes.

```php
AlpineJs::event(JsEvent::SHOW_WHEN_REFRESH, 'main-form')
```

---

## Event Parameters

Events can carry parameters using the `EventParams` class:

```php
use MoonShine\Support\EventParams;

AlpineJs::event(
    JsEvent::TABLE_UPDATED,
    'index',
    EventParams::make()
        ->selectors(['#my-selector' => '<p>New HTML</p>'])
        ->fieldsValues(['title' => 'Updated'])
        ->delay(500)
)
```

### EventParams Methods

| Method | Description |
|---|---|
| `selectors(array $data)` | Key-value pairs of CSS selectors and HTML content for DOM updates |
| `fieldsValues(array $data)` | Key-value pairs of field names and values to set |
| `delay(int $ms)` | Delay event dispatch by specified milliseconds |

The event string format with parameters:
```
event_name:component_name|key1=value1;key2=value2
```

---

## Fragment-Based Partial Updates

Fragments enable partial page updates without full page reload.

### PHP Setup

```php
use MoonShine\UI\Components\Fragment;
use MoonShine\UI\Components\Table\TableBuilder;

Fragment::make([
    TableBuilder::make()
        ->fields([ID::make(), Text::make('Title')])
        ->items($items)
        ->name('data-table')
])->name('data-fragment')
```

### How Fragments Work (JS Internals)

The `Fragment.js` Alpine component:

1. Stores the `asyncUpdateRoute` for re-fetching content.
2. Listens for the `fragment_updated:{name}` event.
3. On event, sends a GET request to `asyncUpdateRoute`.
4. Replaces its own `$root.outerHTML` with the server response.
5. Supports `withParams` (data from other selectors) and `withQueryParams` (forward URL query string).

### Fragment with Parameters

```php
Fragment::make([
    // content
])
->name('stats-fragment')
->updateWith(['#date-filter' => 'date'])  // include data from other elements
->withQueryParams()                        // forward current URL query params
```

### Triggering Fragment Updates

From PHP:
```php
ActionButton::make('Refresh')
    ->dispatchEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'stats-fragment'))
```

From JS:
```js
this.$dispatch('fragment_updated:stats-fragment')
```

From a response:
```php
return MoonShineJsonResponse::make()
    ->events([AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'stats-fragment')]);
```

---

## Alpine.js Integration Details

### Initialization

MoonShine bundles Alpine.js and initializes it automatically. The setup sequence in `app.js`:

1. `window.MoonShine = new MoonShine()` -- Global JS class instantiated.
2. `moonshine:init` event dispatched -- Register callbacks here.
3. All Alpine.js components registered via `Alpine.data()`.
4. `alpine:init` event fires -- Alpine.js-specific initialization.
5. `Alpine.start()` called (only if no prior Alpine instance exists).

### Registered Alpine Components

MoonShine registers these Alpine.js components automatically:

| Alpine Data Name | Source Component |
|---|---|
| `formBuilder` | FormBuilder (async forms, reactive fields, filters) |
| `tableBuilder` | TableBuilder (async tables, sorting, pagination) |
| `cardsBuilder` | CardsBuilder (async card grids) |
| `modal` | Modal (toggle, async content loading) |
| `offCanvas` | OffCanvas (side panels) |
| `actionButton` | ActionButton (async actions, methods) |
| `fragment` | Fragment (partial page updates) |
| `select` | Select (choice.js integration) |
| `dropdown` | Dropdown |
| `popover` | Popover |
| `toasts` | Toast notifications |
| `tooltip` / `navTooltip` | Tooltips |
| `belongsToMany` | BelongsToMany field |
| `range` | Range slider |
| `sortable` | Drag-and-drop sorting |
| `queryTag` | Query tag filters |
| `tabs` | Tabs |
| `collapse` | Collapsible sections |
| `carousel` | Carousel/slider |
| `global` | Global helpers |

### Alpine.js Plugins

MoonShine loads two Alpine.js plugins:
- `@alpinejs/persist` -- Persist state across page loads (used for dark mode).
- `@alpinejs/mask` -- Input masking.

### Creating Custom Components

Register custom Alpine.js components after the `alpine:init` event:

```js
document.addEventListener("alpine:init", () => {
    Alpine.data("myWidget", () => ({
        count: 0,
        init() {
            // Runs when component mounts
        },
        increment() {
            this.count++
        },
    }))
})
```

Use in Blade:
```html
<div x-data="myWidget">
    <span x-text="count"></span>
    <button @click="increment">+1</button>
</div>
```

### Working with the AlpineJs PHP Helper

The `MoonShine\Support\AlpineJs` class provides static methods for building event strings and data attributes used by the JS layer.

#### Key Methods

```php
// Build an event string
AlpineJs::event(JsEvent::TABLE_UPDATED, 'my-table')
// => "table_updated:my-table"

// Build a Blade x-on directive
AlpineJs::eventBlade(JsEvent::FORM_RESET, 'main-form', 'formReset')
// => "@form_reset:main-form.window='formReset'"

// Conditional Blade directive
AlpineJs::eventBladeWhen($condition, JsEvent::MODAL_TOGGLED, 'my-modal')

// Build data attributes for async components
AlpineJs::asyncUrlDataAttributes(
    method: HttpMethod::POST,
    events: [AlpineJs::event(JsEvent::TABLE_UPDATED, 'index')],
    selector: '#my-container',
    callback: AsyncCallback::with(afterResponse: 'myHandler')
)
// Returns: ['data-async-events' => '...', 'data-async-selector' => '...', ...]

// Dispatch multiple events from Alpine
AlpineJs::dispatchEvents([
    AlpineJs::event(JsEvent::TABLE_UPDATED, 'index'),
    AlpineJs::event(JsEvent::FORM_RESET, 'main-form'),
])
// => "$dispatch('table_updated:index');$dispatch('form_reset:main-form')"
```

### Event Separators (Internal)

The `AlpineJs` class uses these separators to compose event strings:

| Constant | Value | Purpose |
|---|---|---|
| `EVENT_SEPARATOR` | `:` | Separates event name from component name |
| `EVENT_PARAMS_SEPARATOR` | `\|` | Separates event string from parameters |
| `EVENT_PARAM_SEPARATOR` | `;` | Separates individual parameter pairs |

Full event string format:
```
event_name:component_name|param1=value1;param2=value2
```

---

## Async Request Flow (JS Internals)

Understanding the request lifecycle helps when building custom integrations.

### Request Core (`Request/Core.js`)

The central `request()` function handles all async communication:

1. **Validation** -- Checks for URL and network connectivity.
2. **Before request** -- Calls registered `beforeRequest` callback if set.
3. **HTTP request** -- Sends request via axios.
4. **Response processing**:
   a. Calls `beforeHandleResponse` callback.
   b. If `responseHandler` is set, delegates entirely to it and returns.
   c. Otherwise: processes `htmlData` for DOM updates via `DOMUpdater`.
   d. Shows toast message if present.
   e. Dispatches events from response or component config.
   f. Calls `afterResponse` callback.
   g. Handles redirect if present.
5. **Error handling** -- Shows error toast, calls `errorCallback`.

### DOM Updater

The `DOMUpdater` function processes `htmlData` arrays from responses:

```js
DOMUpdater({
    htmlData: [{html: '<p>New content</p>', selector: '#target', htmlMode: 'innerHTML'}],
    fields_values: {title: 'New Value'},
})
```

It updates DOM elements matching selectors and sets form field values.

### ComponentRequestData

The `ComponentRequestData` DTO configures async request behavior:

```js
let data = new ComponentRequestData()
data
    .withEvents('table_updated:index')
    .withSelector('#my-container')
    .withBeforeRequest('myBeforeHandler')
    .withResponseHandler('myHandler')
    .withAfterResponse(function(data, type) { /* ... */ })
    .withErrorCallback(function(data, t) { /* ... */ })
    .withResponseType('blob')  // for file downloads
```

This DTO can be populated from a dataset (HTML data attributes) or a plain object:

```js
data.fromDataset(element.dataset)
// Reads: data-async-events, data-async-selector, data-async-response-handler,
//        data-async-before-request, data-async-response-type
```
