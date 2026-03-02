# Field Interactive Features Reference (MoonShine v4)

## Editing in Preview Mode

Available for: `Text`, `Number`, `Checkbox`, `Select`, `Date`.

### updateOnPreview

Edit a field inline in tables/previews. Value is saved on change.

```php
use MoonShine\UI\Fields\Text;

Text::make('Name')->updateOnPreview()
```

Full signature:

```php
updateOnPreview(
    ?Closure $url = null,
    ?ResourceContract $resource = null,
    mixed $condition = null,
    array $events = []
)
```

With custom URL (outside a resource):

```php
Text::make('Name')
    ->updateOnPreview(url: fn() => '/my/custom/endpoint')
```

### withUpdateRow

Like `updateOnPreview()` but refreshes the entire table row without page reload.

```php
Text::make('Name')
    ->withUpdateRow('index-table-post-resource')
```

Combined with custom URL:

```php
Text::make('Name')
    ->updateOnPreview(url: fn() => '/my/url')
    ->withUpdateRow('index-table-post-resource')
```

### updateInPopover

Like `withUpdateRow()` but values appear in a popup editor.

```php
Text::make('Name')
    ->updateInPopover('index-table-post-resource')
```

---

## Reactivity

Make fields react to each other in real time within a `FormBuilder`.

```php
reactive(
    ?Closure $callback = null,
    bool $lazy = false,
    int $debounce = 0,
    int $throttle = 0,
    bool|Closure $silent = false,
    bool|Closure $silentSelf = false,
)
```

Parameters:
- `$callback` - function that receives Fields collection and current value.
- `$lazy` - delayed call.
- `$debounce` - delay between calls (ms).
- `$throttle` - throttle interval (ms).
- `$silent` - skip ALL field re-rendering on change.
- `$silentSelf` - skip only THIS field's re-rendering (keeps focus).

### Auto-generate slug from title

```php
use MoonShine\UI\Collections\Fields;
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Fields\Text;

FormBuilder::make()
    ->name('my-form')
    ->fields([
        Text::make('Title')
            ->reactive(function(Fields $fields, ?string $value): Fields {
                return tap($fields, static fn ($fields) => $fields
                    ->findByColumn('slug')
                    ?->setValue(str($value ?? '')->slug()->value())
                );
            }),

        Text::make('Slug')
            ->reactive()
    ])
```

### Cascading selects

```php
use MoonShine\UI\Fields\Field;
use MoonShine\UI\Fields\Select;
use MoonShine\UI\Collections\Fields;

Select::make('Category', 'category_id')
    ->reactive(function(Fields $fields, ?string $value, Field $field, array $values): Fields {
        $field->setValue($value);

        return tap($fields, static fn ($fields) =>
            $fields
                ->findByColumn('article_id')
                ?->options(
                    Article::where('category_id', $value)
                        ->get()
                        ->pluck('title', 'id')
                        ->toArray()
                )
        );
    })
```

### silent / silentSelf

```php
Select::make('Select 1')->reactive(silentSelf: true)  // skip self re-rendering (keep focus)
Select::make('Select 2')->reactive(silent: true)       // skip all re-rendering

// Conditional via closure
Select::make('Select 1')->reactive(
    silentSelf: fn(string $value, array $values, Select $ctx): bool => $value === 'Option 2'
)
```

---

## Dynamic Display (showWhen)

Show/hide fields based on other field values in real time (no server requests).

### showWhen

```php
use MoonShine\UI\Fields\Text;

// Show when category_id equals 1 (default operator is '=')
Text::make('Name')
    ->showWhen('category_id', 1)

// With explicit operator
Text::make('Name')
    ->showWhen('category_id', '!=', 5)

// With 'in' operator
Text::make('Name')
    ->showWhen('category_id', 'in', [1, 2, 3])

// With 'not in' operator
Text::make('Name')
    ->showWhen('status', 'not in', ['archived', 'deleted'])

// Comparison operators
Text::make('Discount')
    ->showWhen('price', '>', 100)

Text::make('Premium')
    ->showWhen('price', '>=', 1000)
```

**Supported operators:** `=`, `!=`, `>`, `<`, `>=`, `<=`, `in`, `not in`

Hidden fields get `display: none` and `name` removed. To keep name: `protected bool $submitShowWhen = true;` in your resource.

### showWhenDate

For date fields (proper date comparison). Default operator is `=`.

```php
Text::make('Content')->showWhenDate('created_at', '>', '2024-09-15 10:00')
Text::make('Content')->showWhenDate('event_date', '2024-12-25')
```

### showWhenRow (Experimental)

For fields inside `Json` table mode - condition based on same-row values.

```php
use MoonShine\UI\Fields\Json;
use MoonShine\UI\Fields\Select;
use MoonShine\UI\Fields\Text;

Json::make('Options')->fields([
    Select::make('Type')->options([
        1 => 'First',
        2 => 'Second',
    ]),
    Text::make('Value')->showWhenRow('type', 1),
])
```

### Nested fields

Use dot notation for nested field references.

```php
Text::make('Parts')
    ->showWhen('attributes.1.size', '!=', 2)
```

With nested Json:

```php
Json::make('Attributes', 'attributes')->fields([
    Text::make('Size'),
    Text::make('Parts')
        ->showWhen('category_id', 3),
    Json::make('Settings', 'settings')->fields([
        Text::make('Width')
            ->showWhen('category_id', 3),
        Text::make('Height'),
    ])
])
```

### Multiple conditions (AND logic)

```php
use MoonShine\Laravel\Fields\Relationships\BelongsTo;

BelongsTo::make('Category', 'category', resource: CategoryResource::class)
    ->showWhenDate('created_at', '>', '2024-08-05 10:00')
    ->showWhenDate('created_at', '<', '2024-08-05 19:00')
```

---

## Json Field

`MoonShine\UI\Fields\Json` - work with JSON data. Renders as a table of rows by default.

### fields - array of objects

```php
use MoonShine\UI\Fields\Json;
use MoonShine\UI\Fields\Position;
use MoonShine\UI\Fields\Switcher;
use MoonShine\UI\Fields\Text;

Json::make('Product Options', 'options')
    ->fields([
        Position::make(),
        Text::make('Title'),
        Text::make('Value'),
        Switcher::make('Active'),
    ])
```

### keyValue - key/value pairs

```php
Json::make('Data')
    ->keyValue()

// Custom labels and field types
Json::make('Label', 'data')
    ->keyValue(
        keyField: Select::make('Key')
            ->options(['vk' => 'VK', 'email' => 'E-mail']),
        valueField: Text::make('Value'),
    )
```

### onlyValue - flat value array

```php
Json::make('Data')
    ->onlyValue()
```

### object - single object (not array)

```php
Json::make('Product Options', 'options')
    ->fields([
        Text::make('Title'),
        Switcher::make('Active'),
    ])
    ->object()
```

### Nested Json

```php
Json::make('Products', 'products')
    ->fields([
        Text::make('Name', 'name'),
        Json::make('Prices', 'prices')
            ->fields([
                Number::make('Wholesale price', 'wholesale_price'),
                Number::make('Retail price', 'retail_price'),
            ])
            ->object(),
    ])
```

### Default value

```php
Json::make('Data')
    ->keyValue('Key', 'Value')
    ->default([
        ['key' => 'Default key', 'value' => 'Default value']
    ])

Json::make('Values')
    ->onlyValue()
    ->default([
        ['value' => 'Default value']
    ])
```

### stopFilteringEmpty

Disable automatic removal of empty values.

```php
Json::make('data')->stopFilteringEmpty()
```

### creatable / removable

```php
Json::make('Data')
    ->keyValue()
    ->creatable(limit: 6)
    ->removable()
```

Custom add button:

```php
use MoonShine\UI\Components\ActionButton;

Json::make('Data')
    ->keyValue()
    ->creatable(
        button: ActionButton::make('New')->primary()
    )
```

Remove button attributes:

```php
Json::make('Data', 'data.content')
    ->fields([
        Text::make('Title'),
        Image::make('Image'),
        Text::make('Value'),
    ])
    ->removable(attributes: ['@click.prevent' => 'customAsyncRemove'])
    ->creatable()
```

### vertical

```php
Json::make('Data')->vertical()
```

### reorderable (drag-and-drop)

Enabled by default. Disable with:

```php
Json::make('Data')->reorderable(false)
```

### filterMode (for use in filters)

```php
Json::make('Data')
    ->fields([
        Text::make('Title', 'title'),
        Text::make('Value', 'value')
    ])
    ->filterMode()
```

### buttons

Override row action buttons.

```php
use MoonShine\UI\Components\ActionButton;

Json::make('Data', 'data.content')
    ->fields([
        Text::make('Title'),
        Image::make('Image'),
        Text::make('Value'),
    ])
    ->buttons([
        ActionButton::make('')
            ->icon('trash')
            ->onClick(fn() => 'remove()', 'prevent')
            ->secondary()
            ->showInLine()
    ])
```

### Modifiers

```php
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Components\Table\TableBuilder;
use MoonShine\UI\Fields\Json;

// Modify remove button
Json::make('Data')
    ->modifyRemoveButton(
        fn(ActionButton $button) => $button->customAttributes([
            'class' => 'btn-secondary'
        ])
    )

// Modify create button
Json::make('Data')
    ->creatable()
    ->modifyCreateButton(
        fn(ActionButton $button) => $button->customAttributes([
            'class' => 'btn-primary'
        ])
    )

// Modify the underlying table
Json::make('Data')
    ->modifyTable(
        fn(TableBuilder $table, bool $preview) => $table->customAttributes([
            'style' => 'width: 50%;'
        ])
    )
```

---

## Fieldset

`MoonShine\UI\Fields\Fieldset` - group fields visually. Renders as HTML `<fieldset>` in forms.

```php
use MoonShine\UI\Fields\Fieldset;
use MoonShine\Laravel\Fields\Slug;
use MoonShine\UI\Fields\Text;

Fieldset::make('Title', [
    Text::make('Title'),
    Slug::make('Slug'),
])

// Or via fluent method
Fieldset::make()
    ->fields([
        Text::make('Title'),
        Slug::make('Slug'),
    ])
```

### Edit view with LineBreak

```php
use MoonShine\UI\Components\Layout\LineBreak;

Fieldset::make('Title', [
    Text::make('Title'),
    LineBreak::make(),
    Slug::make('Slug'),
])
```

### Conditional display

```php
Fieldset::make('Stack', fn(Fieldset $ctx) => $ctx->getData()?->getOriginal()->id === 3 ? [
        Date::make('Creation date', 'created_at'),
    ] : [
        Date::make('Creation date', 'created_at'),
        LineBreak::make(),
        Email::make('Email', 'email'),
    ]
)
```

---

## Template

`MoonShine\UI\Fields\Template` - construct a field using fluent interface (no built-in implementation).

```php
use MoonShine\UI\Fields\Template;
use MoonShine\UI\Fields\Text;

Template::make()
    ->setLabel('My Field')
    ->fields([
        Text::make('Title')
    ])
```

---

## HiddenIds

`MoonShine\UI\Fields\HiddenIds` - pass primary keys of selected elements for bulk actions.

```php
use MoonShine\UI\Fields\HiddenIds;

HiddenIds::make('index-table')
```

Full usage with bulk action:

```php
use MoonShine\UI\Components\FlexibleRender;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Fields\HiddenIds;
use MoonShine\UI\Components\FormBuilder;

ActionButton::make('Active', route('moonshine.posts.mass-active', $this->uriKey()))
    ->inModal(
        'Active',
        fn (): string => (string) FormBuilder::make(
            route('moonshine.posts.mass-active', $this->uriKey()),
            fields: [
                HiddenIds::make($this->listComponentName()),
                FlexibleRender::make(__('moonshine::ui.confirm_message')),
            ]
        )
        ->async()
        ->submit('Active', ['class' => 'btn-secondary'])
    )
    ->bulk()
```

---

## Assets

Add CSS/JS assets to fields (fields are components). Three approaches:

```php
// On the fly
Text::make('Name')->addAssets([new Css(Vite::asset('resources/css/text-field.css'))])

// In custom field class via assets() method
protected function assets(): array
{
    return [Js::make('/js/custom.js'), Css::make('/css/styles.css')];
}

// In custom field class via booted() method
protected function booted(): void
{
    parent::booted();
    $this->getAssetManager()
        ->add(Css::make('/css/app.css'))
        ->append(Js::make('/js/app.js'));
}
```

---

## Macroable Trait

All fields support `macro()` and `mixin()` from Laravel's `Macroable` trait.

```php
use MoonShine\UI\Fields\Field;

Field::macro('myMethod', fn() => $this->customAttributes(['data-custom' => true]))
Text::make('Title')->myMethod()

Field::mixin(new MyNewMethods())
```
