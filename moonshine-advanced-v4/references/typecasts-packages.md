# TypeCasts, CrudResource, JsonResponse, Notifications, Toasts, and Package Development (MoonShine v4)

## TypeCast System

Fields work with primitive types by default. TypeCasts bridge typed data to MoonShine components.

### DataCasterContract

```php
interface DataCasterContract
{
    public function cast(mixed $data): DataWrapperContract;
    public function paginatorCast(mixed $data): ?PaginatorContract;
}
```

### DataWrapperContract

```php
interface DataWrapperContract
{
    public function getOriginal(): mixed;
    public function getKey(): int|string|null;
    public function toArray(): array;
}
```

### ModelCaster Implementation (built-in)

```php
final readonly class ModelCaster implements DataCasterContract
{
    public function __construct(
        /** @var class-string<T> $class */
        private string $class
    ) {}

    public function getClass(): string
    {
        return $this->class;
    }

    public function cast(mixed $data): ModelDataWrapper
    {
        $model = new ($this->getClass());
        return new ModelDataWrapper($model->fill($data));
    }

    public function paginatorCast(mixed $data): ?PaginatorContract
    {
        if (! $data instanceof Paginator && ! $data instanceof CursorPaginator) {
            return null;
        }

        $paginator = new PaginatorCaster(
            $data->appends(
                moonshine()->getRequest()->getExcept('page')
            )->toArray(),
            $data->items()
        );

        return $paginator->cast();
    }
}
```

### ModelDataWrapper Implementation (built-in)

```php
final readonly class ModelDataWrapper implements DataWrapperContract
{
    public function __construct(private Model $model) {}

    public function getOriginal(): Model
    {
        return $this->model;
    }

    public function getKey(): int|string|null
    {
        return $this->model->getKey();
    }

    public function toArray(): array
    {
        return $this->model->toArray();
    }
}
```

### Usage with FormBuilder / TableBuilder

```php
TableBuilder::make(items: User::paginate())
    ->fields([
        Text::make('Email'),
    ])
    ->cast(new ModelCaster(User::class));

FormBuilder::make()
    ->fields([
        Text::make('Email'),
    ])
    ->fillCast(User::query()->first(), new ModelCaster(User::class));
```

### Generate Custom TypeCast

```shell
php artisan moonshine:type-cast MyCustomCaster
```

Creates a file in `app/MoonShine/TypeCasts/`.

---

## CrudResource (Non-Eloquent Data)

`CrudResource` is located at `MoonShine\Crud\Resources\CrudResource`. It provides a basic structure for working with any data source.

### Abstract Methods

```php
namespace App\MoonShine\Resources;

use Illuminate\Contracts\Pagination\CursorPaginator;
use Illuminate\Contracts\Pagination\Paginator;
use Illuminate\Support\Collection;
use Illuminate\Support\LazyCollection;
use MoonShine\Contracts\Core\DependencyInjection\FieldsContract;
use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;
use MoonShine\Crud\Resources\CrudResource;

final class RestCrudResource extends CrudResource
{
    public function findItem(bool $orFail = false): ?DataWrapperContract
    {
        // Find a single item by ID
    }

    public function getItems(): iterable|Collection|LazyCollection|CursorPaginator|Paginator
    {
        // Return all items (with pagination if needed)
    }

    public function massDelete(array $ids): void
    {
        // Delete multiple items
    }

    public function delete(DataWrapperContract $item, ?FieldsContract $fields = null): bool
    {
        // Delete a single item
    }

    public function save(DataWrapperContract $item, ?FieldsContract $fields = null): DataWrapperContract
    {
        // Save (create or update) an item
    }
}
```

### REST API Example

```php
namespace App\MoonShine\Resources;

use Illuminate\Support\Facades\Http;
use MoonShine\Contracts\Core\DependencyInjection\FieldsContract;
use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;
use MoonShine\Crud\Resources\CrudResource;

final class RestCrudResource extends CrudResource
{
    public function getItems(): iterable
    {
        yield from collect(Http::get('https://jsonplaceholder.typicode.com/todos')->json())
            ->map(fn ($item): DataWrapperContract => $this->getCaster()->cast($item))
            ->toArray();
    }

    public function findItem(bool $orFail = false): ?DataWrapperContract
    {
        return $this->getCaster()->cast(
            Http::get('https://jsonplaceholder.typicode.com/todos/' . $this->getItemID())->json()
        );
    }

    public function massDelete(array $ids): void
    {
        $this->beforeMassDeleting($ids);
        foreach ($ids as $id) {
            $this->delete($this->getCaster()->cast(['id' => $id]));
        }
        $this->afterMassDeleted($ids);
    }

    public function delete(DataWrapperContract $item, ?FieldsContract $fields = null): bool
    {
        return Http::delete(
            'https://jsonplaceholder.typicode.com/todos/' . $item->getOriginal()['id']
        )->successful();
    }

    public function save(DataWrapperContract $item, ?FieldsContract $fields = null): DataWrapperContract
    {
        $originalItem = $item->getOriginal();
        $data = request()->all();

        if ($originalItem['id'] ?? false) {
            return Http::put(
                'https://jsonplaceholder.typicode.com/todos/' . $originalItem['id'],
                $data
            )->json();
        }

        $this->isRecentlyCreated = true;

        return $this->getCaster()->cast(
            Http::post('https://jsonplaceholder.typicode.com/todos', $originalItem)->json()
        );
    }
}
```

### Page Helper Methods

In `DetailPage` and `FormPage`:

- `getItem()` -- get the current resource data item
- `isItemExists()` -- check if a resource data item exists

### Full Customization

For complete control, implement `CrudResourceContract` directly instead of extending `CrudResource`.

---

## JsonResponse

**V4 CHANGE**: `MoonShineJsonResponse` renamed to `JsonResponse` at `MoonShine\Crud\JsonResponse`.

### Merge

```php
use MoonShine\Crud\JsonResponse;

JsonResponse::make()
    ->merge(['options' => $options, 'custom_key' => 'custom_value']);
```

Useful for async search in Select or BelongsToMany fields when returning options with additional actions.

### Toast

```php
JsonResponse::make()
    ->toast('My message', ToastType::SUCCESS, duration: 3000);
```

### Redirect

```php
JsonResponse::make()->redirect('/');
```

### Events

```php
JsonResponse::make()
    ->events([AlpineJs::event(JsEvent::TABLE_UPDATED, 'index')]);
```

### Html

```php
// Insert into the requesting component's selector
JsonResponse::make()->html('Content');

// With mode
JsonResponse::make()->html('Content', HtmlMode::BEFORE_END);
```

HtmlMode values: `INNER_HTML`, `OUTER_HTML`, `BEFORE_BEGIN`, `AFTER_BEGIN`, `BEFORE_END`, `AFTER_END`.

### HtmlData (multiple selectors)

```php
JsonResponse::make()
    ->htmlData((string) Text::make('One'), '#selector1')
    ->htmlData((string) Text::make('Two'), '#selector2', HtmlMode::BEFORE_END);
```

### Fields Values

```php
JsonResponse::make()
    ->fieldsValues([
        '.field-title-1' => 'some value 1',
        '.field-title-2' => 'some value 2',
    ]);
```

When filling the field, the `change` event will also be triggered.

---

## Notifications

**V4 CHANGE**: `NotificationButton` moved to `MoonShine\Crud\Notifications\NotificationButton`.

Uses Laravel Database Notification by default, with replaceable abstractions.

### Send Notification

```php
use MoonShine\Laravel\Notifications\MoonShineNotification;
use MoonShine\Crud\Notifications\NotificationButton;
use MoonShine\Support\Enums\Color;

MoonShineNotification::send(
    message: 'Notification text',
    button: new NotificationButton('Click me', 'https://moonshine.cutcode.dev', attributes: ['target' => '_blank']),
    ids: [1, 2, 3],       // admin user IDs (null = all)
    color: Color::GREEN,
    icon: 'information-circle'
);
```

### Via Dependency Injection

```php
use MoonShine\Crud\Contracts\Notifications\MoonShineNotificationContract;

public function di(MoonShineNotificationContract $notification)
{
    $notification->notify('Hello');
}
```

### Configuration

```php
// config/moonshine.php
'use_notifications' => true,
'use_database_notifications' => true,

// Or in MoonShineServiceProvider
$config->useNotifications();
$config->useDatabaseNotifications();
```

### Custom Notification System

Implement these interfaces: `MoonShineNotificationContract`, `NotificationItemContract`, `NotificationButtonContract`.

```php
public function boot(): void
{
    $this->app->singleton(
        MoonShineNotificationContract::class,
        MyNotificationSystem::class
    );
}
```

WebSocket notifications available via the Rush package.

---

## Toasts

**V4 CHANGE**: `MoonShineUI::toast()` replaced by global `toast()` helper.

```php
use MoonShine\Support\Enums\ToastType;

toast(message: 'Hello');
toast(message: 'Success', type: ToastType::SUCCESS, duration: 3000);
toast(message: 'Sticky toast', duration: false);
```

Via JsonResponse:

```php
JsonResponse::make()->toast('Test', type: ToastType::SUCCESS, duration: 1000);
```

Via JS event:

```php
ActionButton::make('Toast')->dispatchEvent(
    AlpineJs::event(JsEvent::TOAST, params: ToastEventParams::make(ToastType::SUCCESS, 'Hello', duration: 2000))
);
```

---

## Package Development

### ServiceProvider Basics

```php
namespace Author\MoonShineMyPackage;

use Illuminate\Support\ServiceProvider;
use MoonShine\Contracts\Core\DependencyInjection\CoreContract;
use MoonShine\Laravel\DependencyInjection\MoonShine;

class MyPackageServiceProvider extends ServiceProvider
{
    /** @param MoonShine $core */
    public function boot(CoreContract $core): void
    {
        $core
            ->resources([MyPackageResource::class])
            ->pages([MyPackagePage::class]);
    }
}
```

### Menu Manager

```php
use MoonShine\Contracts\MenuManager\MenuManagerContract;

public function boot(CoreContract $core, MenuManagerContract $menu): void
{
    $menu->add([
        MenuItem::make(MyPackagePage::class)
    ]);
}
```

### Asset Manager

```php
use MoonShine\Contracts\AssetManager\AssetManagerContract;

public function boot(CoreContract $core, AssetManagerContract $assets): void
{
    $assets->add([
        InlineCss::make('body {background: red;}')
    ]);
}
```

### Color Manager

```php
use MoonShine\Contracts\ColorManager\ColorManagerContract;

public function boot(CoreContract $core, ColorManagerContract $colors): void
{
    $colors
        ->background('#A3C3D9')
        ->primary('#CCD6EB')
        ->secondary('#AE76A6');
}
```

### Authorization Rules

```php
use MoonShine\Contracts\Core\DependencyInjection\ConfiguratorContract;

public function boot(ConfiguratorContract $configurator): void
{
    $configurator->authorizationRules(
        static function (ResourceContract $resource, Model $user, Ability $ability): bool {
            return true;
        }
    );
}
```

### Push Components to Pages

```php
public function boot(): void
{
    ProfilePage::pushComponent(
        fn() => MyPackageComponent::make()
    );
}
```

### Traits for Resources/Pages

```php
trait HasMyPackageTrait
{
    public function loadHasMyPackageTrait(): void
    {
        $this->getFormPage()->addAssets([
            Js::make('vendor/my-package/js/app.js'),
            Css::make('vendor/my-package/css/app.css'),
        ]);
    }

    public function modifyFormComponent(ComponentContract $component): ComponentContract
    {
        return parent::modifyFormComponent($component)->fields([
            Modal::make('This is my package modal.', ''),
            ...$component->getFields()->toArray(),
        ]);
    }
}
```

### Composer Auto-Discovery

```json
"extra": {
    "laravel": {
        "providers": [
            "Author\\MoonShineMyPackage\\MyPackageServiceProvider"
        ]
    }
}
```

### Custom Field Example (Quill Editor)

```php
namespace App\MoonShine\Fields;

use MoonShine\AssetManager\Css;
use MoonShine\AssetManager\Js;
use MoonShine\UI\Fields\Textarea;

final class Quill extends Textarea
{
    protected string $view = 'moonshine-quill::fields.quill';

    public function assets(): array
    {
        return [
            Css::make('/css/moonshine/quill/quill.snow.css'),
            Js::make('/js/moonshine/quill/quill.js'),
            Js::make('/js/moonshine/quill/quill-init.js'),
        ];
    }
}
```
