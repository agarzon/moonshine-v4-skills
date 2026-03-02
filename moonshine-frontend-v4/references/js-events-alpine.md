# JS Events, Alpine.js & AsyncCallback -- Reference (v4)

This reference covers the complete JavaScript events system, Alpine.js integration, the global MoonShine class, and the AsyncCallback system in MoonShine v4.

---

## JsEvent Enum -- Complete Listing

All events are defined in `MoonShine\Support\Enums\JsEvent`. Each event follows the pattern `event_name:component_name`.

```php
use MoonShine\Support\Enums\JsEvent;
```

| JsEvent Constant | JS Event String | Description |
|---|---|---|
| `JsEvent::FRAGMENT_UPDATED` | `fragment_updated:{name}` | Refresh a Fragment area |
| `JsEvent::TABLE_UPDATED` | `table_updated:{name}` | Refresh an async TableBuilder |
| `JsEvent::TABLE_REINDEX` | `table_reindex:{name}` | Reindex table fields after sorting |
| `JsEvent::TABLE_ROW_UPDATED` | `table_row_updated:{name}-{row-id}` | Refresh a single table row |
| `JsEvent::TABLE_ROW_ADDED` | `table_row_added:{name}` | Append new cloned row (NEW in v4) |
| `JsEvent::TABLE_EMPTY_ROW_ADDED` | `table_empty_row_added:{name}` | Append/prepend new empty row (NEW in v4) |
| `JsEvent::CARDS_UPDATED` | `cards_updated:{name}` | Refresh a CardsBuilder |
| `JsEvent::FORM_RESET` | `form_reset:{name}` | Reset form fields to initial values |
| `JsEvent::FORM_SUBMIT` | `form_submit:{name}` | Trigger form submission |
| `JsEvent::MODAL_TOGGLED` | `modal_toggled:{name}` | Open/close a Modal component |
| `JsEvent::OFF_CANVAS_TOGGLED` | `off_canvas_toggled:{name}` | Open/close an OffCanvas component |
| `JsEvent::POPOVER_TOGGLED` | `popover_toggled:{name}` | Open/close a Popover component |
| `JsEvent::TOAST` | `toast:{name}` | Trigger a Toast notification |
| `JsEvent::SHOW_WHEN_REFRESH` | `show_when_refresh:{name}` | Re-evaluate showWhen field conditions |
| `JsEvent::TAB_ACTIVE` | `tab_active:{name}` | Activate a specific tab (NEW in v4) |

---

## Dispatching Events from PHP

### AlpineJs::event()

Builds event strings for use in PHP (async event arrays, dispatchEvent calls, etc.).

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

// Signature
AlpineJs::event(
    string|JsEvent $event,
    ?string $name = null,
    array $params = [],
)
```

**Examples:**

```php
// Simple event -- returns "table_updated:index"
AlpineJs::event(JsEvent::TABLE_UPDATED, 'index')

// With parameters -- returns "table_updated:index|var=foo"
AlpineJs::event(JsEvent::TABLE_UPDATED, 'index', ['var' => 'foo'])

// Modal toggle -- returns "modal_toggled:my-modal"
AlpineJs::event(JsEvent::MODAL_TOGGLED, 'my-modal')

// Fragment refresh -- returns "fragment_updated:data-fragment"
AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'data-fragment')

// Tab activation -- returns "tab_active:settings-tab"
AlpineJs::event(JsEvent::TAB_ACTIVE, 'settings-tab')
```

### AlpineJs::eventBlade()

Builds Blade attribute strings for use in `x-on` directives and `customAttributes()`.

```php
// Signature
AlpineJs::eventBlade(
    string|JsEvent $event,
    ?string $name = null,
    ?string $call = null,
    array $params = []
)
```

**Examples:**

```php
// Returns "@form-reset-main-form.window"
AlpineJs::eventBlade(JsEvent::FORM_RESET, 'main-form')

// Usage in customAttributes
FormBuilder::make('/crud/update')
    ->name('main-form')
    ->customAttributes([
        // @form-reset-main-form.window="formReset"
        AlpineJs::eventBlade(JsEvent::FORM_RESET, 'main-form') => 'formReset',
    ])
```

### Common PHP Usage Patterns

**Async form that triggers table refresh:**

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Fields\Text;

FormBuilder::make('/crud/update')
    ->name('main-form')
    ->async(events: [
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'crud-table'),
        AlpineJs::event(JsEvent::FORM_RESET, 'main-form'),
    ])
    ->fields([Text::make('Title')])
```

**ActionButton that opens a modal:**

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Components\Modal;
use MoonShine\UI\Components\ActionButton;

Modal::make('Title', 'Content')
    ->name('my-modal'),

ActionButton::make('Show modal window', '/endpoint')
    ->async(
        events: [AlpineJs::event(JsEvent::MODAL_TOGGLED, 'my-modal')]
    )
```

**Fragment-based partial update:**

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Components\Fragment;

Fragment::make([
    TableBuilder::make()->fields($fields)->items($items)->name('data-table')
])->name('data-fragment'),

ActionButton::make('Reload')
    ->dispatchEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'data-fragment'))
```

---

## Dispatching Events from JavaScript

### Native DOM Events

```js
// Fire event when DOM is ready
document.addEventListener("DOMContentLoaded", () => {
    dispatchEvent(new CustomEvent("modal_toggled:my-modal"))
})

// Fire event with parameters
dispatchEvent(new CustomEvent("table_updated:index", {
    detail: { var: 'foo' }
}))

// Fire event on a specific element
document.querySelector('#my-element')
    .dispatchEvent(new CustomEvent("form_submit:my-form"))
```

### Alpine.js $dispatch

Within Alpine.js components, use the `$dispatch` magic method:

```js
// Inside an Alpine.js component
this.$dispatch('modal_toggled:my-modal')
this.$dispatch('table_updated:crud-table')
this.$dispatch('form_reset:main-form')
this.$dispatch('fragment_updated:data-fragment')
this.$dispatch('tab_active:settings-tab')
```

### Listening for Events in Alpine.js

```html
<div x-data="myComponent"
     @table-updated-index.window="handleTableUpdate"
     @modal-toggled-my-modal.window="handleModalToggle"
>
    <!-- component content -->
</div>
```

---

## Response Events

Return events from controllers or resource methods so they fire after the response is processed.

```php
use MoonShine\Crud\JsonResponse;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

// Trigger table refresh after a successful action
return JsonResponse::make()
    ->events([
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'index'),
    ]);

// Trigger multiple events
return JsonResponse::make()
    ->toast('Saved!', ToastType::SUCCESS)
    ->events([
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'index'),
        AlpineJs::event(JsEvent::FORM_RESET, 'create-form'),
        AlpineJs::event(JsEvent::MODAL_TOGGLED, 'edit-modal'),
    ]);
```

> **v4 namespace:** `MoonShine\Crud\JsonResponse` (was `MoonShine\Laravel\Http\Responses\MoonShineJsonResponse`).

---

## Blade Directives

### @defineEvent

Declare an event listener directly in Blade templates.

```php
// Signature
@defineEvent(
    string|JsEvent $event,
    ?string $name = null,
    ?string $call = null,
    array $params = [],
)
```

**Parameters:**

- `$event` -- The event name (string or JsEvent enum).
- `$name` -- The component name suffix.
- `$call` -- The callback function name.
- `$params` -- Additional event parameters.

**Example:**

```blade
<div x-data="myComponent"
    {{-- Generates: @table-updated-index.window="asyncRequest" --}}
    @defineEvent('table-updated', 'index', 'asyncRequest')
>
    <!-- component content -->
</div>
```

**With parameters:**

```blade
<div x-data="myComponent"
    @defineEvent('table-updated', 'index', 'asyncRequest', ['page' => 1])
>
</div>
```

### @defineEventWhen

Conditionally add an event listener based on a boolean condition.

```php
// Signature
@defineEventWhen(
    mixed $condition,
    string|JsEvent $event,
    ?string $name = null,
    ?string $call = null,
    array $params = [],
)
```

**Parameters:**

- `$condition` -- Boolean condition; if truthy, the event listener is added.
- `$event` -- The event name.
- `$name` -- The component name suffix.
- `$call` -- The callback function name.
- `$params` -- Additional event parameters.

**Example:**

```blade
<div x-data="myComponent"
    {{-- Only adds event listener when $isAsync is true --}}
    @defineEventWhen($isAsync, 'table-updated', 'index', 'asyncRequest')
>
</div>
```

---

## Global MoonShine JS Class

The `window.MoonShine` object is available after the `moonshine:init` event fires. Wait for this event before accessing MoonShine methods.

```js
document.addEventListener("moonshine:init", () => {
    // MoonShine is now available
})
```

### MoonShine.request()

Send HTTP requests through MoonShine's request handler.

```js
// Signature
MoonShine.request(ctx, url, method = 'get', body = {}, headers = {}, data = {})
```

**Parameters:**

- `ctx` -- The Alpine.js component context (usually `this`).
- `url` -- The target URL.
- `method` -- HTTP method: `'get'`, `'post'`, `'put'`, `'delete'`.
- `body` -- Request body object.
- `headers` -- Additional request headers.
- `data` -- Extra data passed to response handlers.

**Example:**

```js
document.addEventListener("moonshine:init", () => {
    // GET request
    MoonShine.request(this, '/admin/api/data', 'get')

    // POST request with body
    MoonShine.request(this, '/admin/api/update', 'post', {
        title: 'New Title',
        status: 'active'
    })

    // Request with custom headers
    MoonShine.request(this, '/admin/api/data', 'get', {}, {
        'X-Custom-Header': 'value'
    })
})
```

### MoonShine.ui.toast()

Display a toast notification.

```js
// Signature
MoonShine.ui.toast(message, type)
```

**Types:** `'success'`, `'error'`, `'warning'`, `'info'`.

```js
MoonShine.ui.toast('Record saved successfully', 'success')
MoonShine.ui.toast('Something went wrong', 'error')
MoonShine.ui.toast('Please check the form', 'warning')
MoonShine.ui.toast('Processing started', 'info')
```

### MoonShine.ui.toggleModal()

Open or close a named Modal component.

```js
MoonShine.ui.toggleModal('my-modal')
```

### MoonShine.ui.toggleOffCanvas()

Open or close a named OffCanvas component.

```js
MoonShine.ui.toggleOffCanvas('filters-panel')
```

### MoonShine.iterable.sortable()

Initialize drag-and-drop reordering on a container.

```js
// Signature
MoonShine.iterable.sortable(container, url, group, events, attributes = { handle: '.handle' }, callback)
```

**Parameters:**

- `container` -- DOM element containing sortable items.
- `url` -- Server endpoint to persist new order.
- `group` -- Sort group name (for cross-list sorting).
- `events` -- Events to dispatch after reorder.
- `attributes` -- Configuration object; `handle` specifies the drag handle selector.
- `callback` -- Function called after each sort operation.

**Example:**

```js
const container = document.querySelector('.sortable-list')
MoonShine.iterable.sortable(
    container,
    '/admin/api/reorder',
    'items',
    ['table_updated:index'],
    { handle: '.drag-handle' },
    function(evt) {
        console.log('Item moved', evt.oldIndex, evt.newIndex)
    }
)
```

### MoonShine.iterable.reindex()

Reindex form element `name` attributes after reordering (fixes array indexes).

```js
// Signature
MoonShine.iterable.reindex(container, itemSelector = '.item')
```

**Example:**

```js
const container = document.querySelector('.repeater-fields')
MoonShine.iterable.reindex(container, '.repeater-item')
```

---

## AsyncCallback (NEW in v4)

The `AsyncCallback` DTO allows hooking into the async request lifecycle for `ActionButton`, `FormBuilder`, `TableBuilder`, and any component implementing `HasAsyncContract`.

### PHP: AsyncCallback::with()

```php
use MoonShine\Support\DTOs\AsyncCallback;

// Signature
AsyncCallback::with(
    ?string $responseHandler = null,
    ?string $beforeRequest = null,
    ?string $afterResponse = null,
)
```

**Parameters:**

- `responseHandler` -- JS function name that completely overrides default response handling. When set, you are responsible for event calls, error handling, and notifications.
- `beforeRequest` -- JS function name called before the HTTP request is sent. Use for validation, loading states, or confirmation dialogs.
- `afterResponse` -- JS function name called after successful default processing. Not called if `responseHandler` is set.

### Usage with ActionButton

```php
use MoonShine\Support\DTOs\AsyncCallback;
use MoonShine\UI\Components\ActionButton;

// Override response handling entirely
ActionButton::make('Custom Action')
    ->method('myMethod', callback: AsyncCallback::with(
        responseHandler: 'handleMyResponse'
    ))

// Hook before request
ActionButton::make('Confirmed Action')
    ->method('myMethod', callback: AsyncCallback::with(
        beforeRequest: 'confirmAction'
    ))

// Hook after successful response
ActionButton::make('Track Action')
    ->method('myMethod', callback: AsyncCallback::with(
        afterResponse: 'trackSuccess'
    ))

// Combine before and after hooks
ActionButton::make('Full Lifecycle')
    ->method('myMethod', callback: AsyncCallback::with(
        beforeRequest: 'showLoading',
        afterResponse: 'hideLoading',
    ))
```

### Usage with FormBuilder

```php
use MoonShine\Support\DTOs\AsyncCallback;
use MoonShine\UI\Components\FormBuilder;

FormBuilder::make('/crud/update')
    ->name('edit-form')
    ->async(callback: AsyncCallback::with(
        beforeRequest: 'validateForm',
        afterResponse: 'showSuccessMessage',
    ))
    ->fields([/* ... */])
```

### JS: MoonShine.onCallback()

Register callback functions using the global MoonShine class. Always register inside the `moonshine:init` event.

```js
document.addEventListener("moonshine:init", () => {
    MoonShine.onCallback('callbackName', callbackFunction)
})
```

### responseHandler — Full control of response (default behavior skipped)

```js
// Signature: function(response, element, events, component)
MoonShine.onCallback('handleMyResponse', function(response, element, events, component) {
    response.json().then(data => {
        data.success
            ? MoonShine.ui.toast('Custom success!', 'success')
            : MoonShine.ui.toast('Error: ' + data.message, 'error')
    })
})
```

### beforeRequest — Runs before HTTP request

```js
// Signature: function(element, component)
MoonShine.onCallback('showLoading', function(element, component) {
    element.classList.add('opacity-50')
})
```

### afterResponse — Runs after default response processing (not called if responseHandler set)

```js
// Signature: function(data, messageType, component)
document.addEventListener("moonshine:init", () => {
    MoonShine.onCallback('trackSuccess', function(data, messageType, component) {
        // data        -- parsed JSON response data
        // messageType -- ToastType string ('success', 'error', etc.)
        // component   -- the Alpine.js component instance

        console.log('Action completed:', data)
        // Perform additional logic after default handling
    })

    MoonShine.onCallback('hideLoading', function(data, messageType, component) {
        // Re-enable UI elements
        document.querySelectorAll('.opacity-50').forEach(el => {
            el.classList.remove('opacity-50')
            el.removeAttribute('disabled')
        })
    })
})
```

---

## Creating Alpine.js Components

```shell
php artisan moonshine:component MyComponent
```

Path: `/resources/views/admin/components/my-component.blade.php`.

```html
<div x-data="myComponent">
    <span x-text="count"></span>
    <button @click="increment">+1</button>
</div>
```

```js
document.addEventListener("alpine:init", () => {
    Alpine.data("myComponent", () => ({
        count: 0,
        init() {
            console.log('myComponent initialized')
        },
        increment() {
            this.count++
        },
    }))
})
```

> **Warning:** Alpine.js is already installed (`window.Alpine`). Do not include it again or call `Alpine.start()`.
