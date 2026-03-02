# Resource Recipes (MoonShine v4)

## Soft Deletes

Full implementation of soft deletes with restore, force delete, and query tags.

### Step 1: Prepare the model

Enable [soft deletes](https://laravel.com/docs/eloquent#soft-deleting) on your model using the `SoftDeletes` trait.

### Step 2: Override item query in resource

```php
use Illuminate\Contracts\Database\Eloquent\Builder;

protected function modifyItemQueryBuilder(Builder $builder): Builder
{
    return $builder->withTrashed();
}
```

### Step 3: Add functionality to the index page

```php
use Illuminate\Contracts\Database\Eloquent\Builder;
use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;
use MoonShine\Contracts\UI\ActionButtonContract;
use MoonShine\Crud\JsonResponse;
use MoonShine\Laravel\QueryTags\QueryTag;
use MoonShine\Support\Attributes\AsyncMethod;
use MoonShine\Support\ListOf;
use MoonShine\UI\Components\ActionButton;

// Query tag to filter only deleted items
protected function queryTags(): array
{
    return [
        QueryTag::make(
            'Deleted',
            static fn(Builder $q) => $q->onlyTrashed()
        )
    ];
}

// Restore and force delete buttons
protected function buttons(): ListOf
{
    return parent::buttons()->prepend(
        ActionButton::make('Restore')
            ->method(
                'restore',
                events: [$this->getListEventName()]
            )
            ->canSee(
                fn(Article $model) => $model->trashed()
            ),

        ActionButton::make('Force delete')
            ->method(
                'forceDelete',
                events: [$this->getListEventName()]
            )
            ->canSee(
                fn(Article $model) => $model->trashed()
            ),
    );
}

// Async methods for restore and force delete
#[AsyncMethod]
public function restore(CrudRequestContract $request): JsonResponse
{
    $item = $request->getResource()->getItem();
    $item->restore();

    return JsonResponse::make()->toast('Success');
}

#[AsyncMethod]
public function forceDelete(CrudRequestContract $request): JsonResponse
{
    $item = $request->getResource()->getItem();
    $item->forceDelete();

    return JsonResponse::make()->toast('Success');
}

// Hide standard delete for trashed items
protected function modifyDeleteButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->canSee(
        fn(Article $model) => !$model->trashed()
    );
}

// Hide mass delete when viewing trashed items
protected function modifyMassDeleteButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->canSee(
        fn() => request()->input('query-tag') !== 'deleted'
    );
}
```

---

## Reorderable Resource (Drag-and-Drop Sorting)

Allows drag-and-drop row reordering on the index page. Only suitable for resources with few records (no pagination).

### Step 1: Add properties and async method to the resource

```php
use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;
use MoonShine\Support\Attributes\AsyncMethod;
use MoonShine\Support\Enums\SortDirection;

protected string $sortColumn = 'position';

protected bool $usePagination = false;

protected SortDirection $sortDirection = SortDirection::ASC;

#[AsyncMethod]
public function reorder(CrudRequestContract $request): void
{
    if ($request->str('data')->isNotEmpty()) {
        $request->str('data')->explode(',')->each(
            fn($id, $position) => $this->getModel()
                ->where('id', $id)
                ->update([
                    'position' => $position + 1,
                ]),
        );
    }
}
```

### Step 2: Enable reorderable on the index page

```php
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\UI\Components\Table\TableBuilder;

/**
 * @param TableBuilder $component
 */
protected function modifyListComponent(ComponentContract $component): ComponentContract
{
    return $component->reorderable(
        $this->getResource()->getAsyncMethodUrl('reorder')
    );
}
```

### Step 3 (Optional): Add a drag handle column

Add a handle column to make dragging more precise:

```php
use MoonShine\UI\Fields\Preview;
use MoonShine\UI\Components\Icon;

protected function fields(): iterable
{
    return [
        Preview::make(
            column: '__handle',
            formatted: static fn () => Icon::make('bars-4'),
        )->customWrapperAttributes(['class' => 'handle', 'style' => 'cursor: move']),
        // ... Other table columns
    ];
}
```

Tell SortableJS which element is the handle via `data-handle`:

```php
protected function modifyListComponent(ComponentContract $component): ComponentContract
{
    return $component->reorderable(
        $this->getResource()->getAsyncMethodUrl('reorder')
    )->customAttributes([
        'data-handle' => '.handle',
    ]);
}
```

With this setup, rows can only be dragged by the handle column, leaving other columns free for interaction and text selection.

---

## HasMany Parent ID (File Paths by Parent)

Store files in directories named by the parent resource ID using `ResourceWithParent` trait.

```php
namespace App\MoonShine\Resources;

use App\Models\PostImage;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Image;
use MoonShine\Laravel\Fields\Relationships\BelongsTo;
use MoonShine\Laravel\Resources\ModelResource;
use MoonShine\Laravel\Traits\Resource\ResourceWithParent;

class PostImageResource extends ModelResource
{
    use ResourceWithParent;

    protected string $model = PostImage::class;

    protected function getParentResourceClassName(): string
    {
        return PostResource::class;
    }

    protected function getParentRelationName(): string
    {
        return 'post';
    }

    protected function formFields(): iterable
    {
        return [
            ID::make(),
            BelongsTo::make('Post'),
            Image::make('Path')
                ->when(
                    $parentId = $this->getParentId(),
                    static fn(Image $image): string => $image->dir("post_images/$parentId")
                ),
        ];
    }
}
```

---

## updateOnPreview for Pivot Fields

Implement inline editing of pivot fields directly on the index page using `asyncMethod`.

```php
use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;
use MoonShine\Crud\JsonResponse;
use MoonShine\UI\Components\Layout\{Column, Grid};
use MoonShine\UI\Fields\{ID, Number, Text};
use MoonShine\UI\Fields\Switcher;
use MoonShine\Laravel\Fields\Relationships\{BelongsTo, BelongsToMany};

protected function formFields(): iterable
{
    return [
        Grid::make([
            Column::make([
                ID::make()->sortable(),
                Text::make('Team title')->required(),
                Number::make('Team number'),
                BelongsTo::make('Tournament')->searchable(),
            ]),
            Column::make([
                BelongsToMany::make('Users')->fields([
                    Switcher::make('Approved')->updateOnPreview(
                        $this->getRouter()->getEndpoints()->method(
                            'updatePivot',
                            params: fn($data) => ['parent' => $data->pivot->tournamen_team_id]
                        )
                    ),
                ])->searchable(),
            ])
        ])
    ];
}

public function updatePivot(CrudRequestContract $request): JsonResponse
{
    $item = TournamentTeam::query()->findOrFail($request->get('parent'));

    $column = (string) $request->str('field')->remove('pivot.');

    $item->users()->updateExistingPivot($request->get('resourceItem'), [
        $column => $request->get('value'),
    ]);

    return JsonResponse::make()->toast('Success');
}
```
