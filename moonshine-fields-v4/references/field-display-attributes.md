# Field Display and Attributes Reference (MoonShine v4)

## Label Methods

### setLabel

Change the label after creating the field instance:

```php
use MoonShine\UI\Fields\Field;
use MoonShine\UI\Fields\Slug;

Slug::make('Slug')
    ->setLabel(
        fn(Field $field) => $field->getData()?->exists
            ? 'Slug (do not change)'
            : 'Slug'
    )
```

### translatable

Translate label via Laravel localization:

```php
Text::make('ui.Title')->translatable()
Text::make('Title')->translatable('ui')
Text::make(fn() => __('Title'))
```

### insideLabel

Wrap the field in a `<label>` tag:

```php
Text::make('Name')->insideLabel()
```

### beforeLabel

Display the label after the input field:

```php
Text::make('Name')->beforeLabel()
```

---

## Hint

Add a hint with a description below the field:

```php
Number::make('Rating')
    ->hint('From 0 to 5')
    ->min(0)
    ->max(5)
    ->stars()
```

---

## Link

Add a link next to the field (e.g., instructions):

```php
link(
    string|Closure $link,
    string|Closure $name = '',
    ?string $icon = null,
    bool $withoutIcon = false,
    bool $blank = false
)
```

```php
Text::make('Link')
    ->link('https://cutcode.dev', 'CutCode', blank: true)
```

---

## Badge

Display the field in preview mode as a colored badge:

```php
badge(string|Color|Closure|null $color = null, string|Closure|null $icon = null)
```

Available colors (13): `primary`, `secondary`, `success`, `warning`, `error`, `info`, `purple`, `pink`, `blue`, `green`, `yellow`, `red`, `gray`.

```php
use MoonShine\Support\Enums\Color;

// Static color
Text::make('Title')->badge(Color::PRIMARY)

// Dynamic color
Text::make('Title')
    ->badge(fn($status, Field $field) => 'green')

// With icon
Text::make('Status')
    ->badge(Color::SUCCESS, 'check')

// Dynamic icon
Text::make('Status')
    ->badge(Color::SUCCESS, fn($value, Text $ctx) => 'users')
```

---

## Horizontal Display

Display label and field side by side:

```php
Text::make('Title')->horizontal()
```

---

## Wrapper

### withoutWrapper

Remove the wrapper div (no label, hint, link display):

```php
Text::make('Title')->withoutWrapper()
```

### wrapperClass

Add CSS classes to the wrapper:

```php
Text::make('name')->wrapperClass('my-input')
Text::make('name')->wrapperClass(['class-1', 'class-2'])
```

### removeWrapperClass

Remove wrapper class. Supports regex patterns:

```php
Text::make('name')->removeWrapperClass('my-input')
Text::make('name')->removeWrapperClass('text-(success|primary)')
```

### wrapperStyle

Add CSS styles to the wrapper:

```php
Text::make('name')->wrapperStyle('width: 100px')
Text::make('name')->wrapperStyle(['width: 100px', 'color: red'])
```

### customWrapperAttributes

Add any attributes to the field wrapper:

```php
Text::make('Name')
    ->customWrapperAttributes(['data-handle' => 'handle'])
```

### removeWrapperAttribute

Remove an attribute from the wrapper:

```php
Text::make('Name')
    ->removeWrapperAttribute('data-handle')
```

---

## Text Wrap

Control text overflow in preview mode:

```php
use MoonShine\Support\Enums\TextWrap;

// Clamp (default for Textarea)
Text::make('Field')->textWrap(TextWrap::CLAMP)

// Ellipsis (default for Text)
Text::make('Field')->textWrap(TextWrap::ELLIPSIS)

// Disable text wrapping
Text::make('Field')->withoutTextWrap()
```

---

## Sorting

Enable column sorting in tables:

```php
Text::make('Title')->sortable()

// Sort by specific database column
BelongsTo::make('Author')->sortable('author_id')

// Custom sort callback
Text::make('Title')
    ->sortable(function (Builder $query, string $column, string $direction) {
        $query->orderBy($column, $direction);
    })
```

---

## View Modes

### Default Mode

Always render as form element (`<input>`, `<select>`, etc.) regardless of context:

```php
Text::make('Title')->defaultMode()
```

### Preview Mode

Always render as preview (formatted display) regardless of context:

```php
Text::make('Title')->previewMode()
```

### Raw Mode

Always output the original unmodified value:

```php
Text::make('Title')->rawMode()
```

---

## Required

Mark as required (adds visual indicator and HTML attribute):

```php
Text::make('Title')->required()
Text::make('Title')->required(fn() => true) // conditional
```

---

## Disabled

Disable the input:

```php
Text::make('Title')->disabled()
Text::make('Title')->disabled(fn() => true)
```

---

## Readonly

Make the field read-only:

```php
Text::make('Title')->readonly()
Text::make('Title')->readonly(fn() => true)
```

---

## Custom Attributes

Add any HTML attributes to the field element:

```php
customAttributes(array $attributes, bool $override = false)
```

- `$override = false` - uses merge (existing attributes preserved)
- `$override = true` - overwrites existing attributes

```php
Password::make('Title')
    ->customAttributes(['autocomplete' => 'off'])

Textarea::make('Text')
    ->customAttributes(['rows' => 6])
```

---

## Name Attribute

### setNameAttribute

Override the auto-generated `name` attribute:

```php
Text::make('Name')
    ->setNameAttribute('custom_name')
```

### wrapName

Add a wrapper to the `name` attribute value. Useful for filters:

```php
Text::make('Name')
    ->wrapName('options')
// Result: <input name="options[name]">
```

### virtualName

Use when two fields share the same column but are conditionally displayed:

```php
// Displayed under one condition
File::make('image')
    ->virtualColumn('image_1')

// Displayed under another condition
File::make('image')
    ->virtualColumn('image_2')
```

Then handle both in `onApply()` as needed.

---

## Complete Display Example

```php
use MoonShine\Support\Enums\Color;
use MoonShine\UI\Fields\Text;
use MoonShine\UI\Fields\Number;

Text::make('Title')
    ->setLabel(fn(Field $f) => $f->getData()?->exists ? 'Edit Title' : 'New Title')
    ->hint('Enter the article title')
    ->link('https://example.com/guidelines', 'Guidelines', blank: true)
    ->badge(Color::PRIMARY)
    ->horizontal()
    ->sortable()
    ->required()
    ->customAttributes(['maxlength' => '255'])
    ->customWrapperAttributes(['class' => 'mt-4'])

Number::make('Sort Order', 'sort')
    ->default(0)
    ->min(0)
    ->max(1000)
    ->step(1)
    ->buttons()
    ->hint('Lower numbers appear first')
    ->sortable()
```
