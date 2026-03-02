# SDUI (Server-Driven UI) Reference

MoonShine v3 Server-Driven UI concept, response structure, request headers, component types, and state management.

## SDUI Concept

Server-Driven UI (SDUI) is an approach where the server defines the entire UI structure as a JSON component tree. The client application (mobile, SPA, or desktop) receives this tree and renders it using its own native components. MoonShine provides built-in SDUI support, allowing any page to be requested as a structured JSON response.

> SDUI is currently in beta testing while the MoonShine team gathers community feedback.

### When to Use SDUI

- Building mobile apps that consume MoonShine admin panel structure.
- Creating custom SPA frontends that render MoonShine pages natively.
- Building tools that need to inspect or analyze the component hierarchy.
- Implementing dynamic UI that adapts based on server-defined layouts.

---

## SDUI Response Structure

Every MoonShine UI component can be represented as a JSON object with four key properties:

| Property | Type | Description |
|---|---|---|
| `type` | string | Component class name (e.g., `Dashboard`, `Card`, `Heading`, `Text`) |
| `components` | array | Child components (recursive tree structure) |
| `states` | object | Component state data (title, content, level, etc.) |
| `attributes` | object | HTML attributes (class, id, data-* attributes) |

### Example Response

```json
{
    "type": "Dashboard",
    "components": [
        {
            "type": "Card",
            "components": [
                {
                    "type": "Heading",
                    "states": {
                        "level": 1,
                        "content": "Welcome to Dashboard"
                    },
                    "attributes": {
                        "class": ["text-2xl", "font-bold"],
                        "id": "dashboard-heading"
                    }
                },
                {
                    "type": "Text",
                    "states": {
                        "content": "Here's an overview of your system."
                    },
                    "attributes": {
                        "class": ["mt-2", "text-gray-600"]
                    }
                }
            ],
            "states": {
                "title": "Dashboard Overview"
            },
            "attributes": {
                "class": ["bg-white", "shadow", "rounded-lg"],
                "data-card-id": "dashboard-overview"
            }
        }
    ],
    "states": {
        "title": "Admin Dashboard"
    },
    "attributes": {
        "class": ["container", "mx-auto", "py-6"]
    }
}
```

---

## Request Headers

### Basic Structure Request

```http
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
```

Returns the full component tree including layout, page content, states, and attributes.

### Control Headers

| Header | Value | Effect |
|---|---|---|
| `X-MS-Structure` | `true` | Enable SDUI JSON response |
| `X-MS-Without-States` | `true` | Omit `states` from all components |
| `X-MS-Only-Layout` | `true` | Return only the layout structure (sidebar, header, etc.) |
| `X-MS-Without-Layout` | `true` | Return page content without the layout wrapper |

### Retrieve Structure Without States

Useful when you only need the component hierarchy for layout purposes:

```http
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
X-MS-Without-States: true
```

Response (states omitted):

```json
{
    "type": "Dashboard",
    "components": [
        {
            "type": "Card",
            "components": [
                {
                    "type": "Heading",
                    "attributes": {
                        "class": ["text-2xl", "font-bold"],
                        "id": "dashboard-heading"
                    }
                },
                {
                    "type": "Text",
                    "attributes": {
                        "class": ["mt-2", "text-gray-600"]
                    }
                }
            ],
            "attributes": {
                "class": ["bg-white", "shadow", "rounded-lg"],
                "data-card-id": "dashboard-overview"
            }
        }
    ],
    "attributes": {
        "class": ["container", "mx-auto", "py-6"]
    }
}
```

### Retrieve Only Layout

```http
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
X-MS-Only-Layout: true
```

Returns the layout shell (sidebar, header, footer) without page-specific content.

### Retrieve Page Without Layout

```http
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
X-MS-Without-Layout: true
```

Returns only the page content, omitting the layout wrapper.

---

## Component Types

MoonShine components that appear in SDUI responses include (non-exhaustive):

### Layout Components
- `Dashboard`, `Layout`, `Sidebar`, `Header`, `Footer`, `TopBar`
- `Grid`, `Column`, `Flex`, `Box`, `Divider`

### Content Components
- `Card`, `Heading`, `Text`, `Badge`, `Alert`, `Icon`, `Link`
- `Tabs`, `Tab`, `Collapse`

### Data Components
- `TableBuilder`, `FormBuilder`, `CardsBuilder`
- `Fragment`

### Overlay Components
- `Modal`, `OffCanvas`, `Dropdown`, `Popover`

### Action Components
- `ActionButton`, `ActionGroup`

### Metric Components
- `ValueMetric`, `DonutChartMetric`, `LineChartMetric`

### Field Components
- `Text`, `Textarea`, `Select`, `Checkbox`, `Switcher`, `Date`, `File`, etc.

---

## State Management

States contain the runtime data for each component. Common state properties by component type:

| Component | Common States |
|---|---|
| `Heading` | `level`, `content` |
| `Text` | `content` |
| `Card` | `title`, `subtitle`, `thumbnail` |
| `TableBuilder` | `items`, `fields`, `paginator`, `sortColumn`, `sortDirection` |
| `FormBuilder` | `fields`, `values`, `action`, `method` |
| `Modal` | `open`, `title` |
| `ActionButton` | `label`, `url`, `method` |

---

## Actions and Interactivity

SDUI responses describe component structure statically. For interactivity, the client must implement:

1. **Event dispatching** -- Trigger events that correspond to the `JsEvent` enum.
2. **Async requests** -- Components with `asyncUrl` or `async` states support server round-trips.
3. **Form submission** -- FormBuilder components include `action` and `method` in their states.

The client should map SDUI component types to native widgets and wire up the event system accordingly.
