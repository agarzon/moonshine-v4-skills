# Selection Fields Reference (MoonShine v4)

## Select

`MoonShine\UI\Fields\Select` - Dropdown selection powered by Tom Select library.

### Basic Usage

```php
use MoonShine\UI\Fields\Select;

Select::make('Country', 'country_id')
    ->options([
        'value 1' => 'Option Label 1',
        'value 2' => 'Option Label 2',
    ])
```

### Options via DTO Objects

```php
use MoonShine\Support\DTOs\Select\Option;
use MoonShine\Support\DTOs\Select\OptionProperty;
use MoonShine\Support\DTOs\Select\Options;

Select::make('Select')
    ->options(
        new Options([
            new Option(
                label: 'Option 1',
                value: '1',
                selected: true,
                properties: new OptionProperty(image: 'https://example.com/img.png'),
            ),
            new Option(
                label: 'Option 2',
                value: '2',
                properties: new OptionProperty(image: 'https://example.com/img2.png'),
            ),
        ])
    )
```

### Static Make for Option / OptionGroup

```php
use MoonShine\Support\DTOs\Select\Option;
use MoonShine\Support\DTOs\Select\OptionGroup;
use MoonShine\Support\DTOs\Select\Options;

Option::make('Label', 'value')

OptionGroup::make('Group', new Options([
    Option::make('Option 1', '1'),
    Option::make('Option 2', '2'),
]))
```

### Groups (OptionGroup)

```php
// Array syntax
Select::make('City', 'city_id')
    ->options([
        'Italy' => [
            1 => 'Rome',
            2 => 'Milan',
        ],
        'France' => [
            3 => 'Paris',
            4 => 'Marseille',
        ]
    ])

// OptionGroup DTO syntax
Select::make('City')
    ->options(
        new Options([
            new OptionGroup('Italy', new Options([
                new Option('Rome', '1'),
                new Option('Milan', '2'),
            ])),
            new OptionGroup('France', new Options([
                new Option('Paris', '3'),
                new Option('Marseille', '4'),
            ])),
        ])
    )
```

### Default, Nullable, Placeholder

```php
Select::make('Country', 'country_id')
    ->options([...])
    ->default('value 2')

Select::make('Country', 'country_id')
    ->options([...])
    ->nullable()

Select::make('Country', 'country')
    ->nullable()
    ->placeholder('Country')
```

### Multiple

```php
Select::make('Country', 'country_id')
    ->options([...])
    ->multiple()
```

> When using `multiple()` with a model, add a cast to the model attribute: `'field' => 'json'` or `'field' => 'array'`.

### Searchable

```php
Select::make('Country', 'country_id')
    ->options([...])
    ->searchable()
```

### Async Search

```php
Select::make('Country', 'country_id')
    ->options([...])
    ->async('/search')

// Load values on page display
Select::make('Country', 'country_id')
    ->options([...])
    ->async('/search')
    ->asyncOnInit(whenOpen: false)

// Load on click
Select::make('Country', 'country_id')
    ->async('/search')
    ->asyncOnInit(whenOpen: true)

// Show loader before opening
Select::make('Country', 'country_id')
    ->async('/search')
    ->asyncOnInit(withLoading: true)
```

The async endpoint must return JSON: `[{"value": 1, "label": "Option 1"}, ...]`

Also works with MoonShine JsonResponse for notifications/events:
```php
use MoonShine\Crud\JsonResponse;
use MoonShine\Support\DTOs\Select\Options;

public function selectOptions(): JsonResponse
{
    $options = new Options([...]);

    return JsonResponse::make()
        ->merge(['options' => $options->toArray()])
        ->toast('Loaded', ToastType::SUCCESS);
}
```

### Change Events

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

Select::make('Country', 'country_id')
    ->options([...])
    ->onChangeEvent(
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'selects'),
        exclude: ['text', 'description'],  // exclude fields from payload
        // withoutPayload: true,           // or exclude all payload
    )
```

### optionProperties / OptionImage

Add images to select options:

```php
// Via array
Select::make('Country', 'country_id')
    ->options([1 => 'Andorra', 2 => 'UAE'])
    ->optionProperties(fn() => [
        1 => ['image' => 'https://example.com/ad.png'],
        2 => ['image' => 'https://example.com/ae.png'],
    ])

// Via Options DTO (see above)

// Customized OptionImage
use MoonShine\Support\DTOs\Select\OptionImage;
use MoonShine\Support\Enums\ObjectFit;

new OptionProperty(
    new OptionImage(
        src: 'https://example.com/img.png',
        height: 6,    // h-{x} class, 1-10
        width: 6,     // w-{x} class, 1-10
        objectFit: ObjectFit::CONTAIN
    )
)
```

### Disabled Options

```php
use MoonShine\Support\DTOs\Select\Option;
use MoonShine\Support\DTOs\Select\Options;

Select::make('Country')
    ->options(
        new Options([
            Option::make('Active', 'active'),
            Option::make('Disabled', 'disabled')->disabled(),
            Option::make('Conditional', 'conditional')->disabled(fn() => true),
        ])
    )

// Custom attributes on options
Option::make('Option 1', '1')->customAttributes(['data-info' => 'value'])
OptionGroup::make('Italy', new Options([...]))->customAttributes(['data-country' => 'it'])
```

### Native Mode

```php
Select::make('Type')->native()
```

### Tom Select Plugins

```php
Select::make('Type')
    ->addPlugin(['plugin_1', 'plugin_2'])

Select::make('Type')
    ->addPlugin('plugin_1', ['foo' => 'bar'])
    ->addPlugin('plugin_2', ['foo' => 'bar'])
```

Custom JS plugin:
```js
document.addEventListener('moonshine:select_init', function({ detail: { createPlugin } }) {
    createPlugin('myPlugin', function(pluginOptions) {
        console.log(pluginOptions, this.getValue())
        this.on('change', value => { /* ... */ })
    })
})
```

### Settings (Tom Select)

```php
use MoonShine\Support\DTOs\Select\Settings;

Select::make('Type')
    ->settings(
        Settings::make()
            ->maxOptions(10)
            ->highlight(false)
    );
```

### fieldsNames

```php
use MoonShine\Support\DTOs\Select\FieldsNames;

Select::make('Type')
    ->fieldsNames(
        FieldsNames::make()
            ->value('id')
            ->label('name')
            ->children('children')
    );
```

### asyncSettings

```php
use MoonShine\Support\DTOs\Select\AsyncSettings;

Select::make('Type')
    ->asyncSettings(
        AsyncSettings::make()
            ->queryKey('q')
            ->selectedValuesKey('name')
            ->resultKey('data')
            ->withAllFields()
    );

// Shorthand for withAllFields
Select::make('Type')->asyncWithFields();
```

### selectCreatable

Enable creating new options inline:

```php
Select::make('Type')
    ->selectCreatable(
        filterRegex: '/^\d+$/',
        persist: true,
        createOnBlur: false,
        duplicates: false,
        addPrecedence: false,
    );
```

### selectMaxItems

```php
Select::make('Type')
    ->selectMaxItems(5, 'Max 5 items allowed');
```

### Editing in Preview

```php
Select::make('Country')->updateOnPreview()
```

---

## Enum

`MoonShine\UI\Fields\Enum` - Select from PHP Enum. Inherits from Select.

```php
use MoonShine\UI\Fields\Enum;

Enum::make('Status')
    ->attach(StatusEnum::class)
```

> Model attributes require Enum Cast.

### toString Method

```php
enum StatusEnum: string
{
    case NEW = 'new';
    case DRAFT = 'draft';
    case PUBLIC = 'public';

    public function toString(): ?string
    {
        return match ($this) {
            self::NEW => 'New',
            self::DRAFT => 'Draft',
            self::PUBLIC => 'Public',
        };
    }
}
```

### getColor Method

Displays as colored icon in preview. 13 available colors: primary, secondary, success, warning, error, info, purple, pink, blue, green, yellow, red, gray.

```php
public function getColor(): ?string
{
    return match ($this) {
        self::NEW => 'info',
        self::DRAFT => 'gray',
        self::PUBLIC => 'success',
    };
}
```

### getIcon Method

Displays icon next to value in preview. Requires `getColor()` to also be implemented.

```php
public function getIcon(): ?string
{
    return match ($this) {
        self::NEW => 'sparkles',
        self::DRAFT => 'pencil',
        self::PUBLIC => 'check-circle',
    };
}
```

---

## Checkbox

`MoonShine\UI\Fields\Checkbox` - Yes/no checkbox field.

```php
use MoonShine\UI\Fields\Checkbox;

Checkbox::make('Publish', 'is_publish')
```

### On/Off Values

Default: `1` (on) and `0` (off).

```php
Checkbox::make('Publish', 'is_publish')
    ->onValue('yes')
    ->offValue('no')
```

Supports editing in preview mode.

---

## Switcher

`MoonShine\UI\Fields\Switcher` - Toggle switch. Inherits from Checkbox with different visual design.

```php
use MoonShine\UI\Fields\Switcher;

Switcher::make('Publish', 'is_publish')
```

Has all the same capabilities as Checkbox (onValue, offValue, preview editing).

---

## Preview

`MoonShine\UI\Fields\Preview` - Read-only display field. NOT for data input.

```php
use MoonShine\UI\Fields\Preview;

Preview::make('Preview', 'preview', static fn() => fake()->realText())
```

### Badge

```php
Preview::make('Status')
    ->badge(fn($status, Field $field) => $status === 1 ? 'green' : 'gray')
```

Available colors: primary, secondary, success, warning, error, info, purple, pink, blue, green, yellow, red, gray.

### Boolean

```php
Preview::make('Active')
    ->boolean(hideTrue: false, hideFalse: false)
```

### Link

```php
Preview::make('Link')
    ->link('https://moonshine-laravel.com', blank: false)

Preview::make('Link')
    ->link(fn($link, Field $field) => $link, fn($name, Field $field) => 'Go')
```

Parameters: `$link`, `$name`, `$icon`, `$withoutIcon`, `$blank`.

### Image

```php
Preview::make('Thumb')->image()
```
