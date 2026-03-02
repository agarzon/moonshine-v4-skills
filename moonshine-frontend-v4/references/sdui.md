# Server-Driven UI (SDUI) -- Reference (v4)

This reference provides complete documentation for MoonShine v4's Server-Driven UI (SDUI) system, including the request protocol, JSON response structure, customization headers, and examples.

> SDUI is currently in beta testing while MoonShine gathers community feedback.

---

## Concept

Server-Driven UI (SDUI) is an approach where the server defines the structure, content, and layout of the user interface. Instead of returning rendered HTML, the server returns a JSON component tree that describes what to render. The client application (mobile app, SPA, custom frontend) interprets this tree and renders the appropriate native or web components.

### Benefits

- **Dynamic UI without client updates:** Change the interface by modifying server-side components; no app store release needed.
- **Consistent cross-platform rendering:** The same JSON tree can be rendered on iOS, Android, web, or desktop.
- **Leverage existing MoonShine components:** Every MoonShine UI component can be represented as a JSON structure.
- **Gradual adoption:** Use SDUI for specific pages while keeping standard HTML rendering for others.

---

## Requesting SDUI Responses

To receive a JSON component tree instead of rendered HTML, add the `X-MS-Structure: true` header to any GET request to a MoonShine page.

### Basic Request

```
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
```

### With Authentication (JWT)

```
GET /admin/resource/users/crud HTTP/1.1
X-MS-Structure: true
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## JSON Response Structure

Every SDUI response is a nested JSON object. Each node in the tree represents a UI component with the following keys:

| Key | Type | Description |
|---|---|---|
| `type` | `string` | Component type name (e.g., "Dashboard", "Card", "Heading", "Text", "TableBuilder") |
| `components` | `array` | Array of child component objects (recursive structure) |
| `states` | `object` | Component state data -- titles, content, field values, configuration flags |
| `attributes` | `object` | HTML attributes -- CSS classes, IDs, data attributes, ARIA attributes |

### Full Example Response

Request:

```
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
```

Response:

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

### Component Types

MoonShine maps its Blade components to SDUI type names. Common types include:

| SDUI Type | MoonShine Component | Typical States |
|---|---|---|
| `Dashboard` | Dashboard page | `title` |
| `Card` | Card component | `title`, `subtitle` |
| `Heading` | Heading component | `level`, `content` |
| `Text` | Text/paragraph | `content` |
| `TableBuilder` | TableBuilder | `columns`, `rows`, `paginator` |
| `FormBuilder` | FormBuilder | `action`, `method`, `fields` |
| `Modal` | Modal component | `title`, `isOpen` |
| `OffCanvas` | OffCanvas component | `title`, `position` |
| `ActionButton` | ActionButton | `label`, `url`, `method` |
| `Fragment` | Fragment component | `name`, `asyncUrl` |
| `Tabs` | Tabs component | `activeTab` |

### Attributes Object

The `attributes` object contains HTML attributes as key-value pairs:

- `class` -- Array of CSS class strings.
- `id` -- Element ID string.
- `data-*` -- Custom data attributes.
- `style` -- Inline style string (when applicable).
- `aria-*` -- Accessibility attributes.

---

## Customization Headers

MoonShine provides additional headers to control what is included in the SDUI response.

### X-MS-Structure

**Required.** Enables SDUI mode.

```
X-MS-Structure: true
```

Without this header, MoonShine returns standard HTML responses.

---

### X-MS-Without-States

Omits all `states` objects from the response. Useful when you only need the component tree structure without data.

**Request:**

```
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
X-MS-Without-States: true
```

**Response:**

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

**Use cases:**

- Discovering the component tree structure for a page.
- Building a client-side component registry.
- Reducing response payload when state data is not needed.

---

### X-MS-Only-Layout

Returns only the layout structure (header, sidebar, footer, navigation) without the page content. Useful for building the application shell.

**Request:**

```
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
X-MS-Only-Layout: true
```

**Response (example):**

```json
{
  "type": "Layout",
  "components": [
    {
      "type": "Header",
      "components": [
        {
          "type": "Logo",
          "states": {
            "src": "/images/logo.svg",
            "alt": "MoonShine Admin"
          },
          "attributes": {
            "class": ["h-8"]
          }
        },
        {
          "type": "Navigation",
          "states": {
            "items": [
              {"label": "Dashboard", "url": "/admin", "icon": "home"},
              {"label": "Users", "url": "/admin/resource/users", "icon": "users"},
              {"label": "Posts", "url": "/admin/resource/posts", "icon": "document"}
            ]
          },
          "attributes": {
            "class": ["flex", "gap-4"]
          }
        }
      ],
      "states": {},
      "attributes": {
        "class": ["bg-white", "shadow"]
      }
    },
    {
      "type": "Sidebar",
      "components": [],
      "states": {
        "collapsed": false
      },
      "attributes": {
        "class": ["w-64", "bg-gray-50"]
      }
    }
  ],
  "states": {},
  "attributes": {
    "class": ["min-h-screen"]
  }
}
```

**Use cases:**

- Fetching the navigation structure once on app startup.
- Building a persistent shell layout.
- Rendering sidebar/header independently from page content.

---

### X-MS-Without-Layout

Returns the page content without the layout wrapper (no header, sidebar, footer). Useful for loading page content into an existing shell.

**Request:**

```
GET /admin/dashboard HTTP/1.1
X-MS-Structure: true
X-MS-Without-Layout: true
```

**Response (example):**

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
            "class": ["text-2xl", "font-bold"]
          }
        }
      ],
      "states": {
        "title": "Dashboard Overview"
      },
      "attributes": {
        "class": ["bg-white", "shadow", "rounded-lg"]
      }
    }
  ],
  "states": {
    "title": "Admin Dashboard"
  },
  "attributes": {
    "class": ["py-6"]
  }
}
```

**Use cases:**

- Loading page content into a pre-existing layout shell.
- Navigating between pages without re-fetching the layout.
- Building SPA-like experiences where only content changes.

---

### Combining Headers

Headers can be combined for specific use cases:

```
# Structure without states and without layout
X-MS-Structure: true
X-MS-Without-States: true
X-MS-Without-Layout: true
```

This returns only the page content component tree structure without state data or layout wrappers -- the lightest possible response for understanding page composition.

> **Note:** `X-MS-Only-Layout` and `X-MS-Without-Layout` are mutually exclusive. If both are sent, behavior is undefined; use only one at a time.

---

## Headers Summary

| Header | Value | Effect |
|---|---|---|
| `X-MS-Structure` | `true` | Enable SDUI mode (required) |
| `X-MS-Without-States` | `true` | Remove all `states` objects from response |
| `X-MS-Only-Layout` | `true` | Return only layout components (header, sidebar, nav) |
| `X-MS-Without-Layout` | `true` | Return only page content (no layout wrapper) |

---

## Client Implementation Notes

### Rendering Strategy

1. **Fetch layout once** using `X-MS-Only-Layout: true` on app startup.
2. **Fetch page content** using `X-MS-Without-Layout: true` on navigation.
3. **Map component types** to native views (iOS UIKit/SwiftUI, Android Compose, React components).
4. **Apply attributes** as styling properties (map CSS classes to native styles).
5. **Use states** to populate component data (titles, content, field values).

### Example: React SDUI Renderer

```jsx
function SDUIRenderer({ node }) {
    const Component = componentRegistry[node.type]
    if (!Component) return null

    return (
        <Component states={node.states} attributes={node.attributes}>
            {node.components?.map((child, i) => (
                <SDUIRenderer key={i} node={child} />
            ))}
        </Component>
    )
}

const componentRegistry = {
    Dashboard: DashboardComponent,
    Card: CardComponent,
    Heading: HeadingComponent,
    Text: TextComponent,
    TableBuilder: TableComponent,
    FormBuilder: FormComponent,
}
```

### Example: Swift SDUI Rendering

```swift
func renderNode(_ node: SDUINode) -> some View {
    switch node.type {
    case "Card":
        CardView(title: node.states?["title"] as? String ?? "") {
            ForEach(node.components ?? [], id: \.type) { child in
                renderNode(child)
            }
        }
    case "Heading":
        Text(node.states?["content"] as? String ?? "")
            .font(headingFont(level: node.states?["level"] as? Int ?? 1))
    case "Text":
        Text(node.states?["content"] as? String ?? "")
    default:
        EmptyView()
    }
}
```

---

## Integration with API Mode

SDUI can be combined with API mode for a complete headless MoonShine setup:

```
# SDUI structure for page rendering
GET /admin/resource/users/crud HTTP/1.1
X-MS-Structure: true
X-MS-Without-Layout: true
Authorization: Bearer <token>

# JSON data for CRUD operations
GET /admin/resource/users/crud HTTP/1.1
Accept: application/json
Authorization: Bearer <token>
```

Use SDUI for understanding the page structure and component layout, and API mode for performing CRUD operations with pure data responses.
