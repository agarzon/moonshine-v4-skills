# Overlay Components — Full API

## Modal

**Namespace:** `MoonShine\UI\Components\Modal`

### make()

```php
Modal::make(
    Closure|string $title = '',
    Closure|Renderable|string $content = '',
    Closure|Renderable|ActionButtonContract|string $outer = '',
    Closure|string|null $asyncUrl = null,
    iterable $components = [],
)
```

- `$title` -- modal title.
- `$content` -- modal body content.
- `$outer` -- external trigger block.
- `$asyncUrl` -- URL for async content loading.
- `$components` -- components for the modal.

```php
use MoonShine\UI\Components\Modal;

Modal::make(
    title: 'Confirm',
    content: 'Are you sure?',
)
```

### name() and Events

```php
Modal::make('Title', 'Content')->name('my-modal')

// Toggle via ActionButton
ActionButton::make('Show Modal')->toggleModal('my-modal')

// Toggle via async ActionButton
ActionButton::make('Show Modal', '/endpoint')
    ->async(events: [AlpineJs::event(JsEvent::MODAL_TOGGLED, 'my-modal')])

// JS: document.dispatchEvent(new CustomEvent('modal_toggled:my-modal'))
// Alpine: $dispatch('modal_toggled:my-modal')
// Global: MoonShine.ui.toggleModal('my-modal')
```

### toggleEvents()

Fire events when modal opens/closes.

```php
Modal::make('Title', asyncUrl: '/')
    ->name('my-modal')
    ->toggleEvents(
        [AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Hello'])],
        onlyOpening: false,
        onlyClosing: true,
    )
```

### open()

Open modal on page load.

```php
Modal::make('Title', 'Content')->open()
Modal::make('Title', 'Content')->open(fn() => $shouldOpen)
```

### closeOutside()

Control whether clicking outside closes the modal (default: true).

```php
Modal::make('Title', 'Content')->closeOutside(false)
```

### autoClose()

Control auto-closing after successful async request (default: true).

```php
Modal::make(
    'Form Modal',
    fn() => FormBuilder::make(route('endpoint'))
        ->fields([Text::make('Text')])
        ->async(),
)->name('demo-modal')->autoClose(false)
```

### Width: wide() / full() / auto()

```php
Modal::make('Title', 'Content')->wide()   // maximum width
Modal::make('Title', 'Content')->full()   // full screen width
Modal::make('Title', 'Content')->auto()   // width based on content
```

### Async Content

```php
Modal::make('Title', '', ActionButton::make('Show', '#'), asyncUrl: '/endpoint')

// Always reload on open (default: loads once)
Modal::make('Title', '', asyncUrl: '/endpoint')->alwaysLoad()
```

### outerAttributes()

```php
Modal::make('Title', 'Content', ActionButton::make('Show', '#'))
    ->outerAttributes(['class' => 'mt-2'])
```

---

## OffCanvas

**Namespace:** `MoonShine\UI\Components\OffCanvas`

Side panel component.

### make()

```php
OffCanvas::make(
    Closure|string $title = '',
    Closure|Renderable|string $content = '',
    Closure|string $toggler = '',
    Closure|string|null $asyncUrl = null,
    iterable $components = [],
)
```

```php
use MoonShine\UI\Components\OffCanvas;

OffCanvas::make(
    'Confirm',
    fn() => FormBuilder::make(route('password.confirm'))
        ->async()
        ->fields([Password::make('Password')->eye()])
        ->submit('Confirm'),
    'Show Panel'
)
```

### name() and Events

```php
OffCanvas::make('Title', 'Content')->name('my-canvas')

// Toggle via ActionButton
ActionButton::make('Show Panel')->toggleOffCanvas('my-canvas')

// Toggle via async event
ActionButton::make('Show', '/endpoint')
    ->async(events: [AlpineJs::event(JsEvent::OFF_CANVAS_TOGGLED, 'my-canvas')])

// JS: MoonShine.ui.toggleOffCanvas('my-canvas')
```

### toggleEvents()

```php
OffCanvas::make('Title', asyncUrl: '/')
    ->name('my-off-canvas')
    ->toggleEvents(
        [AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Hello'])],
        onlyOpening: false,
        onlyClosing: true,
    )
```

### open()

```php
OffCanvas::make('Title', 'Content', 'Show')->open()
```

### Position: left()

Default position is right. Use `left()` for left side.

```php
OffCanvas::make('Title', 'Content', 'Show')->left()
```

### Width: wide() / full()

```php
OffCanvas::make('Title', 'Content', 'Show')->wide()
OffCanvas::make('Title', 'Content', 'Show')->full()
```

### Async Content

```php
OffCanvas::make('Title', '', 'Show', asyncUrl: '/endpoint')

// Always reload on each open
OffCanvas::make('Title', '', 'Show', asyncUrl: '/endpoint')->alwaysLoad()
```

### autoClose()

```php
OffCanvas::make(
    'Form Panel',
    fn() => FormBuilder::make(route('endpoint'))
        ->fields([Text::make('Text')])
        ->async(),
    'Show'
)->autoClose(false)
```

### togglerAttributes()

```php
OffCanvas::make('Title', 'Content', 'Show')
    ->togglerAttributes(['class' => 'mt-2'])
```

---

## Popover

**Namespace:** `MoonShine\UI\Components\Popover`

Tooltip/popover shown on hover.

```php
Popover::make(
    string $title,
    string $trigger,
    string $placement = 'right',
)
```

- `$title` -- popover heading.
- `$trigger` -- text/HTML that triggers the popover on hover.
- `$placement` -- position relative to trigger (see tippy.js placement options).

```php
use MoonShine\UI\Components\Popover;

Popover::make('Popover Title', 'Hover me')
    ->content('HTML content inside popover')
```

Placements: `top`, `bottom`, `left`, `right`, `top-start`, `top-end`, `bottom-start`, `bottom-end`, etc.

---

## Dropdown

**Namespace:** `MoonShine\UI\Components\Dropdown`

Dropdown block with optional search, items list, title, and footer.

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
use MoonShine\UI\Components\Dropdown;

Dropdown::make(
    'Menu',
    'Click me',
    'Custom HTML content',
    ['Item 1', 'Item 2']
)

// Searchable dropdown
Dropdown::make(
    title: 'Options',
    toggler: 'Select',
    items: ['Option A', 'Option B', 'Option C'],
    searchable: true,
    searchPlaceholder: 'Search...',
    placement: 'bottom-start',
    footer: 'Dropdown footer',
)
```

---

## Alert

**Namespace:** `MoonShine\UI\Components\Alert`

Page notification/alert component.

```php
Alert::make(
    string $type = '',
    string $icon = '',
    bool $removable = false,
)
```

### Types

```php
use MoonShine\UI\Components\Alert;

Alert::make(type: 'primary')->content('Primary message')
Alert::make(type: 'secondary')->content('Secondary message')
Alert::make(type: 'success')->content('Success message')
Alert::make(type: 'warning')->content('Warning message')
Alert::make(type: 'error')->content('Error message')
Alert::make(type: 'info')->content('Info message')
```

### Icon

```php
Alert::make(icon: 'academic-cap')->content('Custom icon alert')
```

### Removable

Auto-removes after a timeout.

```php
Alert::make(removable: true)->content('This will disappear')
```

---

## Flash

**Namespace:** `MoonShine\UI\Components\Layout\Flash`

Displays session flash messages and toast notifications.

```php
Flash::make(
    string $key = 'alert',
    string|FlashType $type = FlashType::INFO,
    bool $withToast = true,
    bool $removable = true,
)
```

```php
use MoonShine\UI\Components\Layout\Flash;

Flash::make()
```

### Toast via Session

```php
session()->flash('toast', [
    'type' => FlashType::INFO->value,
    'message' => 'Info',
]);
```

### Toast via JS Event

```php
use MoonShine\Support\EventParams\ToastEventParams;
use MoonShine\Support\Enums\ToastType;

AlpineJs::event(
    JsEvent::TOAST,
    params: ToastEventParams::make(ToastType::SUCCESS, 'Success')
)
```

---

## Notifications

**Namespace:** `MoonShine\Crud\Components\Layout\Notifications`

Dropdown element in layout for displaying system notifications.

```php
Notifications::make()
```

Typically placed in `Header` or `Sidebar`.

---

## Spinner

**Namespace:** `MoonShine\UI\Components\Spinner`

Loading indicator.

```php
Spinner::make(
    string $size = 'sm',
    string|Color $color = '',
    bool $fixed = false,
    bool $absolute = false,
)
```

```php
use MoonShine\UI\Components\Spinner;

Spinner::make()
Spinner::make(size: 'lg', color: 'primary')
Spinner::make(fixed: true)     // fixed position
Spinner::make(absolute: true)  // absolute position
```

Sizes: `sm`, `md`, `lg`, `xl`.
Colors: `primary`, `secondary`, `success`, `warning`, `error`, `info`.

---

## Loader

**Namespace:** `MoonShine\UI\Components\Loader`

Styled loading indicator (different visual from Spinner).

```php
use MoonShine\UI\Components\Loader;

Loader::make()

// Change the global view
Loader::changeView('my-custom-view-path')
```

---

## When

**Namespace:** `MoonShine\UI\Components\When`

Conditional rendering of components.

```php
When::make(
    Closure $condition,
    Closure $components,
    ?Closure $default = null,
)
```

```php
use MoonShine\UI\Components\When;

When::make(
    static fn() => config('moonshine.auth.enabled', true),
    static fn() => [Profile::make()],
    static fn() => [FlexibleRender::make('Auth disabled')],
)
```

Example in layout:

```php
Sidebar::make([
    Menu::make(),
    When::make(
        fn() => config('moonshine.auth.enabled', true),
        fn() => [Profile::make()],
    ),
])
```
