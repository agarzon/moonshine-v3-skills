---
name: moonshine-frontend-v3
description: Use when working with MoonShine v3 as an API backend, JWT auth for API, SDUI (Server-Driven UI), Alpine.js events, JavaScript helpers, reactive fields, or async UI updates.
---

# MoonShine v3 -- Frontend (API, SDUI, JS & Alpine.js)

## Overview

MoonShine v3 provides a complete frontend interaction layer built on Alpine.js. It supports three modes of operation:

1. **Standard HTML** -- Server-rendered Blade views with Alpine.js reactivity (default).
2. **API mode** -- Add `Accept: application/json` to get JSON responses from CRUD routes, optionally with JWT authentication.
3. **SDUI (Server-Driven UI)** -- Request the component tree as a JSON structure for rendering in custom clients (mobile, SPA).

Alpine.js is bundled and initialized automatically. The global `MoonShine` JS class provides request helpers, UI toggles, and a callback registry for async response handling.

## API Mode

Switch any MoonShine route to return JSON by including `Accept: application/json` in the request header.

### Available CRUD Routes

```
GET    /admin/resource/{resourceUri}/crud                    -- Listing
POST   /admin/resource/{resourceUri}/crud                    -- Create
PUT    /admin/resource/{resourceUri}/crud/{resourceItem}     -- Edit
DELETE /admin/resource/{resourceUri}/crud/{resourceItem}     -- Delete
DELETE /admin/resource/{resourceUri}/crud                    -- Mass delete (ids[])
```

When using MoonShine purely as an API backend, disable session middleware in `config/moonshine.php`:

```php
'middleware' => [
    // remove session-related middleware
],
```

> See [references/api-jwt.md](references/api-jwt.md) for full API setup, JWT configuration, and OpenAPI generation.

## JWT Quick Start

Install the JWT package for token-based API authentication:

```shell
composer require moonshine/jwt
php artisan vendor:publish --provider="MoonShine\JWT\Providers\JWTServiceProvider"
```

Add a base64-encoded secret to `.env`:

```ini
JWT_SECRET=YOUR_BASE64_SECRET_HERE
```

Configure middleware and auth pipeline in `config/moonshine.php`:

```php
use MoonShine\JWT\JWTAuthPipe;
use MoonShine\JWT\Http\Middleware\AuthenticateApi;

return [
    'middleware' => [
        // empty -- no session middleware
    ],
    'auth' => [
        'middleware' => AuthenticateApi::class,
        'pipelines' => [
            JWTAuthPipe::class,
        ],
    ],
];
```

After successful authentication, use the returned token in subsequent requests:

```
Authorization: Bearer <token>
```

> Cross-reference: **moonshine-security-v3** covers JWT auth config, auth pipelines, and guard setup in detail.

## Alpine.js Events

MoonShine uses custom DOM events (dispatched via Alpine.js `$dispatch`) to coordinate UI updates between components. Every event follows the pattern `event_name:component_name`.

### Default Events (JsEvent enum)

| Event | JsEvent constant | Purpose |
|---|---|---|
| `fragment_updated:{name}` | `JsEvent::FRAGMENT_UPDATED` | Refresh a Fragment area |
| `table_updated:{name}` | `JsEvent::TABLE_UPDATED` | Refresh an async TableBuilder |
| `table_reindex:{name}` | `JsEvent::TABLE_REINDEX` | Reindex table fields after sort |
| `table_row_updated:{name}-{row-id}` | `JsEvent::TABLE_ROW_UPDATED` | Refresh a single table row |
| `cards_updated:{name}` | `JsEvent::CARDS_UPDATED` | Refresh a CardsBuilder |
| `form_reset:{name}` | `JsEvent::FORM_RESET` | Reset form fields |
| `form_submit:{name}` | `JsEvent::FORM_SUBMIT` | Trigger form submission |
| `modal_toggled:{name}` | `JsEvent::MODAL_TOGGLED` | Open/close a Modal |
| `off_canvas_toggled:{name}` | `JsEvent::OFF_CANVAS_TOGGLED` | Open/close an OffCanvas |
| `popover_toggled:{name}` | `JsEvent::POPOVER_TOGGLED` | Open/close a Popover |
| `toast:{name}` | `JsEvent::TOAST` | Trigger a Toast notification |
| `show_when_refresh:{name}` | `JsEvent::SHOW_WHEN_REFRESH` | Re-evaluate showWhen conditions |

### Dispatching Events from PHP (Backend)

Use the `AlpineJs` helper class to build event strings:

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

// Simple event
AlpineJs::event(JsEvent::TABLE_UPDATED, 'index')
// => "table_updated:index"

// Event with parameters
AlpineJs::event(JsEvent::TABLE_UPDATED, 'index', ['var' => 'foo'])
// => "table_updated:index|var=foo"

// Blade attribute format (for x-on directives)
AlpineJs::eventBlade(JsEvent::FORM_RESET, 'main-form')
// => "@form_reset:main-form.window"
```

Common usage pattern -- async form triggers table refresh:

```php
FormBuilder::make('/crud/update')
    ->name('main-form')
    ->async(events: [
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'crud-table'),
        AlpineJs::event(JsEvent::FORM_RESET, 'main-form'),
    ])
    ->fields([Text::make('Title')])
```

### Dispatching Events from JavaScript (Frontend)

Native DOM:

```js
document.addEventListener("DOMContentLoaded", () => {
    dispatchEvent(new CustomEvent("modal_toggled:my-modal"))
})
```

Alpine.js `$dispatch`:

```js
this.$dispatch('modal_toggled:my-modal')
```

### Dispatching Events via MoonShineJsonResponse

Return events from a controller or resource method so they fire after the response is processed:

```php
use MoonShine\Laravel\Http\Responses\MoonShineJsonResponse;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

return MoonShineJsonResponse::make()
    ->toast('Saved!', ToastType::SUCCESS)
    ->events([
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'index'),
    ]);
```

### Blade Directives for Events

```blade
{{-- Declares: @table-updated-index.window="asyncRequest" --}}
<div x-data="myComponent"
    @defineEvent('table-updated', 'index', 'asyncRequest')
>
</div>

{{-- With condition --}}
<div x-data="myComponent"
    @defineEventWhen($isAsync, 'table-updated', 'index', 'asyncRequest')
>
</div>
```

## Reactive Fields

MoonShine supports reactive form fields that send their values to the server on change and receive updated field HTML in return. This is handled by the `FormBuilder` Alpine.js component, which watches the `reactive` data object.

When a reactive field value changes:
1. A POST request is sent to `reactiveUrl` with `{ _component_name, values }`.
2. The server returns `{ fields: { column: html }, values: { column: value } }`.
3. The field wrapper HTML is replaced and focus is preserved on text inputs.

> Cross-reference: **moonshine-fields-v3** covers how to declare fields as reactive. **moonshine-components-v3** covers FormBuilder's `async()` and reactive configuration.

## Global MoonShine JS Class

The `window.MoonShine` object is available after the `moonshine:init` event fires.

### Request Helper

```js
MoonShine.request(ctx, '/url', method = 'get', body = {}, headers = {}, data = {})
```

### UI Helpers

```js
MoonShine.ui.toast('Hello world', 'success')
MoonShine.ui.toggleModal('modal-name')
MoonShine.ui.toggleOffCanvas('canvas-name')
```

### Iterable Helpers

```js
// Drag-and-drop reordering
MoonShine.iterable.sortable(container, url, group, events, attributes, callback)

// Reindex form element names after reorder
MoonShine.iterable.reindex(container, itemSelector)
```

### Config

```js
MoonShine.config().setToastDuration(5000)
MoonShine.config().forceRelativeUrls(true)
```

## Async Response Callbacks

Components with async behavior (ActionButton, FormBuilder, TableBuilder, fields implementing `HasAsyncContract`) support the `AsyncCallback` DTO for hooking into the request lifecycle.

### AsyncCallback Parameters

```php
use MoonShine\Support\DTOs\AsyncCallback;

ActionButton::make('Do Something')
    ->method('myMethod', callback: AsyncCallback::with(
        beforeRequest: 'myBeforeRequest',    // JS function name
        responseHandler: 'myResponseHandler', // overrides default handling
        afterResponse: 'myAfterResponse',    // called after success (if no responseHandler)
    ))
```

### Registering JS Callbacks

```js
document.addEventListener("moonshine:init", () => {
    // Before the request is sent
    MoonShine.onCallback('myBeforeRequest', function(element, component) {
        console.log('Before request', element, component)
    })

    // Full response handler (replaces default behavior)
    MoonShine.onCallback('myResponseHandler', function(response, element, events, component) {
        console.log('Custom handler', response)
    })

    // After successful default processing
    MoonShine.onCallback('myAfterResponse', function(data, messageType, component) {
        console.log('After response', data, messageType)
    })
})
```

## Fragment-Based Partial Updates

Wrap components in a `Fragment` to enable partial page updates without full reload:

```php
Fragment::make([
    TableBuilder::make()->fields($fields)->items($items)->name('data-table')
])->name('data-fragment'),

ActionButton::make('Reload')
    ->dispatchEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'data-fragment'))
```

When the `fragment_updated:data-fragment` event fires, the Fragment component makes a GET request to its `asyncUpdateRoute`, replaces its own `outerHTML` with the response, and preserves Alpine.js state.

## SDUI (Server-Driven UI)

Request any MoonShine page as a JSON component tree by adding the `X-MS-Structure: true` header. The response describes component types, states, attributes, and nested children -- suitable for rendering in mobile apps or custom SPA clients.

```
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
```

Additional control headers:

| Header | Effect |
|---|---|
| `X-MS-Structure: true` | Return JSON component tree |
| `X-MS-Without-States: true` | Omit component states from response |
| `X-MS-Only-Layout: true` | Return only the layout structure |
| `X-MS-Without-Layout: true` | Return page content without layout wrapper |

> See [references/sdui.md](references/sdui.md) for full SDUI response structure and [references/js-events-alpine.md](references/js-events-alpine.md) for JS events and Alpine.js details.

## Creating Custom Alpine.js Components

```shell
php artisan moonshine:component MyComponent
```

In the Blade view:

```html
<div x-data="myComponent"></div>
```

Register the Alpine.js component (in a separate JS file loaded via AssetManager):

```js
document.addEventListener("alpine:init", () => {
    Alpine.data("myComponent", () => ({
        init() {
            // component initialization
        },
    }))
})
```

> Alpine.js is already installed and running (`window.Alpine`). Do not re-initialize it.

## Cross-references

- **moonshine-security-v3** -- JWT auth configuration, auth pipelines, guard setup, and the `AuthenticateApi` middleware.
- **moonshine-components-v3** -- Async components (FormBuilder, TableBuilder, Fragment), event wiring, and the full component API.
- **moonshine-resources-v3** -- API endpoints for resources, CRUD routes, `MoonShineJsonResponse` modifiers, and resource method handlers.
- **moonshine-appearance-v3** -- Asset management for custom JS/CSS files.

## Reference Files

- `references/api-jwt.md` -- Full API mode setup, JWT package configuration, and OpenAPI Generator.
- `references/sdui.md` -- SDUI concept, response structure, headers, component types, and state management.
- `references/js-events-alpine.md` -- JavaScript events reference, event parameters, fragment updates, Alpine.js integration, and async request flow.
