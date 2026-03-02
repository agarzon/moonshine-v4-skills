# Basic Input Fields Reference (MoonShine v4)

## Text

`MoonShine\UI\Fields\Text` - Basic text input (`<input type="text">`).

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')
Text::make('Title', 'title')
```

### Placeholder

```php
Text::make('Username', 'username')
    ->placeholder('Enter username')
```

### Mask

```php
Text::make('Phone', 'phone')
    ->mask('+7 (999) 999-99-99')
```

### Tags

Transforms text field into a tag input field.

```php
Text::make('Tags', 'tags')
    ->tags(5) // optional limit
```

### Disable Escaping

```php
Text::make('HTML Content', 'content')
    ->unescape()
```

### Extensions

```php
// Copy button
Text::make('Token', 'token')->copy()
Text::make('Token', 'token')->copy('{{value}}') // custom template

// Show/hide value
Text::make('Password', 'password')->eye()

// Lock icon
Text::make('Protected', 'protected_field')->locked()

// Prefix and suffix
Text::make('Domain', 'domain')->prefix('https://')->suffix('.com')
```

### PrettyLimit

Display field value in preview mode with limited width. Full text shown on hover.

```php
use MoonShine\Support\Enums\Color;

Text::make('Title', 'title')
    ->prettyLimit(Color::PRIMARY);

// With dynamic values
Text::make('Status', 'status')
    ->prettyLimit(
        color: fn($value, $field) => $value === 'active' ? Color::SUCCESS : Color::ERROR,
        label: fn($value, $field) => $value === 'active' ? 'Active' : 'Inactive',
        limit: 200
    );
```

### Editing in Preview

```php
Text::make('Name')
    ->updateOnPreview()
    ->locked()
```

---

## Textarea

`MoonShine\UI\Fields\Textarea` - Multi-line text (`<textarea>`).

```php
use MoonShine\UI\Fields\Textarea;

Textarea::make('Text')

// Set height via rows attribute
Textarea::make('Text')
    ->customAttributes(['rows' => 6])

// Disable HTML escaping
Textarea::make('HTML Content', 'content')
    ->unescape()
```

---

## Number

`MoonShine\UI\Fields\Number` - Numeric input (`<input type="number">`).

```php
use MoonShine\UI\Fields\Number;

Number::make('Sort')

// With default value
Number::make('Sort')->default(2)

// Placeholder
Number::make('Rating', 'rating')
    ->nullable()
    ->placeholder('Product Rating')

// +/- buttons
Number::make('Rating')->buttons()

// Min, max, step
Number::make('Price')
    ->min(0)
    ->max(100000)
    ->step(5)

// Stars display in preview
Number::make('Rating')
    ->stars()
    ->min(1)
    ->max(10)
```

### Extensions (same as Text)

```php
Number::make('Price')->copy()
Number::make('Price')->eye()
Number::make('Price')->locked()
Number::make('Price')->suffix('USD')
```

Supports editing in preview mode.

---

## Password / PasswordRepeat

`MoonShine\UI\Fields\Password` - Password input. Auto-hashes via `Hash` facade on apply.
`MoonShine\UI\Fields\PasswordRepeat` - Confirmation field (no apply, no data modification).

```php
use MoonShine\UI\Fields\Password;
use MoonShine\UI\Fields\PasswordRepeat;

Password::make('Password', 'password'),
PasswordRepeat::make('Password repeat', 'password_repeat')
```

Inherits from Text. Displays as "***" in preview mode.

---

## Email

`MoonShine\UI\Fields\Email` - Email input (`type=email`). Inherits from Text.

```php
use MoonShine\UI\Fields\Email;

Email::make('Email')
```

---

## Url

`MoonShine\UI\Fields\Url` - URL input (`type=url`). Inherits from Text.

```php
use MoonShine\UI\Fields\Url;

Url::make('Link')

// Custom title for preview link
Url::make('Link')
    ->title(fn(string $url, Url $ctx) => str($url)->limit(3))

// Open in new tab
Url::make('Link')->blank()
```

---

## Phone

`MoonShine\UI\Fields\Phone` - Phone input (`type=tel`). Inherits from Text.

```php
use MoonShine\UI\Fields\Phone;

Phone::make('Phone')
Phone::make('Phone')->mask('7 999 999-99-99')
```

---

## Hidden

`MoonShine\UI\Fields\Hidden` - Hidden input (`type=hidden`). Hidden in form, shown in preview.

```php
use MoonShine\UI\Fields\Hidden;

Hidden::make('category_id')

// Show value while keeping hidden behavior
Hidden::make('category_id')->showValue()
```

---

## ID

`MoonShine\UI\Fields\ID` - Primary key field. Inherits from Hidden. Shown only in preview mode.

```php
use MoonShine\UI\Fields\ID;

ID::make()                       // defaults: label='ID', column='id'
ID::make(column: 'primary_key')  // custom primary key name
```

---

## Slug

`MoonShine\Laravel\Fields\Slug` - Auto-generated slug. Inherits from Text. **Depends on Eloquent model.**

```php
use MoonShine\Laravel\Fields\Slug;

Slug::make('Slug')

// Generate from another field
Slug::make('Slug')->from('title')

// Custom separator (default is '-')
Slug::make('Slug')->separator('_')

// Set locale
Slug::make('Slug')->locale('ru')

// Only unique slugs
Slug::make('Slug')->unique()

// Dynamic slug (reactive)
Text::make('Title')->reactive(),
Slug::make('Slug')->from('title')->live()

// Lazy mode
Text::make('Title')->reactive(lazy: true),
Slug::make('Slug')->from('title')->live(lazy: true)
```

---

## Position

`MoonShine\UI\Fields\Position` - Row numbering for repeating elements. Inherits from Preview.

```php
use MoonShine\UI\Fields\Json;
use MoonShine\UI\Fields\Position;
use MoonShine\UI\Fields\Text;

Json::make('Product Options', 'options')
    ->fields([
        Position::make(),
        Text::make('Title'),
        Text::make('Value'),
    ])
```

---

## Color

`MoonShine\UI\Fields\Color` - Color picker.

```php
use MoonShine\UI\Fields\Color;

Color::make('Color')
```

---

## Date

`MoonShine\UI\Fields\Date` - Date/datetime input (`<input type="date">`).

```php
use MoonShine\UI\Fields\Date;

Date::make('Created at', 'created_at')

// Date and time
Date::make('Created at', 'created_at')->withTime()

// Custom preview format
Date::make('Created at', 'created_at')->format('d.m.Y')

// Custom input format (e.g. time only)
Date::make('Created at', 'created_at')
    ->setAttribute('type', 'time')
    ->inputFormat('H:i')
```

### Time-Only Field

```php
Text::make('Time')->setAttribute('type', 'time')
```

### Extensions (copy, eye, locked, suffix)

Supports editing in preview mode.

---

## DateRange

`MoonShine\UI\Fields\DateRange` - Date range with from/to values.

```php
use MoonShine\UI\Fields\DateRange;

DateRange::make('Dates')
    ->fromTo('date_from', 'date_to')

// With time
DateRange::make('Dates')
    ->fromTo('date_from', 'date_to')
    ->withTime()

// Custom format
DateRange::make('Dates')
    ->fromTo('date_from', 'date_to')
    ->format('d.m.Y')

// Min/max/step
DateRange::make('Dates')
    ->fromTo('date_from', 'date_to')
    ->min('2024-01-01')
    ->max('2024-12-31')
    ->step(5)

// Custom attributes per field
DateRange::make('Dates')
    ->fromTo('date_from', 'date_to')
    ->fromAttributes(['class' => 'bg-black'])
    ->toAttributes(['class' => 'bg-white'])

// When used in filters, fromTo is NOT used
DateRange::make('Creation date', 'created_at')
```

---

## Range

`MoonShine\UI\Fields\Range` - Numeric range with from/to values.

```php
use MoonShine\UI\Fields\Range;

Range::make('Age', 'age')
    ->fromTo('age_from', 'age_to')

// With attributes
Range::make('Age')
    ->fromTo('age_from', 'age_to')
    ->fromAttributes(['placeholder' => 'from'])
    ->toAttributes(['placeholder' => 'to'])

// Min/max/step
Range::make('Price')
    ->fromTo('price_from', 'price_to')
    ->min(0)
    ->max(10000)
    ->step(5)

// Stars display
Range::make('Rating')
    ->fromTo('rating_from', 'rating_to')
    ->stars()

// When used in filters, fromTo is NOT used
Range::make('Age', 'age')
```

---

## RangeSlider

`MoonShine\UI\Fields\RangeSlider` - Range with slider UI. Inherits from Range.

```php
use MoonShine\UI\Fields\RangeSlider;

RangeSlider::make('Age', 'age')
    ->fromTo('age_from', 'age_to')

// When used in filters
RangeSlider::make('Age', 'age')
```
