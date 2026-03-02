# Layout Components — Full API

## Grid

**Namespace:** `MoonShine\UI\Components\Layout\Grid`

12-column grid system for page layout.

```php
Grid::make(
    iterable $components = [],
    int $gap = 6,
)
```

```php
use MoonShine\UI\Components\Layout\Grid;
use MoonShine\UI\Components\Layout\Column;

Grid::make([
    Column::make([Box::make('Left')], colSpan: 6, adaptiveColSpan: 12),
    Column::make([Box::make('Right')], colSpan: 6, adaptiveColSpan: 12),
])
```

---

## Column

**Namespace:** `MoonShine\UI\Components\Layout\Column`

```php
Column::make(
    iterable $components = [],
    int $colSpan = 12,
    int $adaptiveColSpan = 12,
)
```

- `colSpan` -- columns occupied at >= 1280px (out of 12).
- `adaptiveColSpan` -- columns occupied at < 1280px.

```php
Column::make([Text::make('Name')], colSpan: 4, adaptiveColSpan: 12)
```

---

## Box

**Namespace:** `MoonShine\UI\Components\Layout\Box`

Container that highlights content areas.

```php
Box::make(
    Closure|string|iterable $labelOrComponents = [],
    iterable $components = []
)
```

```php
use MoonShine\UI\Components\Layout\Box;

// Without heading
Box::make([Alert::make()->content('Text')])

// With heading
Box::make('Title box', ['Hello!'])

// Dark style
Box::make(['Hello!'])->dark()

// With icon
Box::make('Title box', ['Hello!'])->icon('users')
```

---

## Div

**Namespace:** `MoonShine\UI\Components\Layout\Div`

Simple `<div>` wrapper.

```php
Div::make(iterable $components)
```

```php
use MoonShine\UI\Components\Layout\Div;

Div::make([Text::make('Field 1')])->class('my-class')
```

---

## Flex

**Namespace:** `MoonShine\UI\Components\Layout\Flex`

Flexbox container for inline positioning.

```php
Flex::make(
    iterable $components = [],
    int $colSpan = 12,
    int $adaptiveColSpan = 12,
    string $itemsAlign = 'center',
    string $justifyAlign = 'center',
    bool $withoutSpace = false,
)
```

```php
use MoonShine\UI\Components\Layout\Flex;

Flex::make([
    Text::make('Field 1'),
    Text::make('Field 2'),
])
    ->justifyAlign('between')  // Tailwind justify-between
    ->itemsAlign('start')      // Tailwind items-start
    ->wrap()                   // flex-wrap (default)
    ->unwrap()                 // disable wrapping
    ->withoutSpace             // remove margins
```

---

## Wrapper

**Namespace:** `MoonShine\UI\Components\Layout\Wrapper`

Layout wrapper ensuring correct sidebar + content display. Used after `Body`.

```php
use MoonShine\UI\Components\Layout\{Body, Wrapper};

Body::make([
    Wrapper::make([
        // sidebar + content
    ])
])
```

---

## Fragment

**Namespace:** `MoonShine\Crud\Components\Fragment`

Wraps a page area for partial async updates via `@fragment` Blade directive.

```php
use MoonShine\Crud\Components\Fragment;

Fragment::make(iterable $components = [])
```

### Core Methods

```php
// Name (required for events)
Fragment::make($components)->name('fragment-name')

// Update via event
FormBuilder::make()
    ->async(events: AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'fragment-name'))

// Pass additional params with update request
Fragment::make($components)
    ->name('fragment-name')
    ->updateWith(
        params: ['resourceItem' => request('resourceItem')],
        events: [AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'next-fragment')],
        callback: AsyncCallback::with(afterResponse: 'myFunction'),
    )

// Pass field values via selectors
Fragment::make($components)
    ->withSelectorsParams(['start_date' => '#start_date', 'end_date' => '#end_date'])
    ->name('fragment-name')

// Include current URL query params
Fragment::make($components)->name('fg')->withQueryParams()

// Auto-update at interval (ms)
Fragment::make([
    ValueMetric::make('Metric')->value(random_int(1, 1000)),
])
    ->name('fragment-metric')
    ->autoUpdate(5000)
```

### Sequential Fragment Updates

```php
Fragment::make([FlexibleRender::make('<p>Step 1: ' . time() . '</p>')])
    ->updateWith(events: [AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'fg-step-2')])
    ->name('fg-step-1'),

Fragment::make([FlexibleRender::make('<p>Step 2: ' . time() . '</p>')])
    ->name('fg-step-2'),

ActionButton::make('Start')
    ->dispatchEvent([AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'fg-step-1')]),
```

---

## Tabs

**Namespace:** `MoonShine\UI\Components\Tabs`, `MoonShine\UI\Components\Tabs\Tab`

```php
use MoonShine\UI\Components\Tabs;
use MoonShine\UI\Components\Tabs\Tab;

Tabs::make([
    Tab::make('Tab 1', [Text::make('Text 1')]),
    Tab::make('Tab 2', [Text::make('Text 2')])->active(),
])
```

### Active Tab

```php
Tab::make('Tab 1', $components)->active()
Tab::make('Tab 1', $components)->active(session()->has('key'))
```

### Vertical Mode

```php
Tabs::make($tabs)->vertical()
```

### Activation via Event

```php
Tab::make('Tab 1', $components)->name('first-tab'),
Tab::make('Tab 2', $components)->name('second-tab'),

ActionButton::make('Go to Tab 2')
    ->dispatchEvent(AlpineJs::event(JsEvent::TAB_ACTIVE, 'second-tab'))
```

### Label Attributes

```php
Tab::make($components)->labelAttributes(['x-show' => '!flag'])
```

---

## Collapse

**Namespace:** `MoonShine\UI\Components\Collapse`

Collapsible content block.

```php
Collapse::make(
    Closure|string $label = '',
    iterable $components = [],
    bool $open = false,
    bool $persist = true,
)
```

```php
use MoonShine\UI\Components\Collapse;

Collapse::make('Title/Slug', [
    Text::make('Title'),
    Text::make('Slug'),
])
    ->open()           // expanded by default
    ->persist(false)   // do not remember state
    ->icon('cog')
```

---

## Card

**Namespace:** `MoonShine\UI\Components\Card`

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
use MoonShine\UI\Components\Card;

Card::make(
    title: 'Article Title',
    thumbnail: '/images/image_1.jpg',
    url: fn() => '/articles/1',
    values: ['ID' => 1, 'Author' => 'John'],
    subtitle: date('d.m.Y'),
)
    ->header(fn() => Badge::make('new', 'success'))
    ->actions(fn() => ActionButton::make('Edit', '/edit'))
```

### Key Methods

```php
Card::make('Title')
    ->subtitle(fn() => 'Subtitle')
    ->url(fn() => '/articles/1')
    ->thumbnail(['/img/1.jpg', '/img/2.jpg'])  // carousel of images
    ->values(['Key' => 'Value'])
    ->header(fn() => 'Header HTML')
    ->actions(fn() => ActionButton::make('Edit', '/edit'))
```

---

## CardsBuilder

**Namespace:** `MoonShine\UI\Components\CardsBuilder`

Displays a list of items as cards.

```php
CardsBuilder::make(iterable $items = [], FieldsContract|iterable $fields = [])
```

```php
use MoonShine\UI\Components\CardsBuilder;

CardsBuilder::make()
    ->fields([ID::make(), Text::make('title')])
    ->items([['id' => 1, 'title' => 'Title 1'], ['id' => 2, 'title' => 'Title 2']])
    ->title('title')
    ->subtitle(fn() => 'Subtitle')
    ->thumbnail('image')
    ->overlay()
    ->header(fn() => Badge::make('new', 'success'))
    ->content('Custom content')
    ->buttons([
        ActionButton::make('Edit', route('name.edit')),
    ])
    ->columnSpan(4, adaptiveColumnSpan: 12)
    ->cast(new ModelCaster(Article::class))
    ->paginator(
        (new ModelCaster(Article::class))->paginatorCast(Article::query()->paginate())
    )
    ->customComponent(fn(Article $article, int $index, CardsBuilder $builder) =>
        Badge::make($article->title, 'green')
    )
```

### Async Mode

```php
CardsBuilder::make()
    ->items(Article::paginate())
    ->fields([ID::make(), Text::make('Title')])
    ->name('my-cards')
    ->async(events: [AlpineJs::event(JsEvent::CARDS_UPDATED, 'my-cards')])
```

---

## Heading

**Namespace:** `MoonShine\UI\Components\Heading`

```php
Heading::make(
    Closure|string $label = '',
    ?int $h = null,
    bool $asClass = true,
)
```

```php
use MoonShine\UI\Components\Heading;

Heading::make('Title', 2)          // <div class="h2">
Heading::make('Title')->h(1)       // <div class="h1">
Heading::make('Title')->h(1, false) // <h1>
Heading::make('Title')->tag('p')->h(2) // <p class="h2">
```

---

## Title

**Namespace:** `MoonShine\UI\Components\Title`

Page-level heading component.

```php
Title::make(Closure|string|null $value, int $h = 1)
```

```php
Title::make('Hello world')
```

---

## Link

**Namespace:** `MoonShine\UI\Components\Link`

```php
Link::make(Closure|string $href, Closure|string $label = '')
```

```php
use MoonShine\UI\Components\Link;

Link::make('https://moonshine-laravel.com', 'MoonShine')
    ->icon('arrow-top-right-on-square')
    ->filled()
    ->tooltip('Visit site')
```

---

## Snippet

**Namespace:** `MoonShine\UI\Components\Snippet`

Displays text as code with copy-to-clipboard button.

```php
use MoonShine\UI\Components\Snippet;
use MoonShine\Support\Enums\Color;

Snippet::make('composer require moonshine/moonshine', Color::PRIMARY)
```

Colors: `primary`, `secondary`, `success`, `info`, `warning`, `error`, `purple`, `pink`, `blue`, `green`, `yellow`, `red`, `gray`.

---

## HTML (Layout)

**Namespace:** `MoonShine\UI\Components\Layout\Html`

Foundation `<html>` tag wrapper (includes `<!DOCTYPE html>`).

```php
use MoonShine\UI\Components\Layout\Html;

Html::make([Head::make([]), Body::make([])])
```

---

## FlexibleRender

**Namespace:** `MoonShine\UI\Components\FlexibleRender`

Renders text, HTML, or Blade views via closure.

```php
use MoonShine\UI\Components\FlexibleRender;

FlexibleRender::make('HTML content')
FlexibleRender::make(view('path_to_blade'))
FlexibleRender::make(view('path_to_blade', ['key' => 'val']))
FlexibleRender::make(fn($data) => view('path', $data), fn() => ['key' => 'val'])
```

---

## LineBreak

**Namespace:** `MoonShine\UI\Components\Layout\LineBreak`

Adds vertical spacing between elements.

```php
LineBreak::make()
```

---

## Divider

**Namespace:** `MoonShine\UI\Components\Layout\Divider`

Horizontal separator, optionally with label.

```php
use MoonShine\UI\Components\Layout\Divider;

Divider::make()
Divider::make('Section Title')->centered()
```

---

## Content

**Namespace:** `MoonShine\UI\Components\Layout\Content`

Main content area wrapper in layouts.

```php
use MoonShine\UI\Components\Layout\Content;

Content::make([
    Title::make($this->getPage()->getTitle()),
    Components::make($this->getPage()->getComponents()),
])
```
