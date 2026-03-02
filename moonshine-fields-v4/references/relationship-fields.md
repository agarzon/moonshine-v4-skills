# Relationship Fields Reference (MoonShine v4)

All relationship fields require a registered `ModelResource`. The resource must be registered in `MoonShineServiceProvider` via `$core->resources()`.

## BelongsTo

`MoonShine\Laravel\Fields\Relationships\BelongsTo`

```php
use MoonShine\Laravel\Fields\Relationships\BelongsTo;

BelongsTo::make(
    Closure|string $label,
    ?string $relationName = null,      // auto-detected from label via camelCase
    Closure|string|null $formatted = null, // column or closure for display
    ModelResource|string|null $resource = null, // auto-detected from relation
)

// Minimal usage (all auto-detected)
BelongsTo::make('User')

// Explicit
BelongsTo::make('User', 'user', resource: UserResource::class)

// Custom display
BelongsTo::make('Country', 'country', fn($item) => "$item->id. $item->title")

// Custom column after fill
BelongsTo::make('Category', resource: CategoryResource::class)
    ->afterFill(fn($field) => $field->setColumn('changed_category_id'))
```

### Key Methods

```php
->default(Country::find(1))      // default value (pass Model instance)
->nullable()                      // allow NULL
->placeholder('Select country')
->searchable()                    // enable Tom Select search
->native()                        // disable Tom Select, use native select

// Creatable - create new related object via modal
->creatable()
->creatable(button: ActionButton::make('Custom button', ''))

// When inside another relationship (e.g., HasMany modal)
->creatable(fragmentUrl: $this->getResource()->getFragmentLoadUrl(
    'user', page: $this, key: $this->getItem()?->getKey()
))

// Query modification
->valuesQuery(fn(Builder $query, Field $field) => $query->where('active', true))

// Async search
->asyncSearch()
->asyncSearch('title', searchQuery: fn(Builder $q, string $term, Request $r, Field $f) => $q->where('id', '!=', 2), formatted: fn($item, Field $f) => $item->id . ' | ' . $item->title, limit: 10)

// Associated fields (dependent dropdowns)
->associatedWith('country_id')
->associatedWith('country_id')->asyncOnInit(whenOpen: false)

// Images in dropdown
->withImage('thumb', 'public', 'countries')

// Link customization
->link(link: fn(string $value, BelongsTo $ctx) => $ctx->getResource()->getDetailPageUrl($ctx->getData()->getKey()), name: fn(string $value) => $value, icon: 'users', blank: true)
```

---

## HasOne

`MoonShine\Laravel\Fields\Relationships\HasOne`

> The `$formatted` parameter is NOT used in HasOne.

```php
use MoonShine\Laravel\Fields\Relationships\HasOne;

HasOne::make('Profile', 'profile', resource: ProfileResource::class)
HasOne::make('Profile')  // auto-detect relation and resource
```

### Fields

```php
HasOne::make('Profile', resource: ProfileResource::class)
    ->fields([
        Phone::make('Phone'),
        Text::make('Address'),
    ])
```

### Parent ID (ResourceWithParent)

```php
use MoonShine\Laravel\Traits\Resource\ResourceWithParent;

class PostImageResource extends ModelResource
{
    use ResourceWithParent;

    protected function getParentResourceClassName(): string { return PostResource::class; }
    protected function getParentRelationName(): string { return 'post'; }
}

// Usage: $this->getParentId();
```

### Modification

```php
->modifyTable(fn(TableBuilder $table) => $table)
->modifyForm(fn(FormBuilder $form) => $form->submit('Custom title'))
->redirectAfter(fn(int $parentId) => route('home'))
```

### Display Modes

```php
->tabMode()                  // display in Tabs component
->modalMode()                // display in modal triggered by button
->modalMode(
    modifyButton: fn(ActionButtonContract $button, HasOne $ctx) => $button->warning(),
    modifyModal: fn(Modal $modal, ActionButtonContract $ctx) => $modal->autoClose(false)
)
->disableOutside()           // render inside main form (only works in modalMode)
```

---

## HasMany

`MoonShine\Laravel\Fields\Relationships\HasMany`

> The `$formatted` parameter is NOT used. A BelongsTo field referring to the parent is required.

```php
use MoonShine\Laravel\Fields\Relationships\HasMany;

HasMany::make('Comments', 'comments', resource: CommentResource::class)
HasMany::make('Comments')    // auto-detect
->disableOutside()           // display inside main form
```

### Fields and Creatable

```php
->fields([BelongsTo::make('User'), Text::make('Text')])
->creatable()
->creatable(button: ActionButton::make('Custom button', ''))
```

### Limit and RelatedLink

```php
->limit(1)
->relatedLink()
->relatedLink('comment')
->relatedLink(condition: fn(int $count, Field $field) => $count > 10)
```

### Edit Button and Modals

```php
->changeEditButton(ActionButton::make('Edit', fn(Comment $c) => app(CommentResource::class)->formPageUrl($c)))
->withoutModals()
```

### Searchable

```php
->searchable(false)  // disable search field on form page
```

### Modification Methods

```php
->modifyItemButtons(fn(ActionButton $detail, $edit, $delete, $massDelete, HasMany $ctx) => [$detail])
->modifyRelatedLink(fn(ActionButton $button, bool $preview) => $button->primary())
->modifyCreateButton(fn(ActionButton $button) => $button->setLabel('Custom create'))
->modifyEditButton(fn(ActionButton $button) => $button->setLabel('Custom edit'))
->modifyTable(fn(TableBuilder $table, bool $preview) => $table->customAttributes(['style' => 'background: blue']))
->modifyBuilder(fn(Relation $query, HasMany $ctx) => $query)
->redirectAfter(fn(int $parentId) => route('home'))
```

### Active Actions

```php
use MoonShine\Support\Enums\Action;

->activeActions(Action::VIEW, Action::UPDATE)
->withoutActions(Action::VIEW)
```

### Adding Buttons

```php
->indexButtons([ActionButton::make('Custom button')])
->formButtons([ActionButton::make('Custom form button')])
```

### Display Modes

```php
->tabMode()
->modalMode()
->modalMode(modifyButton: fn($btn, $ctx) => $btn->warning(), modifyModal: fn($modal, $ctx) => $modal->autoClose(false))
```

### Advanced: nowOn

Use HasMany outside CRUD pages:

```php
HasMany::make('Comments', resource: CommentResource::class)
    ->creatable()
    ->nowOn(page: $resource->getFormPage(), resource: $resource, params: ['resourceItem' => $item->getKey()])
    ->fillCast($item, new ModelCaster(Article::class))
```

---

## HasManyThrough

`MoonShine\Laravel\Fields\Relationships\HasManyThrough` - Inherits from HasMany. All HasMany methods available.

```php
use MoonShine\Laravel\Fields\Relationships\HasManyThrough;

HasManyThrough::make('Deployments', 'deployments', resource: DeploymentResource::class)
```

---

## HasOneThrough

`MoonShine\Laravel\Fields\Relationships\HasOneThrough` - Inherits from HasOne. All HasOne methods available.

```php
use MoonShine\Laravel\Fields\Relationships\HasOneThrough;

HasOneThrough::make('Car owner', 'carOwner', resource: OwnerResource::class)
```

---

## BelongsToMany

`MoonShine\Laravel\Fields\Relationships\BelongsToMany`

```php
use MoonShine\Laravel\Fields\Relationships\BelongsToMany;

BelongsToMany::make('Categories', 'categories', resource: CategoryResource::class)
BelongsToMany::make('Categories', 'categories', fn($item) => "$item->id. $item->title")
```

### Column Label and Pivot Fields

```php
->columnLabel('Title')
->fields([Text::make('Contact', 'text')])  // pivot fields
```

> Declare pivot fields in the Laravel relationship with `withPivot()`.

### Deduplication

```php
->deduplication(false)  // allow same related key with different pivot data
```

### Pivot Modal Mode

```php
->pivotModalMode()  // edit/add via modal (requires saved record)
->pivotCardsMode()  // cards layout instead of table

// Button customization in pivotModalMode
->modifyCreateButton(fn(ActionButtonContract $btn) => $btn->success())
->modifyEditButton(fn(ActionButtonContract $btn) => $btn->warning())
->modifyDeleteButton(fn(ActionButtonContract $btn) => $btn->secondary())
```

### Select Mode / Tree / Horizontal

```php
->selectMode()                       // dropdown instead of checkboxes
->tree('parent_id')                  // tree with checkboxes
->horizontalMode(true, minColWidth: '100px', maxColWidth: '33%')
```

### Preview Display

```php
->onlyCount()                        // show count only
->inLine(separator: ' ', badge: fn($model, $value) => Badge::make((string) $value, 'primary'), link: fn($prop, $value, $field) => Link::make(url, $value))
```

### RelatedLink

```php
->relatedLink()
->relatedLink('category')
->relatedLink(condition: fn(int $count, Field $field) => $count > 10)
```

### Values Query, Async Search, Associated

```php
->valuesQuery(fn(Builder $query, Field $field) => $query->where('active', true))
->asyncSearch('title', searchQuery: fn(Builder $q, string $term, Request $r, Field $f) => $q, formatted: fn($item, Field $f) => $item->title, limit: 10)
->associatedWith('country_id')
->withImage('thumb', 'public', 'countries')
```

### Buttons

```php
->buttons([ActionButton::make('Check all', '')->onClick(fn() => 'checkAll', 'prevent')])
->withCheckAll()
```

---

## MorphTo

`MoonShine\Laravel\Fields\Relationships\MorphTo` - Inherits from BelongsTo.

```php
use MoonShine\Laravel\Fields\Relationships\MorphTo;

MorphTo::make('Commentable')->types([
    Article::class => 'title'
])

// Array syntax: [field_name, display_label]
MorphTo::make('Imageable')->types([
    Company::class => ['short_name', 'Organization']
])

// With explicit resource
MorphTo::make('Commentable', resource: PolyCommentResource::class)->types([
    Post::class => 'name',
    Project::class => 'name',
])
```

---

## MorphOne / MorphMany / MorphToMany

```php
use MoonShine\Laravel\Fields\Relationships\MorphOne;    // inherits HasOne
use MoonShine\Laravel\Fields\Relationships\MorphMany;    // inherits HasMany
use MoonShine\Laravel\Fields\Relationships\MorphToMany;  // inherits BelongsToMany

MorphOne::make('Profile', 'profile', resource: ProfileResource::class)
MorphMany::make('Comments', 'comments', resource: CommentResource::class)
MorphToMany::make('Categories', 'categories', resource: CategoryResource::class)
```

All have the same capabilities as their parent classes.

---

## RelationRepeater

`MoonShine\Laravel\Fields\Relationships\RelationRepeater` - Inline HasMany/HasOne editing inside main form.

```php
use MoonShine\Laravel\Fields\Relationships\RelationRepeater;
use MoonShine\UI\Fields\ID;
use MoonShine\UI\Fields\Text;

RelationRepeater::make('Characteristics', 'characteristics')
    ->fields([
        ID::make(),              // ID field is REQUIRED
        Text::make('Name', 'name'),
        Text::make('Value', 'value'),
    ])
```

> The `ID` field is required, otherwise records will always be added (never updated).

### Vertical Mode

```php
RelationRepeater::make('Comments', 'comments')->vertical()
```

### Reorderable

```php
->reorderable(url: '/endpoint')
```

### Creatable / Removable

```php
->creatable(limit: 5)
->creatable(button: ActionButton::make('Add New'))
->removable()
->removable(attributes: ['@click.prevent' => 'customRemove'])
```

### Buttons and Modifiers

```php
->buttons([ActionButton::make('', '#')->icon('trash')->onClick(fn() => 'remove()', 'prevent')->showInLine()])
->modifyTable(fn(TableBuilder $table, bool $preview) => $table->customAttributes(['class' => 'custom']))
->modifyCreateButton(fn(ActionButton $button) => $button->customAttributes(['class' => 'btn-primary']))
->modifyRemoveButton(fn(ActionButton $button) => $button->customAttributes(['class' => 'btn-secondary']))
```
