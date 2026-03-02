# Forms, Fields, and Tables Recipes (MoonShine v4)

## Form with Events (Table Refresh + Reset)

On successful form submission, refresh the table and reset the form fields.

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Components\Table\TableBuilder;
use MoonShine\UI\Fields\{ID, Text, Textarea};

FormBuilder::make(route('form-table.store'))
    ->fields([
        Text::make('Title')
    ])
    ->name('main-form')
    ->async(events: [
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'main-table'),
        AlpineJs::event(JsEvent::FORM_RESET, 'main-form')
    ]),

TableBuilder::make()
    ->fields([
        ID::make(),
        Text::make('Title'),
        Textarea::make('Body'),
    ])
    ->name('main-table')
    ->async()
```

### Custom Events with Blade

```blade
<div x-data=""
     @form_updated:my-event.window="alert()"
>
</div>
```

Using the `defineEvent` directive (equivalent):

```blade
<div
  x-data=""
  @defineEvent('form_updated', 'my-event', 'alert()')
>
</div>
```

### Custom event with Alpine.js component

```blade
<div x-data="my"
     @defineEvent('form_updated', 'my-event', 'asyncRequest')
>
</div>

<script>
    document.addEventListener("alpine:init", () => {
        Alpine.data("my", () => ({
            init() {},
            asyncRequest() {
                this.$event.preventDefault()
                // this.$el
                // this.$root
            }
        }))
    })
</script>
```

Trigger the custom event from FormBuilder:

```php
FormBuilder::make(route('form-table.store'))
    ->fields([
        Text::make('Title')
    ])
    ->name('main-form')
    ->async(events: ['form_updated:my-event'])
```

---

## Change Field Logic (Storing Images in Related Table)

Block `onApply()` and move logic to `onAfterApply()` to access the parent model on creation.

```php
use MoonShine\UI\Fields\Image;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Storage;

Image::make('Images', 'images')
    ->multiple()
    ->removable()
    ->changeFill(function (Model $data, Image $field) {
        // return $data->images->pluck('file');
        // or raw query
        return DB::table('images')->pluck('file');
    })
    ->onApply(function (Model $data): Model {
        // block default onApply
        return $data;
    })
    ->onAfterApply(function (Model $data, false|array $values, Image $field) {
        // $field->getRemainingValues() = values remaining after deletions
        // $field->toValue() = current images
        // $field->toValue()->diff($field->getRemainingValues()) = deleted images

        if ($values !== false) {
            foreach ($values as $value) {
                DB::table('images')->insert([
                    'file' => $field->getApplyClass()->store($field, $value),
                ]);
            }
        }

        foreach ($field->toValue()->diff($field->getRemainingValues()) as $removed) {
            DB::table('images')->where('file', $removed)->delete();
            Storage::disk('public')->delete($removed);
        }

        // or $field->removeExcludedFiles();

        return $data;
    })
    ->onAfterDestroy(function (Model $data, mixed $values, Image $field) {
        foreach ($values as $value) {
            Storage::disk('public')->delete($value);
        }

        return $data;
    })
```

---

## Fields Group via Template

Group fields under common logic (e.g., saving to a JSON column or sending a request) using the Template field.

```php
use MoonShine\UI\Fields\Template;
use MoonShine\UI\Fields\Text;
use MoonShine\UI\Components\FieldsGroup;

Template::make('Config', 'config')->fields([
    Text::make('Language'),
    Text::make('Cache'),
])
    ->changeFill(fn(mixed $data) => data_get($data, 'config'))
    ->changeRender(fn(mixed $value, Template $ctx) => FieldsGroup::make($ctx->getPreparedFields())->fill($value))
    ->onApply(function(mixed $item, mixed $value) {
        $item->config = $value;

        return $item;
    })
```

---

## Mass Edit (Bulk Records)

Add a bulk button on the index page to edit headers of all selected entries via a modal.

```php
use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;
use MoonShine\Crud\JsonResponse;
use MoonShine\Support\Attributes\AsyncMethod;
use MoonShine\Support\Enums\ToastType;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Fields\{HiddenIds, Text};

#[AsyncMethod]
public function massEdit(CrudRequestContract $request): JsonResponse
{
    Post::query()
        ->whereIn('id', $request->array('ids'))
        ->update([
            'name' => $request->input('name')
        ]);

    return JsonResponse::make()->toast('Success', ToastType::SUCCESS);
}

protected function buttons(): ListOf
{
    return parent::buttons()->add(
        ActionButton::make()
            ->bulk()
            ->icon('pencil')
            ->inModal(
                'Mass edit',
                fn() => FormBuilder::make()
                    ->name('mass-edit')
                    ->fields([
                        HiddenIds::make($this->getListComponentName()),
                        Text::make('Name')->required(),
                    ])
                    ->asyncMethod('massEdit', events: [
                        AlpineJs::event(
                            JsEvent::TABLE_UPDATED,
                            $this->getListComponentName()
                        )
                    ])
                    ->submit('Save'),
            ),
    );
}
```

---

## Saving to Config File

Using the Template field to save field values to a PHP config file.

```php
use MoonShine\UI\Fields\Template;
use MoonShine\UI\Fields\Text;
use MoonShine\UI\Components\FieldsGroup;

Template::make('Config', 'config')->fields([
    Text::make('Var'),
    Text::make('Bar'),
])
    ->changeFill(fn(mixed $data) => config('test'))
    ->changeRender(fn(mixed $value, Template $ctx) => FieldsGroup::make($ctx->getPreparedFields())->fill($value))
    ->onApply(function(mixed $item, mixed $value) {
        $content = str_replace(['array (', ')'], ['[', ']'], var_export($value, true));

        file_put_contents(config_path('test.php'), "<\?php \n\nreturn $content;");

        return $item;
    })
```

The resulting config file (`config/test.php`):

```php
return [
    'var' => 'foo',
    'bar' => 'test',
];
```

---

## Select Patterns

### Async Select via asyncMethod

Use `asyncMethod` to avoid creating a separate controller:

```php
use MoonShine\Crud\JsonResponse;
use MoonShine\Support\Attributes\AsyncMethod;

protected function formFields(): iterable
{
    return [
        Select::make('Select')->async(
            $this->getAsyncMethodUrl('selectOptions'),
        )->asyncOnInit(),
    ];
}

#[AsyncMethod]
public function selectOptions(): JsonResponse
{
    $options = new Options([
        new Option(
            label: 'Option 1',
            value: '1',
            selected: true,
            properties: new OptionProperty(image: 'https://cutcode.dev/images/platforms/youtube.png')
        ),
        new Option(
            label: 'Option 2',
            value: '2',
            properties: new OptionProperty(image: 'https://cutcode.dev/images/platforms/youtube.png')
        ),
    ]);

    return JsonResponse::make(data: $options->toArray());
}
```

### Reactive Selects

```php
Select::make('Company', 'company')->options([
    1 => 'Laravel',
    2 => 'CutCode',
    3 => 'Symfony',
])->reactive(function (FieldsContract $fields, mixed $value, Select $ctx, array $values): FieldsContract {
    $fields->findByColumn('dynamic_value')?->options((int) $value === 1 ? [
        4 => 4,
    ] : [2 => 2]);

    return $fields;
}),

Select::make('Dynamic value', 'dynamic_value')->options([4 => 4])->reactive(),
```

### ShowWhen with Duplicate Selects

Since `ShowWhen` removes hidden elements from the DOM, duplicate Select fields with different options:

```php
Select::make('Company', 'company')->options([
    1 => 'Laravel',
    2 => 'CutCode',
    3 => 'Symfony',
]),

Select::make('Dynamic value', 'dynamic_value')
    ->setNameAttribute('dynamic_value_1')
    ->showWhen('company', '1')
    ->options([1 => 1, 2 => 2]),

Select::make('Dynamic value', 'dynamic_value')
    ->setNameAttribute('dynamic_value_2')
    ->showWhen('company', '2')
    ->options([3 => 3, 4 => 4]),
```

### onChangeMethod (Return HTML for Next Select)

```php
use MoonShine\Crud\JsonResponse;
use MoonShine\UI\Components\Layout\Div;

public function selectValues(): JsonResponse
{
    $options = new Options([
        new Option('Option 1', '1', false, new OptionProperty(image: 'https://cutcode.dev/images/platforms/youtube.png')),
        new Option('Option 2', '2', true, new OptionProperty(image: 'https://cutcode.dev/images/platforms/youtube.png')),
    ]);

    return JsonResponse::make()
        ->html(
            (string) Select::make('Next')->options($options)
        );
}

protected function formFields(): iterable
{
    return [
        Select::make('Select')->options([
            1 => 1,
            2 => 2,
        ])->onChangeMethod('selectValues', selector: '.next-select'),

        Div::make()->class('next-select'),
    ];
}
```

### Fragment-Based Dynamic Selects

Fragment sends form data along with the request, allowing dynamic Select sets on reload:

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Components\Fragment;

protected function formFields(): iterable
{
    $selects = [];
    $value = request()->integer('_data.first', 1);

    if ($value === 1) {
        $selects[] = Select::make('Second')->options([1 => 1, 2 => 2]);
    }

    if ($value === 2) {
        $selects[] = Select::make('Third')->options([1 => 1, 2 => 2]);
    }

    return [
        Fragment::make([
            Select::make('First')->options([
                1 => 1,
                2 => 2,
            ])->setValue($value)->onChangeEvent(
                AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'selects')
            ),

            ...$selects,
        ])->name('selects'),
    ];
}
```
