# Field Values & Lifecycle Reference (MoonShine v4)

## Default Value

```php
use MoonShine\UI\Fields\Text;

Text::make('Name')
    ->default('Default value')
```

```php
use MoonShine\UI\Fields\Enum;

Enum::make('Status')
    ->attach(StatusEnum::class)
    ->default(StatusEnum::from('B')->value)
```

## Nullable

Keep NULL as default when value is empty.

```php
use MoonShine\UI\Fields\Password;

Password::make('Title')->nullable()
```

Accepts a condition closure:

```php
Password::make('Title')->nullable(fn() => true)
```

---

## Custom View

Replace the default blade view for a field.

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')
    ->customView('fields.my-custom-input')

// With additional data
Text::make('Title')
    ->customView('fields.my-custom-input', ['extra' => 'data'])
```

## changePreview

Override preview rendering (tables, detail pages - everywhere except the form).

```php
use MoonShine\UI\Components\Thumbnails;
use MoonShine\UI\Fields\Text;

Text::make('Thumbnail')
    ->changePreview(function (?string $value, Text $field) {
        return Thumbnails::make($value);
    })
```

Select with image carousel preview:

```php
use MoonShine\UI\Components\Carousel;
use MoonShine\UI\Fields\Select;

Select::make('Links')->options([
    '/images/1.png' => 'Picture 1',
    '/images/2.png' => 'Picture 2',
])
    ->multiple()
    ->fill(['/images/1.png', '/images/2.png'])
    ->changePreview(
        fn(?array $values, Select $ctx) => Carousel::make($values)
    )
```

---

## Hook Before Render (onBeforeRender)

Access a field just before it renders.

```php
use MoonShine\UI\Fields\Text;

Text::make('Thumbnail')
    ->onBeforeRender(function(Text $ctx) {
        // modify field state right before rendering
    })
```

---

## Request Value

### onRequestValue

Redefine how a value is obtained from the Request.

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')
    ->onRequestValue(function(mixed $value, string $name, mixed $default, Text $ctx): mixed {
        return mb_strtoupper($value);
    })
```

Parameters: `$value` (default value), `$name` (request parameter name), `$default` (fallback), `$ctx` (field).

### requestValueResolver (Global/Static)

Globally override request value resolution for all fields.

```php
use MoonShine\UI\Fields\FormElement;

FormElement::requestValueResolver(function (string|int|null $index, mixed $default, $ctx): mixed {
    return request()->input($ctx->getRequestNameDot($index), $default);
});
```

---

## Before and After Rendering

Display content before or after the field output.

```php
use MoonShine\UI\Fields\Field;
use MoonShine\UI\Fields\Text;

Text::make('Title')
    ->beforeRender(function(Field $field) {
        return $field->preview(); // show preview text before the input
    })

Text::make('Title')
    ->afterRender(function(Field $field) {
        return '<small>Helper text</small>';
    })
```

---

## Conditional Methods

### canSee

Conditionally show/hide a field based on its data.

```php
use MoonShine\UI\Fields\Text;

Text::make('Name')
    ->canSee(function (Text $field) {
        return $field->toValue() !== 'hide';
    })
```

With relationship fields (receives model as first argument):

```php
use MoonShine\Laravel\Fields\Relationships\BelongsTo;

BelongsTo::make('Item', 'item', resource: ItemResource::class)
    ->canSee(function (Comment $comment, BelongsTo $field) {
        return $comment->is_published;
    })
```

### when

Fluent conditional - executes callback when condition is true.

```php
use MoonShine\UI\Fields\Field;
use MoonShine\UI\Fields\Text;

Text::make('Slug')
    ->when(
        fn() => auth()->user()->isAdmin(),
        fn(Field $field) => $field->locked()
    )
```

### unless

Opposite of `when()` - executes when condition is false.

```php
Text::make('Slug')
    ->unless(
        fn() => auth()->user()->isAdmin(),
        fn(Field $field) => $field->readonly()
    )
```

---

## Fill Methods

### fill

Fill a field with a value manually.

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')
    ->fill('Some title')
```

Signature: `fill(mixed $value = null, ?DataWrapperContract $casted = null, int $index = 0)`

### changeFill

Override the filling logic. Receives the original data object (e.g., Model).

```php
use MoonShine\UI\Fields\Select;

Select::make('Images')->options([
    '/images/1.png' => 'Picture 1',
    '/images/2.png' => 'Picture 2',
])
    ->multiple()
    ->changeFill(
        fn(Article $article, Select $ctx) => $article->images
            ->map(fn($value) => "https://cutcode.dev$value")
            ->toArray()
    )
```

### afterFill

Run logic after the field has been filled (unlike `when()` which runs before fill).

```php
use MoonShine\UI\Fields\Select;

Select::make('Links')->options([
    '/images/1.png' => 'Picture 1',
    '/images/2.png' => 'Picture 2',
])
    ->multiple()
    ->afterFill(
        function(Select $ctx) {
            if(collect($ctx->toValue())->every(fn($value) => str_contains($value, 'cutcode.dev'))) {
                return $ctx->customWrapperAttributes(['class' => 'full-url']);
            }

            return $ctx;
        }
    )
```

---

## Apply Process (Saving / Filtering)

The apply process modifies original data (Model saving, QueryBuilder filtering, etc.).

**Lifecycle order:** `onBeforeApply()` -> `onApply()` -> model `save()` -> `onAfterApply()`

### onApply

Override the main apply logic.

```php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Storage;
use MoonShine\UI\Fields\Text;

Text::make('Thumbnail by link', 'thumbnail')
    ->onApply(function(Model $item, $value, Text $field) {
        $path = 'thumbnail.jpg';

        if ($value) {
            $item->thumbnail = Storage::put($path, file_get_contents($value));
        }

        return $item;
    })
```

For filters (receives QueryBuilder):

```php
use Illuminate\Contracts\Database\Eloquent\Builder;
use MoonShine\UI\Fields\Text;

Text::make('Title')
    ->onApply(function (Builder $query, mixed $value, Text $field) {
        $query->where('title', $value);
    })
```

### onBeforeApply

Run logic before the main apply.

```php
Text::make('Title')
    ->onBeforeApply(function(Model $item, mixed $value, Text $field) {
        // pre-processing before save
        return $field;
    })
```

### onAfterApply

Run logic after save (useful for relationships needing a saved model ID).

```php
Text::make('Title')
    ->onAfterApply(function(Model $item, mixed $value, Text $field) {
        // post-processing after save
        return $field;
    })
```

### canApply

Conditionally skip the apply process entirely.

```php
Text::make('Title')
    ->canApply(fn() => false) // will not apply/save
```

### refreshAfterApply

Re-render the field after apply (enabled by default for File fields).

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')
    ->refreshAfterApply(fn(Text $ctx) => $ctx)
```

Disable for File/Image fields:

```php
use MoonShine\Laravel\Fields\Image;

Image::make('Avatar')
    ->disableRefreshAfterApply()
```

### Global Apply Logic

Create a custom apply class and bind it globally.

```shell
php artisan moonshine:apply FileModelApply
```

```php
use MoonShine\Contracts\UI\ApplyContract;
use MoonShine\Contracts\UI\FieldContract;

/**
 * @implements ApplyContract<File>
 */
final class FileModelApply implements ApplyContract
{
    /**
     * @param  File  $field
     */
    public function apply(FieldContract $field): Closure
    {
        return function (mixed $item) use ($field): mixed {
            $requestValue = $field->getRequestValue();
            $newValue = /* your logic */;
            return data_set($item, $field->getColumn(), $newValue);
        };
    }
}
```

Register in service provider:

```php
use App\MoonShine\Applies\FileModelApply;
use MoonShine\Contracts\Core\DependencyInjection\AppliesRegisterContract;
use MoonShine\Contracts\Core\DependencyInjection\CoreContract;
use MoonShine\Laravel\DependencyInjection\ConfiguratorContract;
use MoonShine\Laravel\Resources\ModelResource;
use MoonShine\UI\Fields\File;

public function boot(
    CoreContract $core,
    ConfiguratorContract $config,
    AppliesRegisterContract $applies,
): void
{
    $applies
        ->for(ModelResource::class)
        ->fields()
        ->add(File::class, FileModelApply::class);
}
```

---

## onChange Methods

### onChangeUrl

Trigger an async request when field value changes.

```php
use MoonShine\UI\Fields\Switcher;

Switcher::make('Active')
    ->onChangeUrl(fn() => '/endpoint')
```

With selector replacement (returns HTML or JSON `{html: '...'}` from endpoint):

```php
Switcher::make('Active')
    ->onChangeUrl(fn() => '/endpoint', selector: '#my-selector')
```

Full signature:

```php
onChangeUrl(
    Closure $url,
    HttpMethod $method = HttpMethod::GET,
    array $events = [],
    ?string $selector = null,
    ?AsyncCallback $callback = null
)
```

### onChangeMethod

Call a resource/page method asynchronously on field change.

```php
use MoonShine\UI\Fields\Switcher;

Switcher::make('Active')
    ->onChangeMethod('someMethod')
```

The method in your resource/page:

```php
use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;

#[AsyncMethod]
public function someMethod(CrudRequestContract $request): void
{
    // Logic triggered on field change
}
```

Full signature:

```php
onChangeMethod(
    string $method,
    array|Closure $params = [],
    ?string $message = null,
    ?string $selector = null,
    array $events = [],
    ?AsyncCallback $callback = null,
    ?PageContract $page = null,
    ?ResourceContract $resource = null
)
```

---

## changeRender

Completely replace the field render with another field or component.

```php
use MoonShine\UI\Fields\Select;
use MoonShine\UI\Fields\Text;

Select::make('Links')->options([
    '/images/1.png' => 'Picture 1',
    '/images/2.png' => 'Picture 2',
])
    ->multiple()
    ->changeRender(
        fn(?array $values, Select $ctx) => Text::make($ctx->getLabel())->fill(implode(',', $values))
    )
```

---

## Raw Value Methods

### fromRaw

Convert a raw value (e.g., from import) into a field-usable value.

```php
use MoonShine\Laravel\Fields\Relationships\BelongsTo;

BelongsTo::make('User')
    ->fromRaw(
        fn(string $name) => User::where('name', $name)->value('id')
    )
```

```php
use MoonShine\UI\Fields\Enum;

Enum::make('Status')
    ->attach(StatusEnum::class)
    ->fromRaw(fn(string $raw, Enum $ctx) => StatusEnum::tryFrom($raw))
```

### modifyRawValue

Modify the raw value output (e.g., for export).

```php
use MoonShine\Laravel\Fields\Relationships\BelongsTo;

BelongsTo::make('User')
    ->modifyRawValue(
        fn(int $rawUserId, Article $model, BelongsTo $ctx) => $model->user->name
    )
```

```php
use MoonShine\UI\Fields\Enum;

Enum::make('Status')
    ->attach(StatusEnum::class)
    ->modifyRawValue(fn(StatusEnum $raw, Order $data, Enum $ctx) => $raw->value)
```

---

## View Modes

Force a field into a specific rendering mode regardless of context.

```php
use MoonShine\UI\Fields\Text;

// Always render as a form input element
Text::make('Title')->defaultMode()

// Always render in preview mode (read-only display)
Text::make('Title')->previewMode()

// Always render the raw original value (ideal for export)
Text::make('Title')->rawMode()
```

---

## Field Life Cycle Summary

### FormBuilder cycle

1. Field declared in resource.
2. Field enters FormBuilder.
3. FormBuilder fills the field (`fill` / `changeFill` / `afterFill`).
4. FormBuilder renders the field (`onBeforeRender` / `beforeRender` / `afterRender`).
5. On submit: `onBeforeApply()` -> `onApply()` -> model `save()` -> `onAfterApply()`.

### TableBuilder cycle

1. Field declared in resource.
2. Field enters TableBuilder in preview mode.
3. TableBuilder iterates data, fills each field.
4. TableBuilder renders rows with field previews (`changePreview`).

### Export cycle

1. Field declared in resource export method.
2. Field set to raw mode.
3. Handler iterates data, fills fields, outputs raw values (`modifyRawValue`).
