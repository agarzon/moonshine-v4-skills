# Display & Decoration Components Reference (v4)

## Fragment

`MoonShine\Crud\Components\Fragment` — wraps page areas for isolated async updates via events. Uses Laravel Blade `@fragment` directive.

```php
use MoonShine\Crud\Components\Fragment;
use MoonShine\UI\Fields\Text;

Fragment::make([
    Text::make('Name', 'first_name')
])->name('fragment-name')
```

### Update via events

```php
// Trigger fragment update from a form
FormBuilder::make()
    ->async(events: AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'fragment-name'))

// Pass parameters with the request
Fragment::make($components)
    ->name('fragment-name')
    ->updateWith(params: ['resourceItem' => request('resourceItem')])
```

### Pass field values via selectors

```php
Fragment::make($components)
    ->withSelectorsParams([
        'start_date' => '#start_date',
        'end_date' => '#end_date',
    ])
    ->name('fragment-name')
```

### Chain fragment updates sequentially

```php
Fragment::make([
    FlexibleRender::make('<p>Step 1: ' . time() . '</p>')
])
    ->updateWith(
        events: [AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'fg-step-2')],
    )
    ->name('fg-step-1'),

Fragment::make([
    FlexibleRender::make('<p>Step 2: ' . time() . '</p>')
])->name('fg-step-2'),

ActionButton::make('Start')
    ->dispatchEvent([AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'fg-step-1')])
```

### Response callback

```php
Fragment::make($components)
    ->updateWith(
        callback: AsyncCallback::with(afterResponse: 'afterResponseFunction')
    )
    ->name('fragment-name')
```

### Preserve URL query params

```php
Fragment::make($components)->name('fragment-name')->withQueryParams()
```

### Auto-update at interval

```php
Fragment::make([
    ValueMetric::make('Metric')->value(random_int(1, 1000))->columnSpan(6),
])
    ->name('fragment-metric')
    ->withQueryParams()
    ->autoUpdate(5000) // every 5 seconds
```

---

## Metrics

### ValueMetric

`MoonShine\UI\Components\Metrics\Wrapped\ValueMetric` — displays a single value (e.g., record counts).

```php
use MoonShine\UI\Components\Metrics\Wrapped\ValueMetric;

ValueMetric::make('Completed orders')
    ->value(fn(): int => Order::completed()->count())
```

**Progress indicator:**

```php
ValueMetric::make('Open tasks')
    ->value(fn(): int => Task::opened()->count())
    ->progress(fn(): int => Task::count())
```

**Value formatting:**

```php
ValueMetric::make('Profit')
    ->value(fn(): int => Order::completed()->sum('price'))
    ->valueFormat(fn(int $value): string => Number::forHumans($value))
```

**Icon and column span:**

```php
ValueMetric::make('Orders')
    ->value(fn(): int => Order::count())
    ->icon('shopping-bag')
    ->columnSpan(4, 12) // desktop: 4/12, mobile: 12/12
```

### LineChartMetric / DonutChartMetric

Require `moonshine/apexcharts` package (based on ApexCharts):

```shell
composer require moonshine/apexcharts
```

See the moonshine/apexcharts repository for API details.

---

## Badge

`MoonShine\UI\Components\Badge` — styled badge/label display.

```php
use MoonShine\Support\Enums\Color;
use MoonShine\UI\Components\Badge;

Badge::make('Active', Color::SUCCESS, 'check')
Badge::make('Pending', Color::WARNING, 'clock')
Badge::make('Users', Color::PURPLE, 'users')
```

**Available colors (Color enum + string):** PRIMARY, SECONDARY, SUCCESS, INFO, WARNING, ERROR, and strings: `'purple'`, `'pink'`, `'blue'`, `'green'`, `'yellow'`, `'red'`, `'gray'`.

**Blade:**

```blade
<x-moonshine::badge color="success" :icon="'check'">Active</x-moonshine::badge>
```

---

## Carousel

`MoonShine\UI\Components\Carousel` — image carousel.

```php
use MoonShine\UI\Components\Carousel;

Carousel::make(
    items: ['/images/image_1.jpg', '/images/image_2.jpg'],
    portrait: false,
    alt: 'Gallery images'
)

// Or fluent
Carousel::make(alt: 'Some alt')
    ->items(['/images/image_1.jpg', '/images/image_2.jpg'])
```

**Blade:**

```blade
<x-moonshine::carousel :items="['/images/image_1.jpg']" :alt="'Alt text'" />
```

---

## Rating

`MoonShine\UI\Components\Rating` — styled rating display.

```php
use MoonShine\UI\Components\Rating;

Rating::make(value: 3, min: 1, max: 5)
Rating::make(value: 8, min: 1, max: 10)
```

**Blade:**

```blade
<x-moonshine::rating value="8" min="1" max="10" />
```

---

## Thumbnails

`MoonShine\UI\Components\Thumbnails` — thumbnail image display.

```php
use MoonShine\UI\Components\Thumbnails;

Thumbnails::make(['/images/image_1.jpg', '/images/image_2.jpg'])
Thumbnails::make('/images/single.jpg') // single thumbnail
```

**Blade:**

```blade
<x-moonshine::thumbnails :items="['/images/image_1.jpg']" alt="Description" />
```

---

## Collapse

`MoonShine\UI\Components\Collapse` — collapsible content wrapper with state persistence.

```php
use MoonShine\UI\Components\Collapse;

Collapse::make('Title/Slug', [
    Text::make('Title'),
    Text::make('Slug'),
])
```

**Constructor parameters:**
- `$label` — title
- `$components` — components inside
- `$open` — expanded by default (default: `false`)
- `$persist` — remember state (default: `true`)

**Methods:**

```php
Collapse::make('Settings', $components)
    ->open()           // start expanded
    ->persist(false)   // don't remember state
    ->icon('cog')      // add icon
```

**Blade:**

```blade
<x-moonshine::collapse :label="'Title/Slug'" :components='$components' />
```

---

## Divider

`MoonShine\UI\Components\Layout\Divider` — horizontal separator.

```php
use MoonShine\UI\Components\Layout\Divider;

Divider::make()                    // simple line
Divider::make('Section')           // with label
Divider::make('Section')->centered() // centered label
```

**Blade:**

```blade
<x-moonshine::layout.divider />
```

---

## FlexibleRender

`MoonShine\UI\Components\FlexibleRender` — render text, HTML, or Blade views.

```php
use MoonShine\UI\Components\FlexibleRender;

FlexibleRender::make('Plain HTML')
FlexibleRender::make(view('path_to_blade'))
FlexibleRender::make(view('path_to_blade', ['data' => 'value']))
FlexibleRender::make(
    fn($data) => view('path_to_blade', $data),
    fn() => ['data' => 'value']
)
```

---

## When

`MoonShine\UI\Components\When` — conditional component display.

```php
use MoonShine\UI\Components\When;

When::make(
    condition: static fn() => config('moonshine.auth.enabled', true),
    components: static fn() => [Profile::make()],
    default: static fn() => [Text::make('Auth disabled')], // optional fallback
)
```

---

## Title

`MoonShine\UI\Components\Layout\Title` — page heading.

```php
use MoonShine\UI\Components\Layout\Title;

Title::make('Page Title')
Title::make('Sub heading', h: 2)
Title::make(fn() => 'Dynamic Title')
```
