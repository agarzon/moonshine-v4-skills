# MoonShine v4 -- Events, Buttons, Metrics & Import/Export Reference

## CRUD Events

In v4 the parameter type is `DataWrapperContract` instead of `mixed`.

```php
use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;

class PostResource extends ModelResource
{
    protected function beforeCreating(DataWrapperContract $item): DataWrapperContract
    {
        if (auth()->user()->moonshine_user_role_id !== 1) {
            request()->merge(['author_id' => auth()->id()]);
        }
        return $item;
    }

    protected function afterCreated(DataWrapperContract $item): DataWrapperContract { return $item; }
    protected function beforeUpdating(DataWrapperContract $item): DataWrapperContract { return $item; }
    protected function afterUpdated(DataWrapperContract $item): DataWrapperContract { return $item; }
    protected function beforeDeleting(DataWrapperContract $item): DataWrapperContract { return $item; }
    protected function afterDeleted(DataWrapperContract $item): DataWrapperContract { return $item; }
    protected function beforeMassDeleting(array $ids): void {}
    protected function afterMassDeleted(array $ids): void {}
}
```

### Practical Examples

```php
// Auto-set author
protected function beforeCreating(DataWrapperContract $item): DataWrapperContract
{
    request()->merge(['author_id' => auth()->id()]);
    return $item;
}

// Clear cache after changes
protected function afterCreated(DataWrapperContract $item): DataWrapperContract
{
    cache()->tags(['posts'])->flush();
    return $item;
}

// Log mass deletions
protected function beforeMassDeleting(array $ids): void
{
    logger()->info('Mass deleting posts', ['ids' => $ids, 'user' => auth()->id()]);
}
```

## Resource Lifecycle Hooks

```php
protected function onLoad(): void
{
    // Resource is active
    $this->getAssetManager()->add(Css::make('/css/posts.css'));
}

protected function onBoot(): void
{
    // MoonShine creates this resource instance
}
```

### Trait Hooks

Use naming convention `load{TraitName}` / `boot{TraitName}`:

```php
trait WithAuditLog
{
    protected function loadWithAuditLog(): void { /* runs during onLoad */ }
    protected function bootWithAuditLog(): void { /* runs during onBoot */ }
}
```

## CRUD Operation Handlers

Override save/delete/mass-delete logic using attributes:

```php
use MoonShine\Crud\Attributes\SaveHandler;
use MoonShine\Crud\Attributes\DestroyHandler;
use MoonShine\Crud\Attributes\MassDestroyHandler;

#[SaveHandler(PostHandlers::class, 'save')]
#[DestroyHandler(PostHandlers::class, 'destroy')]
#[MassDestroyHandler(PostHandlers::class, 'massDestroy')]
class PostResource extends ModelResource { }
```

```php
final readonly class PostHandlers
{
    public function save(Post $model, array $data): Post
    {
        $model->fill($data);
        $model->save();
        return $model;
    }

    public function destroy(Post $model): bool { return $model->delete(); }

    public function massDestroy(array $ids): void
    {
        foreach ($ids as $id) { Post::query()->whereKey($id)->delete(); }
    }
}
```

Or invokable classes: `#[SaveHandler(PostSaveHandler::class)]` with `public function __invoke(Post $model, array $data): Post`.

## Buttons

In v4, buttons are defined on page classes.

### IndexPage Buttons

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;

class PostIndexPage extends IndexPage
{
    // Above the table (left/right)
    protected function topLeftButtons(): ListOf
    {
        return parent::topLeftButtons()->add(
            ActionButton::make('Refresh', '#')
                ->dispatchEvent(AlpineJs::event(JsEvent::TABLE_UPDATED, $this->getListComponentName()))
        );
    }

    protected function topRightButtons(): ListOf
    {
        return parent::topRightButtons()->add(ActionButton::make('Export CSV', '/export'));
    }

    // Per-row buttons in the table
    protected function buttons(): ListOf
    {
        return parent::buttons()->prepend(
            ActionButton::make('Preview', fn(Model $item) => '/preview/' . $item->getKey())
        );
    }
}
```

#### Bulk Actions

```php
protected function buttons(): ListOf
{
    return parent::buttons()->prepend(
        ActionButton::make('Publish All', '/bulk-publish')->bulk()
    );
}
```

#### Modify Built-in Buttons

```php
use MoonShine\Contracts\UI\ActionButtonContract;

protected function modifyCreateButton(ActionButtonContract $button): ActionButtonContract { return $button->error(); }
protected function modifyDetailButton(ActionButtonContract $button): ActionButtonContract { return $button->warning(); }
protected function modifyEditButton(ActionButtonContract $button): ActionButtonContract { return $button->icon('pencil-square'); }
protected function modifyDeleteButton(ActionButtonContract $button): ActionButtonContract { return $button->icon('x-mark'); }
protected function modifyMassDeleteButton(ActionButtonContract $button): ActionButtonContract { return $button->icon('x-mark'); }
protected function modifyFiltersButton(ActionButtonContract $button): ActionButtonContract { return $button->error(); }
```

### FormPage Buttons

```php
class PostFormPage extends FormPage
{
    // Page-level buttons (outside the form)
    protected function buttons(): ListOf
    {
        return parent::buttons()->add(
            ActionButton::make('Custom Action')->method('updateSomething')
        );
    }

    // Inside the form (next to Save)
    protected function formButtons(): ListOf
    {
        return parent::formButtons()->add(
            ActionButton::make('Back', fn() => $this->getIndexPageUrl())->class('btn-lg')
        );
    }
}
```

### DetailPage Buttons

```php
class PostDetailPage extends DetailPage
{
    protected function buttons(): ListOf
    {
        return parent::buttons()->add(ActionButton::make('Print', '/print'));
    }
}
```

### Link to Custom Page

```php
public function buttons(): ListOf
{
    return parent::buttons()->add(
        ActionButton::make('To page', url: fn($model) => $this->getResource()?->getPageUrl(
            PostCustomPage::class, params: ['resourceItem' => $model->getKey()]
        )),
    );
}
```

## Metrics

Defined on `IndexPage`. Display statistics blocks.

```php
use MoonShine\UI\Components\Metrics\Wrapped\ValueMetric;

class PostIndexPage extends IndexPage
{
    protected function metrics(): array
    {
        return [
            ValueMetric::make('Total Posts')->value(fn() => Post::count())->columnSpan(4),
            ValueMetric::make('Published')->value(fn() => Post::where('status', 'published')->count())->columnSpan(4),
            ValueMetric::make('Drafts')->value(fn() => Post::where('status', 'draft')->count())->columnSpan(4),
        ];
    }
}
```

### Fragment Wrapper for Auto-Update

```php
use MoonShine\Crud\Components\Fragment;

protected function fragmentMetrics(): ?Closure
{
    return static fn(array $components): Fragment => Fragment::make($components)->name('metrics');
}
```

## Response Modifiers

### Resource-Level (Async Mode)

```php
use MoonShine\Crud\JsonResponse;

public function modifySaveResponse(JsonResponse $response): JsonResponse { return $response; }
public function modifyDestroyResponse(JsonResponse $response): JsonResponse { return $response; }
public function modifyMassDeleteResponse(JsonResponse $response): JsonResponse { return $response; }
public function modifyErrorResponse(Response $response, Throwable $exception): Response { return $response; }
```

### Page-Level

```php
protected function modifyResponse(): ?Response
{
    if (request()->has('redirect')) { return redirect()->to(request()->get('redirect')); }
    return null;
}
```

## Import / Export

Requires `moonshine/import-export` package.

### Setup

```bash
composer require moonshine/import-export
```

```php
use MoonShine\ImportExport\Contracts\HasImportExportContract;
use MoonShine\ImportExport\Traits\ImportExportConcern;

class PostResource extends ModelResource implements HasImportExportContract
{
    use ImportExportConcern;
}
```

### Import Fields

```php
protected function importFields(): iterable
{
    return [ID::make(), Text::make('Title')];
}
```

Always include `ID` so existing records get updated. Use `fromRaw()` to modify values during import:

```php
Enum::make('Status')->attach(StatusEnum::class)
    ->fromRaw(static fn(string $raw, Enum $ctx) => StatusEnum::tryFrom($raw)),
```

### Import Settings

```php
protected function import(): ?Handler
{
    return ImportHandler::make(__('moonshine::ui.import'))
        ->notifyUsers(fn(ImportHandler $ctx) => [auth()->id()])
        ->disk('public')->dir('/imports')->deleteAfter()
        ->delimiter(',')->queue()
        ->modifyButton(fn(ActionButton $btn) => $btn->class('my-class'));
}
```

Return `null` to hide the import button.

### Import Events

```php
public function beforeImportFilling(array $data): array { return $data; }
public function beforeImported(mixed $item): mixed { return $item; }
public function afterImported(mixed $item): mixed { return $item; }
```

### Export Fields

```php
protected function exportFields(): iterable
{
    return [ID::make(), Text::make('Title')];
}
```

Use `modifyRawValue()` to modify values during export.

### Export Settings

```php
protected function export(): ?Handler
{
    return ExportHandler::make(__('moonshine::ui.export'))
        ->notifyUsers(fn() => [auth()->id()])
        ->disk('public')->filename(sprintf('export_%s', date('Ymd-His')))
        ->dir('/exports')->csv()->delimiter(',')
        ->withConfirm()->queue();
}
```

Return `null` to hide the export button. Default format is XLSX; use `->csv()` for CSV.

### Common Handler Methods

Both `ImportHandler` and `ExportHandler` support: `icon()`, `queue()`, `modifyButton()`, `notifyUsers()`, `when()`, `unless()`.

### Custom Implementation

```bash
php artisan moonshine:handler CustomExportHandler
```

Extend `ImportHandler` or `ExportHandler` instead of base `Handler`.

## Complete Resource Example

```php
namespace App\MoonShine\Resources\Post;

use App\Models\Post;
use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;
use MoonShine\Crud\Handlers\Handler;
use MoonShine\Crud\JsonResponse;
use MoonShine\ImportExport\Contracts\HasImportExportContract;
use MoonShine\ImportExport\ExportHandler;
use MoonShine\ImportExport\ImportHandler;
use MoonShine\ImportExport\Traits\ImportExportConcern;
use MoonShine\Laravel\Resources\ModelResource;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

class PostResource extends ModelResource implements HasImportExportContract
{
    use ImportExportConcern;

    protected string $model = Post::class;
    protected string $title = 'Posts';

    protected function pages(): array
    {
        return [PostIndexPage::class, PostFormPage::class, PostDetailPage::class];
    }

    protected function importFields(): iterable { return [ID::make(), Text::make('Title')]; }
    protected function exportFields(): iterable { return [ID::make(), Text::make('Title')]; }
    protected function import(): ?Handler { return ImportHandler::make('Import')->queue(); }
    protected function export(): ?Handler { return ExportHandler::make('Export')->csv()->queue(); }

    protected function beforeCreating(DataWrapperContract $item): DataWrapperContract
    {
        request()->merge(['author_id' => auth()->id()]);
        return $item;
    }

    protected function afterCreated(DataWrapperContract $item): DataWrapperContract
    {
        cache()->tags(['posts'])->flush();
        return $item;
    }

    public function modifySaveResponse(JsonResponse $response): JsonResponse { return $response; }
}
```
