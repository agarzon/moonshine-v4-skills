# MoonShine v4 -- CRUD Pages Reference

## Overview

In MoonShine v4, CRUD pages are first-class citizens. When you create a resource, three page classes are generated: `IndexPage`, `FormPage`, and `DetailPage`. Each page owns its fields, validation, component modifiers, buttons, and more.

## Creating Pages

Pages are auto-generated with `php artisan moonshine:resource Post` in `app/MoonShine/Resources/Post/Pages/`. Register in the resource:

```php
class PostResource extends ModelResource
{
    protected function pages(): array
    {
        return [PostIndexPage::class, PostFormPage::class, PostDetailPage::class];
    }
}
```

Page types: `PageType::INDEX`, `PageType::FORM`, `PageType::DETAIL`.

## IndexPage

### Full IndexPage Example

```php
namespace App\MoonShine\Resources\Post\Pages;

use App\Models\Post;
use Illuminate\Database\Eloquent\Builder;
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Laravel\Pages\Crud\IndexPage;
use MoonShine\Laravel\QueryTags\QueryTag;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Components\Metrics\Wrapped\ValueMetric;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

class PostIndexPage extends IndexPage
{
    protected bool $isAsync = true;
    protected bool $isLazy = false;

    protected function fields(): iterable
    {
        return [ID::make()->sortable(), Text::make('Title')];
    }

    protected function filters(): iterable
    {
        return [Text::make('Title', 'title')];
    }

    protected function metrics(): array
    {
        return [
            ValueMetric::make('Articles')->value(fn() => Post::count())->columnSpan(6),
        ];
    }

    protected function queryTags(): array
    {
        return [
            QueryTag::make('With author', fn(Builder $query) => $query->whereNotNull('author_id')),
        ];
    }

    protected function topLeftButtons(): ListOf
    {
        return parent::topLeftButtons()->add(
            ActionButton::make('Refresh', '#')
                ->dispatchEvent(AlpineJs::event(JsEvent::TABLE_UPDATED, $this->getListComponentName()))
        );
    }

    protected function buttons(): ListOf
    {
        return parent::buttons()->prepend(
            ActionButton::make('Preview', fn($item) => '/preview/' . $item->getKey())
        );
    }

    protected function modifyListComponent(ComponentContract $component): ComponentContract
    {
        return $component->sticky()->stickyButtons()->columnSelection();
    }
}
```

### Get/Replace Main Component

```php
$page->getListComponent();
$resource->getIndexPage()->getListComponent();
```

Replace with custom class: `protected string $component = CustomListComponent::class;` implementing `DefaultListComponentContract`.

## FormPage

### Full FormPage Example

```php
namespace App\MoonShine\Resources\Post\Pages;

use MoonShine\Contracts\Core\TypeCasts\DataWrapperContract;
use MoonShine\Contracts\UI\FormBuilderContract;
use MoonShine\Laravel\Pages\Crud\FormPage;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Components\Layout\Box;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

class PostFormPage extends FormPage
{
    protected bool $isPrecognitive = false;

    protected function fields(): iterable
    {
        return [
            Box::make([
                ID::make(),
                Text::make('Title')->required(),
            ]),
        ];
    }

    protected function rules(DataWrapperContract $item): array
    {
        return ['title' => ['required', 'string', 'min:5']];
    }

    public function validationMessages(): array
    {
        return ['title.required' => 'Title is required'];
    }

    public function prepareForValidation(): void
    {
        request()->merge(['slug' => request()->string('slug')->lower()->value()]);
    }

    protected function buttons(): ListOf
    {
        return parent::buttons()->add(ActionButton::make('Action')->method('doSomething'));
    }

    protected function formButtons(): ListOf
    {
        return parent::formButtons()->add(
            ActionButton::make('Back', fn() => $this->getIndexPageUrl())->class('btn-lg')
        );
    }

    protected function modifyFormComponent(FormBuilderContract $component): FormBuilderContract
    {
        return $component->withoutRedirect();
    }
}
```

Get/Replace: `$page->getFormComponent();` or set `protected string $component = CustomFormComponent::class;`.

## DetailPage

### Full DetailPage Example

```php
namespace App\MoonShine\Resources\Post\Pages;

use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Contracts\UI\FieldContract;
use MoonShine\Laravel\Pages\Crud\DetailPage;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Components\Layout\Column;
use MoonShine\UI\Components\Table\TableBuilder;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

class PostDetailPage extends DetailPage
{
    protected function fields(): iterable
    {
        return [ID::make(), Text::make('Title'), Text::make('Content')];
    }

    protected function buttons(): ListOf
    {
        return parent::buttons()->add(ActionButton::make('Print', '/print'));
    }

    public function modifyDetailComponent(ComponentContract $component): ComponentContract
    {
        return $component->vertical(
            title: fn(FieldContract $field, Column $default, TableBuilder $ctx) => $default->columnSpan(2),
            value: fn(FieldContract $field, Column $default, TableBuilder $ctx) => $default->columnSpan(10),
        );
    }
}
```

Get/Replace: `$page->getDetailComponent();` or set `protected string $component = CustomDetailComponent::class;`.

## Page Layers

All CRUD pages have three layers: `TopLayer` (metrics, extra buttons), `MainLayer` (table/form), `BottomLayer` (additional info).

```php
class PostIndexPage extends IndexPage
{
    protected function topLayer(): array
    {
        return [Heading::make('Custom top'), ...parent::topLayer()];
    }

    protected function bottomLayer(): array
    {
        return [Heading::make('Extra info'), ...parent::bottomLayer()];
    }
}
```

Push from resource:

```php
protected function onLoad(): void
{
    $this->getFormPage()->pushToLayer(
        layer: Layer::BOTTOM,
        component: Permissions::make('Permissions', $this)
    );
}
```

Access: `$this->getLayerComponents(Layer::BOTTOM);`

## Page Lifecycle

```php
protected function onLoad(): void { parent::onLoad(); /* page is active */ }
protected function booted(): void { parent::booted(); /* instance created */ }
```

### Before Rendering

```php
protected function prepareBeforeRender(): void
{
    parent::prepareBeforeRender();
    if (! auth()->user()->isAdmin()) { abort(403); }
}
```

### Response Modifier

```php
protected function modifyResponse(): ?Response
{
    if (request()->has('redirect_to')) {
        return redirect()->to(request()->get('redirect_to'));
    }
    return null;
}
```

## Layout Modification

```php
protected ?string $layout = AppLayout::class;

// Or via attribute:
#[Layout(AppLayout::class)]
class PostIndexPage extends IndexPage { }

// Dynamic modification:
protected function modifyLayout(LayoutContract $layout): LayoutContract
{
    return $layout->title('Custom Title');
}
```

## Page Title, Subtitle, Alias, Breadcrumbs

```php
protected string $title = 'Posts';
protected string $subtitle = 'Manage your posts';
protected ?string $alias = 'custom-page-alias';

public function getBreadcrumbs(): array { return ['#' => $this->getTitle()]; }
public function getTitle(): string { return 'Posts (' . Post::count() . ')'; }
```

## Modal Windows (on Resource)

```php
protected bool $createInModal = true;
protected bool $editInModal = true;
protected bool $detailInModal = true;
```

Modifiers (all return `ModalContract`):

```php
protected function modifyCreateModal(ModalContract $modal): ModalContract { return $modal->full(); }
protected function modifyEditModal(ModalContract $modal): ModalContract { return $modal->full(); }
protected function modifyDetailModal(ModalContract $modal): ModalContract { return $modal->full(); }
protected function modifyDeleteModal(ModalContract $modal): ModalContract { return $modal->auto(); }
protected function modifyMassDeleteModal(ModalContract $modal): ModalContract { return $modal->auto(); }
```

Filters panel: `protected function modifyFiltersOffCanvas(OffCanvasContract $offCanvas): OffCanvasContract { return $offCanvas->full()->autoClose(false); }`

## Redirects, Active Actions, Authorization (on Resource)

```php
protected ?PageType $redirectAfterSave = PageType::FORM;
public function getRedirectAfterSave(): string { return $this->getIndexPageUrl(); }
public function getRedirectAfterDelete(): string { return $this->getIndexPageUrl(); }
```

```php
protected function activeActions(): ListOf
{
    return parent::activeActions()->except(Action::VIEW, Action::MASS_DELETE);
}
```

```php
protected bool $withPolicy = true;
```

Policy methods: `viewAny`, `view`, `create`, `update`, `delete`, `massDelete`, `restore`, `forceDelete`. Generate: `php artisan moonshine:policy PostPolicy`.

Custom logic: `protected function isCan(Ability $ability): bool { return parent::isCan($ability); }`

## Resource Routes

```php
$resource->getUrl();                              // first page
$resource->getRoute($name, $key, $params);        // advanced route
$resource->getPageUrl($page, $params, $fragment);  // page URL
$resource->getIndexPageUrl();                     // index page
$resource->getFormPageUrl();                      // create page
$resource->getFormPageUrl($item->getKey());       // edit page
$resource->getDetailPageUrl($item->getKey());     // detail page
$resource->getAsyncMethodUrl('updateSomething');  // async method
$resource->getRoute('crud.update', $data->getKey()); // PUT
$resource->getRoute('crud.store');                    // POST
$resource->getRoute('crud.destroy', $data->getKey()); // DELETE
$resource->getRoute('crud.massDelete');               // DELETE
```

### Active Page Detection

```php
$resource->getActivePage();     // ?PageContract
$resource->isIndexPage();       // bool
$resource->isFormPage();        // bool
$resource->isDetailPage();      // bool
$resource->isCreateFormPage();  // bool
$resource->isUpdateFormPage();  // bool
```

### toPage Helper

```php
toPage(page: PostIndexPage::class, resource: PostResource::class);
toPage(page: PostFormPage::class, redirect: true);
```

### Simulate Route

```php
class HomeController extends Controller
{
    public function __invoke(PostFormPage $page, PostResource $resource)
    {
        return $page->simulateRoute($page, $resource);
    }
}
```
