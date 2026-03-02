# MoonShine v4 -- Filters, Search & Query Tags Reference

## Filters

### Basics

Filters in MoonShine v4 are created using the same field classes used for form input. In v4, filters are defined on the `IndexPage` class, not on the resource.

> The `filters()` method is also available on `ModelResource` for backward compatibility, but it is recommended to define filters directly in `IndexPage`.

If the method returns an empty array, the filters panel will not be displayed.

```php
namespace App\MoonShine\Resources\Post\Pages;

use MoonShine\Laravel\Pages\Crud\IndexPage;
use MoonShine\UI\Fields\Text;
use MoonShine\UI\Fields\Select;
use MoonShine\UI\Fields\Date;

class PostIndexPage extends IndexPage
{
    protected function filters(): iterable
    {
        return [
            Text::make('Title', 'title'),
            Select::make('Category', 'category_id')
                ->options([
                    1 => 'News',
                    2 => 'Articles',
                    3 => 'Reviews',
                ]),
            Date::make('Created At', 'created_at'),
        ];
    }
}
```

Some fields cannot participate in filtering and will be automatically excluded.

### Custom Filter Logic

Use the `onApply()` method on a field to override its filtering behavior:

```php
use Illuminate\Contracts\Database\Eloquent\Builder;

Text::make('Title', 'title')
    ->onApply(fn(Builder $query, mixed $value, Text $field) =>
        $query->where('title', 'LIKE', "%{$value}%")
    ),
```

### Cache Filter State

Persist filter selections across page reloads. Set on the resource:

```php
class PostResource extends ModelResource
{
    protected bool $saveQueryState = true;
}
```

### Filters OffCanvas Modifier

Customize the filters slide-out panel from the resource:

```php
use MoonShine\Contracts\UI\OffCanvasContract;

protected function modifyFiltersOffCanvas(OffCanvasContract $offCanvas): OffCanvasContract
{
    return $offCanvas->full()->autoClose(false);
}
```

### Modify Filters Button

On `IndexPage`, customize the filter button appearance:

```php
use MoonShine\Contracts\UI\ActionButtonContract;

protected function modifyFiltersButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->error();
}
```

## Search

### Basics

Search is configured on the resource. Specify which model fields participate in the search:

```php
class PostResource extends ModelResource
{
    protected function search(): array
    {
        return ['id', 'title', 'text'];
    }
}
```

If the method returns an empty array, the search input will not be displayed.

### Full-Text Search

Use the `SearchUsingFullText` attribute for full-text search. Make sure a full-text index exists on the columns.

```php
use MoonShine\Support\Attributes\SearchUsingFullText;

class PostResource extends ModelResource
{
    #[SearchUsingFullText(['title', 'text'])]
    protected function search(): array
    {
        return ['id'];
    }
}
```

### JSON Key Search

For `Json` fields used as key-value (`keyValue()`):

```php
protected function search(): array
{
    return ['data->title'];
}
```

For multidimensional Json fields:

```php
protected function search(): array
{
    return ['data->[*]->title'];
}
```

### Relation Search

Search through related model fields:

```php
protected function search(): array
{
    return ['category.title'];
}
```

### Custom Search Query

Override the search query entirely:

```php
use Illuminate\Contracts\Database\Eloquent\Builder;

protected function searchQuery(string $terms): void
{
    $this->newQuery()->where(function (Builder $builder) use ($terms): void {
        $builder->where('title', 'LIKE', "%{$terms}%")
            ->orWhere('slug', 'LIKE', "%{$terms}%");
    });
}
```

Extend the default search query:

```php
protected function searchQuery(string $terms): void
{
    parent::searchQuery($terms);

    $this->newQuery()->where(function (Builder $builder) use ($terms): void {
        $builder->orWhere('custom_field', 'LIKE', "%{$terms}%");
    });
}
```

Completely override search resolution including full-text:

```php
protected function resolveSearch(string $terms, ?iterable $fullTextColumns = null): static
{
    // Your complete custom search logic
    return $this;
}
```

### Global Search

MoonShine supports global search via Laravel Scout integration.

1. Install the package:

```bash
composer require moonshine/scout
php artisan vendor:publish --provider="MoonShine\Scout\Providers\ScoutServiceProvider"
```

2. Configure models in `config/moonshine-scout.php`:

```php
'models' => [
    Article::class,
    User::class,
],
```

3. Implement the interface in models:

```php
use Laravel\Scout\Builder;
use Laravel\Scout\Searchable;
use MoonShine\Scout\HasGlobalSearch;
use MoonShine\Scout\SearchableResponse;

class Article extends Model implements HasGlobalSearch
{
    use Searchable;

    public function searchableQuery(Builder $builder): Builder
    {
        return $builder->take(4);
    }

    public function toSearchableResponse(): SearchableResponse
    {
        return new SearchableResponse(
            group: 'Articles',
            title: $this->title,
            url: '/',
            preview: $this->text,
            image: $this->thumbnail,
        );
    }
}
```

4. Replace the Search component in Layout:

```php
protected function getSearchComponent(): ComponentContract
{
    return MoonShine\Scout\Components\Search::make();
}
```

## Query Modification

### Modify All Queries

Add global conditions to all resource queries. Defined on the resource:

```php
use Illuminate\Contracts\Database\Eloquent\Builder;

class PostResource extends ModelResource
{
    protected function modifyQueryBuilder(Builder $builder): Builder
    {
        return $builder->where('active', true);
    }
}
```

To completely override the base builder, override `newQuery()`.

### Modify Single-Record Query

For retrieving individual records (edit, detail pages):

```php
protected function modifyItemQueryBuilder(Builder $builder): Builder
{
    return $builder->withTrashed();
}
```

To completely override single-record retrieval, override `findItem()`.

### Eager Loading

```php
class PostResource extends ModelResource
{
    protected array $with = ['user', 'categories', 'tags'];
}
```

### Custom Sorting

Override the `resolveOrder()` method for custom sorting logic:

```php
protected function resolveOrder(string $column, string $direction, ?Closure $callback): static
{
    if ($callback instanceof Closure) {
        $callback($this->newQuery(), $column, $direction);
    } else {
        $this->newQuery()->orderBy($column, $direction);
    }

    return $this;
}
```

## Query Tags

### Basics

Query tags are quick-filter buttons displayed above the index table. In v4, they are defined on `IndexPage`.

> The `queryTags()` method is also available on `ModelResource` for backward compatibility, but it is recommended to define tags directly in `IndexPage`.

```php
namespace App\MoonShine\Resources\Post\Pages;

use Illuminate\Database\Eloquent\Builder;
use MoonShine\Laravel\Pages\Crud\IndexPage;
use MoonShine\Laravel\QueryTags\QueryTag;

class PostIndexPage extends IndexPage
{
    protected function queryTags(): array
    {
        return [
            QueryTag::make(
                'With author',
                fn(Builder $query) => $query->whereNotNull('author_id')
            ),
            QueryTag::make(
                'Published',
                fn(Builder $query) => $query->where('status', 'published')
            ),
        ];
    }
}
```

### Icon

Add an icon to a query tag:

```php
QueryTag::make(
    'With author',
    fn(Builder $query) => $query->whereNotNull('author_id')
)->icon('users')
```

### Default Active Tag

Make a tag active by default:

```php
QueryTag::make(
    'All posts',
    fn(Builder $query) => $query
)->default()
```

With a condition:

```php
->default(fn() => request()->missing('query-tag'))
```

### Display Condition

Show a tag only when certain conditions are met:

```php
QueryTag::make(
    'Admin posts',
    fn(Builder $query) => $query->where('is_admin_post', true)
)->canSee(fn() => auth()->user()->moonshine_user_role_id === 1)
```

### Custom Alias

By default, the URL value is generated from the label. Set a custom alias:

```php
QueryTag::make(
    'Archived Posts',
    fn(Builder $query) => $query->where('is_archived', true)
)->alias('archive')
```

### Dropdown Display

Display query tags in a dropdown instead of inline buttons. Set on the resource:

```php
class PostResource extends ModelResource
{
    protected bool $queryTagsInDropdown = true;
}
```

### Events

Fire custom events after query tag selection:

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

QueryTag::make('Refresh data', fn($q) => $q)
    ->events([
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'custom-fragment'),
    ])
```

### Button Modifier

Customize the query tag button:

```php
use MoonShine\UI\Components\ActionButton;

QueryTag::make('Custom Tag', fn($q) => $q)
    ->modifyButton(fn(ActionButton $btn) => $btn->class('btn-primary'))
```

### Complete Query Tags Example

```php
use Illuminate\Database\Eloquent\Builder;
use MoonShine\Laravel\QueryTags\QueryTag;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;
use MoonShine\UI\Components\ActionButton;

class PostIndexPage extends IndexPage
{
    protected function queryTags(): array
    {
        return [
            QueryTag::make(
                'All',
                fn(Builder $query) => $query
            )->default()->icon('list-bullet'),

            QueryTag::make(
                'Published',
                fn(Builder $query) => $query->where('status', 'published')
            )->icon('check-circle')->alias('published'),

            QueryTag::make(
                'Drafts',
                fn(Builder $query) => $query->where('status', 'draft')
            )->icon('pencil')->alias('drafts'),

            QueryTag::make(
                'Archived',
                fn(Builder $query) => $query->where('is_archived', true)
            )
                ->icon('archive-box')
                ->alias('archived')
                ->canSee(fn() => auth()->user()->isAdmin())
                ->events([
                    AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'metrics'),
                ])
                ->modifyButton(fn(ActionButton $btn) => $btn->class('btn-warning')),
        ];
    }
}
```

### Link to Query Tag via URL

```php
$resource->getIndexPageUrl(['query-tag' => $tag->uri()]);
```
