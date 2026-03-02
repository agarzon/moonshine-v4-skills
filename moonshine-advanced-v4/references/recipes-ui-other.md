# UI and Other Recipes (MoonShine v4)

## Index Page via CardsBuilder

Replace the default table on the index page with CardsBuilder. Create a reusable component implementing `DefaultListComponentContract`.

### Step 1: Create the cards list component

```php
namespace App\MoonShine\Components;

use MoonShine\Contracts\Core\DependencyInjection\CoreContract;
use MoonShine\Contracts\Core\DependencyInjection\FieldsContract;
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Core\Traits\WithCore;
use MoonShine\Crud\Contracts\Page\IndexPageContract;
use MoonShine\Crud\Contracts\PageComponents\DefaultListComponentContract;
use MoonShine\UI\Components\CardsBuilder;

final class CardsListComponent implements DefaultListComponentContract
{
    use WithCore;

    public function __construct(CoreContract $core)
    {
        $this->setCore($core);
    }

    /**
     * @param iterable<array-key, mixed> $items
     */
    public function __invoke(
        IndexPageContract $page,
        iterable $items,
        FieldsContract $fields
    ): ComponentContract {
        $resource = $page->getResource();

        return CardsBuilder::make($items, $fields)
            ->cast($resource->getCaster())
            ->name($page->getListComponentName())
            ->overlay()
            ->url(fn ($user) => $page->getUrl())
            ->thumbnail(fn ($user) => asset($user->avatar))
            ->buttons($page->getButtons())
            ->when($page->isAsync(), function (CardsBuilder $cardsBuilder) use ($page): void {
                $cardsBuilder->async(
                    url: fn (): string => $page->getRouter()->getEndpoints()->component(
                        name: $cardsBuilder->getName(),
                        additionally: $this->getCore()->getRequest()->getRequest()->getQueryParams(),
                    ),
                );
            })
            ->when(
                ! \is_null($resource->getItemsResolver()),
                function (CardsBuilder $cardsBuilder) use ($resource): void {
                    $cardsBuilder->itemsResolver(
                        $resource->getItemsResolver(),
                    );
                },
            );
    }
}
```

### Step 2: Override the component property in the index page

```php
namespace App\MoonShine\Resources\User\Pages;

use App\MoonShine\Components\CardsListComponent;
use MoonShine\Crud\Contracts\PageComponents\DefaultListComponentContract;
use MoonShine\Laravel\Pages\Crud\IndexPage;

class MoonshineUserIndexPage extends IndexPage
{
    /**
     * @var class-string<DefaultListComponentContract>
     */
    protected string $component = CardsListComponent::class;
}
```

---

## Template Field for HasOne Relationship

Implement a HasOne relationship through the Template field for full control over rendering and saving.

```php
use MoonShine\UI\Fields\Template;
use MoonShine\UI\Components\Components;

protected function formFields(): iterable
{
    return [
        Template::make('Comment')
            ->changeFill(fn (Article $data) => $data->comment)
            ->changePreview(fn($data) => $data?->id ?? '-')
            ->fields(app(CommentResource::class)->getFormFields())
            ->changeRender(function (?Comment $data, Template $field) {
                $fields = $field->getPreparedFields();
                $fields->fill($data?->toArray() ?? []);

                return Components::make($fields);
            })
            ->onAfterApply(function (Article $item, array $value) {
                $item->comment()->updateOrCreate([
                    'id' => $value['id']
                ], $value);

                return $item;
            })
    ];
}
```

---

## Relationship Fields in Tabs

Separate relationship fields into tabs on the form page.

### Step 1: Create custom form page

```php
class ArticleResource extends ModelResource
{
    protected function pages(): array
    {
        return [
            IndexPage::class,
            ArticleFormPage::class,  // replaces FormPage
            DetailPage::class,
        ];
    }
}
```

### Step 2: Implement the tabbed form page

```php
namespace App\MoonShine\Pages\Article;

use App\Models\Comment;
use App\MoonShine\Resources\CommentResource;
use MoonShine\Laravel\Fields\Relationships\{HasMany, HasOne};
use MoonShine\Laravel\Pages\Crud\FormPage;
use MoonShine\Laravel\TypeCasts\ModelCaster;
use MoonShine\UI\Components\Table\TableBuilder;
use MoonShine\UI\Components\Tabs;
use MoonShine\UI\Components\Tabs\Tab;
use MoonShine\UI\Fields\{ID, Text};

final class ArticleFormPage extends FormPage
{
    protected function fields(): iterable
    {
        return [
            ID::make(),
            // ... other fields ...

            // IMPORTANT: Include relationship fields in fields() so MoonShine can find them
            $this->getCommentsField(),
            $this->getCommentField(),
        ];
    }

    private function getCommentsField(): HasMany
    {
        return HasMany::make('Comments', resource: CommentResource::class)
            ->fillData($this->getResource()->getItem())
            ->async()
            ->creatable();
    }

    private function getCommentField(): HasOne
    {
        return HasOne::make('Comment', resource: CommentResource::class)
            ->fillData($this->getResource()->getItem())
            ->async();
    }

    protected function mainLayer(): array
    {
        return [
            Tabs::make([
                Tab::make('Basics', parent::mainLayer()),
                Tab::make('Comments', [
                    $this->getResource()->getItem()
                        ? $this->getCommentsField()
                        : 'To add comments, save the article',
                ]),
                Tab::make('Comment', [
                    $this->getResource()->getItem()
                        ? $this->getCommentField()
                        : 'To add comments, save the article',
                ]),
                Tab::make('Table', [
                    TableBuilder::make()
                        ->fields([
                            ID::make(),
                            Text::make('Text')
                        ])
                        ->cast(new ModelCaster(Comment::class))
                        ->items($this->getResource()->getItem()?->comments ?? [])
                ]),
            ]),
        ];
    }

    // Nullify bottomLayer to avoid duplicating relationship fields
    protected function bottomLayer(): array
    {
        return [];
    }
}
```

Important rules:
1. Always include relationship fields in the `fields()` method so MoonShine can find them.
2. Do not create forms within forms (will cause conflicts).
3. Remember to fill fields with data and convert to the required type.

---

## Table Paginator

Add pagination to a standalone TableBuilder on a custom page.

```php
use MoonShine\Laravel\TypeCasts\PaginatorCaster;
use MoonShine\UI\Components\Table\TableBuilder;
use MoonShine\UI\Fields\Text;

protected function components(): iterable
{
    $posts = Post::query()->paginate(); // or ->simplePaginate() or ->cursorPaginate()

    $paginator = (new PaginatorCaster(
        $posts->appends(request()->except('page'))->toArray(),
        $posts->items()
    ))->cast();

    return [
        TableBuilder::make()
            ->fields([
                Text::make('Name')
            ])
            ->items($paginator)
    ];
}
```

---

## Multiple Selectors and Fragments

### Multiple selectors with JsonResponse

Send content to multiple selectors at once:

```php
use MoonShine\Crud\JsonResponse;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Components\Layout\Div;

public function multipleSelectors(): JsonResponse
{
    return JsonResponse::make()->html([
        '.selector1' => 'here 1',
        '.selector2' => 'here 2',
    ]);
}

protected function components(): iterable
{
    return [
        ActionButton::make('Test')->method('multipleSelectors', selector: [
            '.selector1',
            '.selector2',
        ]),

        Div::make([])->class('selector1'),
        Div::make([])->class('selector2'),
    ];
}
```

### Async update of multiple fragments

```php
use MoonShine\UI\Components\ActionButton;
use MoonShine\Crud\Components\Fragment;
use MoonShine\UI\Components\Layout\Div;

ActionButton::make('Fragments', $this->getRouter()->getEndpoints()->toPage($this, extra: [
    'fragment' => [
        '.selector1' => '_content1',
        '.selector2' => '_content2',
    ]
]))->async(selector: [
    '.selector1',
    '.selector2',
]),

Div::make([])->class('selector1'),
Div::make([])->class('selector2'),

Fragment::make([
    time(),
])->name('_content1'),

Fragment::make([
    time(),
])->name('_content2'),
```

---

## Async Remove on Click

### Image: Remove avatar on click

```php
use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;

protected function formFields(): iterable
{
    return [
        Image::make('Avatar')
            ->removable(attributes: [
                'data-async-url' => $this->getRouter()->getEndpoints()->method(
                    'removeAvatar',
                    params: ['resourceItem' => $this->getItemID()]
                ),
                '@click.prevent' => <<<'JS'
                    fetch($event.target.closest('button').dataset.asyncUrl)
                        .then(() => $event.target.closest('.x-removeable').remove())
                JS
            ]),
    ];
}

public function removeAvatar(CrudRequestContract $request): void
{
    $item = $request->getResource()?->getItem();

    if (is_null($item)) {
        return;
    }

    $item->update(['avatar' => null]);
}
```

### Json: Remove individual entries on click

```php
protected function formFields(): iterable
{
    return [
        Json::make('Data')->fields([
            Text::make('Title'),
        ])->removable(attributes: [
            'data-async-url' => $this->getActivePage()
                ? $this->getRouter()->getEndpoints()->method(
                    'removeJsonData',
                    params: ['resourceItem' => $this->getItemID()]
                )
                : null,
            '@click.prevent' => <<<'JS'
                fetch(`${$event.target.closest('a').dataset.asyncUrl}&index=${$event.target.closest('tr').rowIndex}`)
                    .then(() => remove())
            JS
        ]),
    ];
}

public function removeJsonData(CrudRequestContract $request): void
{
    $item = $request->getResource()?->getItem();
    $index = $request->integer('index') - 1;

    if (is_null($item)) {
        return;
    }

    $data = $item->data->toArray();
    unset($data[$index]);
    sort($data);

    $item->update(['data' => $data]);
}
```
