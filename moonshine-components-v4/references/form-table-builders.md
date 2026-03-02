# FormBuilder, TableBuilder, ActionButton, ActionGroup, FieldsGroup — Full API

## FormBuilder

**Namespace:** `MoonShine\UI\Components\FormBuilder`

### make()

```php
FormBuilder::make(
    string $action = '',
    FormMethod $method = FormMethod::POST,
    FieldsContract|iterable $fields = [],
    mixed $values = [],
)
```

### fields()

```php
FormBuilder::make('/crud/update')
    ->fields([
        Heading::make('Title'),
        Text::make('Text'),
    ])
```

### Form Name

```php
FormBuilder::make('/crud/update')
    ->name('main-form')
```

Required for event targeting. The `async()` method must come after `name()`.

### fill() / fillCast()

```php
// Fill with array
FormBuilder::make('/crud/update')
    ->fields([Text::make('Text')])
    ->fill(['text' => 'value'])

// Fill and cast to model
use MoonShine\Laravel\TypeCasts\ModelCaster;

FormBuilder::make('/crud/update')
    ->fields([Text::make('Text')])
    ->fillCast(User::query()->first(), new ModelCaster(User::class))
```

### cast()

```php
FormBuilder::make('/crud/update')
    ->fields([Text::make('Text')])
    ->values(['text' => 'value'])
    ->cast(new ModelCaster(User::class))
```

### submit() / hideSubmit()

```php
FormBuilder::make('/crud/update')
    ->submit(label: 'Click me', attributes: ['class' => 'btn-primary'])

FormBuilder::make('/crud/update')
    ->hideSubmit()
```

### buttons()

```php
FormBuilder::make('/crud/update')
    ->buttons([
        ActionButton::make('Delete', route('name.delete'))
    ])
```

### action() / method()

```php
FormBuilder::make()
    ->action('/crud/update')
    ->method(FormMethod::POST)
```

### Custom Attributes

```php
FormBuilder::make()->customAttributes(['class' => 'custom-form'])
```

### async()

```php
FormBuilder::make('/crud/update')
    ->name('main-form')
    ->async(
        url: null,            // defaults to action URL
        events: [             // events after success
            AlpineJs::event(JsEvent::TABLE_UPDATED, 'crud-table'),
            AlpineJs::event(JsEvent::FORM_RESET, 'main-form'),
        ],
        callback: null,       // JS callback
    )
```

### asyncMethod() / download()

Call a page/resource method asynchronously. Methods must be marked with `#[AsyncMethod]`.

```php
FormBuilder::make()
    ->asyncMethod('updateSomething')
    ->fields([Text::make('Title')])

// For file download
FormBuilder::make()
    ->asyncMethod('zip')
    ->download()
```

Method examples:

```php
use MoonShine\Support\Attributes\AsyncMethod;

#[AsyncMethod]
public function updateSomething(CrudRequestContract $request, JsonResponse $response): JsonResponse
{
    return $response->toast('My message', ToastType::SUCCESS);
}

#[AsyncMethod]
public function updateSomething(CrudRequestContract $request, JsonResponse $response): JsonResponse
{
    return $response->redirect('/');
}
```

### Reactivity

```php
FormBuilder::make()
    ->reactiveUrl(
        fn(FormBuilder $form) => $form->getCore()->getRouter()->getEndpoints()->reactive($page, $resource, $extra)
    )
```

### Field Values via Selectors

```php
FormBuilder::make()
    ->asyncMethod('formAction')
    ->asyncSelector(['.some-class1', '.some-class2'])
    ->fields([
        Div::make([])->class('some-class1'),
        Div::make([])->class('some-class2'),
    ])
```

### Validation

```php
// Disable errors above form
FormBuilder::make('/crud/update')->errorsAbove(false)

// Disable error toast
FormBuilder::make('/crud/update')->withoutErrorToast()

// Precognitive validation
FormBuilder::make('/crud/update')->precognitive()

// Multiple forms - use matching errorBag names
FormBuilder::make(route('forms.one'))->name('formOne')
// In FormRequest: protected $errorBag = 'formOne';
```

### apply()

```php
$form->apply(
    fn(Model $item) => $item->save(),
    before: fn(Model $item) => $item,
    after: fn(Model $item) => $item,
    throw: true
);
```

### dispatchEvent()

```php
FormBuilder::make()
    ->dispatchEvent(
        AlpineJs::event(JsEvent::OFF_CANVAS_TOGGLED, 'default'),
        exclude: ['text', 'description'],
        withoutPayload: false
    )
```

---

## TableBuilder

**Namespace:** `MoonShine\UI\Components\Table\TableBuilder`

### make()

```php
TableBuilder::make(iterable $fields = [], iterable $items = [])
```

### fields()

```php
TableBuilder::make()
    ->fields([
        ID::make()->sortable(),
        Text::make('Title')
            ->customWrapperAttributes(['class' => 'my-class']),
    ])
```

### items()

```php
TableBuilder::make()->items($this->getCollection())
```

### paginator()

```php
use MoonShine\Laravel\TypeCasts\ModelCaster;

TableBuilder::make()
    ->paginator(
        (new ModelCaster(Article::class))
            ->paginatorCast(Article::query()->paginate())
    )

// Simple pagination style
TableBuilder::make()->simple()
```

### cast() / castKeyName()

```php
TableBuilder::make()->cast(new ModelCaster(User::class))

// Without cast, specify key name for bulk operations
TableBuilder::make()->castKeyName('id')
```

### buttons() / stickyButtons()

```php
TableBuilder::make()
    ->buttons([
        ActionButton::make('Delete', fn() => route('name.delete')),
        ActionButton::make('Edit', fn() => route('name.edit'))->showInDropdown(),
        ActionButton::make('Mass Delete', fn() => route('name.mass_delete'))->bulk(),
    ])
    ->stickyButtons()
```

### vertical()

```php
// Basic vertical (detail page)
TableBuilder::make()->fields($fields)->items([$item])->vertical()

// With column span customization
$component->vertical(title: 2, value: 10)

// With closure
$component->vertical(
    title: fn(FieldContract $field, Column $default, TableBuilder $ctx) => $default->columnSpan(2),
    value: fn(FieldContract $field, Column $default, TableBuilder $ctx) => $default->columnSpan(10),
)
```

### editable() / preview()

```php
TableBuilder::make()->editable()   // all fields in form mode
TableBuilder::make()->preview()    // no buttons, no sorting
```

### withNotFound()

```php
TableBuilder::make()->withNotFound()
```

### creatable()

```php
TableBuilder::make()
    ->creatable(
        reindex: true,
        limit: 5,
        label: 'Add',
        icon: 'plus',
        attributes: ['class' => 'my-class'],
        button: null  // or custom ActionButton
    )
```

### reindex() / withoutKey()

```php
TableBuilder::make()->reindex()
TableBuilder::make()->withoutKey()  // use ordinal indexes instead of unique keys
```

### reorderable() (drag and drop)

```php
TableBuilder::make()
    ->reorderable(url: '/reorder-url', key: 'id', group: 'group-name')
```

### sticky()

```php
TableBuilder::make()->sticky()
```

### columnSelection()

```php
TableBuilder::make()
    ->fields([
        Text::make('Title')->columnSelection(false),  // always visible
        Text::make('Text'),
    ])
    ->columnSelection(prefix: fn() => auth()->id())
```

### searchable()

```php
TableBuilder::make()->searchable()
```

### clickAction()

```php
use MoonShine\Support\Enums\ClickAction;

TableBuilder::make()->clickAction(ClickAction::EDIT)
TableBuilder::make()->clickAction(ClickAction::DETAIL, '.detail-button')
// ClickAction::SELECT, ClickAction::EDIT, ClickAction::DETAIL
```

### async() / lazy() / whenAsync()

```php
TableBuilder::make()
    ->name('crud')
    ->async(
        url: null,
        events: [AlpineJs::event(JsEvent::TOAST, params: ['text' => 'Success'])],
        callback: null,
    )
    ->lazy()
    ->whenAsync(fn(TableBuilder $t) => $t->items(Http::get('/api/data')->json()))
    ->withMorphLoad()  // preserve Alpine.js state during updates
```

### Row Customization

```php
TableBuilder::make()
    ->rows(fn(TableRowsContract $default) => $default->pushRow(
        TableCells::make()->pushCell('td content')
    ))
    ->headRows(fn(TableRowContract $default) => TableRows::make([$default])->pushRow(
        TableCells::make()->pushCell('th content')
    ))
    ->footRows(fn(?TableRowContract $default) => TableRows::make([$default])->pushRow(
        TableCells::make()->pushCell('footer content')
    ))
```

### Attribute Configuration

```php
TableBuilder::make()
    ->trAttributes(fn(?DataWrapperContract $data, int $row) => ['class' => $row % 2 ? 'bg-gray-100' : ''])
    ->tdAttributes(fn(?DataWrapperContract $data, int $row, int $cell) => ['class' => $cell === 0 ? 'font-bold' : ''])
    ->headAttributes(['class' => 'bg-blue-500 text-white'])
    ->bodyAttributes(['class' => 'text-sm'])
    ->footAttributes(['class' => 'bg-gray-200'])
```

### Slots

```php
TableBuilder::make()
    ->topLeft(fn() => [])
    ->topRight(fn() => [Div::make([/* ... */])])
```

### Filters via FormBuilder

```php
FormBuilder::make()
    ->name('dashboard-form')
    ->fields([Date::make('Date')])
    ->dispatchEvent([AlpineJs::event(JsEvent::TABLE_UPDATED, 'dashboard-table')]),

TableBuilder::make()
    ->name('dashboard-table')
    ->withFilters('dashboard-form')
    ->async()
    ->fields([Text::make('Title')->sortable()])
    ->items([['title' => fake()->word()]])
```

### Other Options

```php
TableBuilder::make()->pushState()           // save state in URL
TableBuilder::make()->queryParamPrefix('users_')  // prefix for pagination params
TableBuilder::make()->skeleton(true)        // skeleton loading mode
TableBuilder::make()->loader(true)          // spinner loading mode
```

---

## ActionButton

**Namespace:** `MoonShine\UI\Components\ActionButton`

### make()

```php
ActionButton::make(
    Closure|string $label,
    Closure|string $url = '#',
    ?DataWrapperContract $data = null,
)
```

### blank() / icon() / color / badge()

```php
ActionButton::make('Link', '/url')->blank()           // target="_blank"
ActionButton::make('Edit')->icon('pencil')
ActionButton::make('Save')->primary()                  // also: secondary(), success(), warning(), error(), info()
ActionButton::make('Comments')->badge(fn() => Comment::count())
```

### onClick()

```php
ActionButton::make('Alert')->onClick(fn() => "alert('Hello')", 'prevent')
```

### inModal()

```php
ActionButton::make('Open')
    ->inModal(
        title: 'Modal Title',
        content: 'Modal Content',
        name: 'my-modal',
        builder: fn(Modal $modal, ActionButton $ctx) => $modal,
        components: [],
    )

// Unique name per item (in tables)
ActionButton::make('Delete')
    ->inModal(
        name: fn(mixed $item, ActionButtonContract $ctx) => "delete-{$ctx->getData()?->getKey()}"
    )
```

### toggleModal() / openModal()

```php
ActionButton::make('Show Modal')->toggleModal('my-modal')
```

### withConfirm()

```php
ActionButton::make('Delete', '/delete')
    ->withConfirm(
        title: 'Confirmation',
        content: 'Are you sure?',
        button: 'Confirm',
        fields: null,
        method: HttpMethod::POST,
        formBuilder: null,
        modalBuilder: null,
        name: 'confirm-modal',
    )
```

### inOffCanvas()

```php
ActionButton::make('Filters')
    ->inOffCanvas(
        title: fn() => 'Panel Title',
        content: fn() => 'Content',
        name: false,
        builder: fn(OffCanvas $oc, ActionButton $ctx) => $oc->left(),
        components: [],
    )
```

### toggleOffCanvas()

```php
ActionButton::make('Show Panel')->toggleOffCanvas('my-canvas')
```

### async()

```php
ActionButton::make('Save', '/endpoint')
    ->async(
        method: HttpMethod::GET,
        selector: '#my-selector',
        events: [AlpineJs::event(JsEvent::TABLE_UPDATED, 'my-table')],
        callback: AsyncCallback::with(responseHandler: 'myFunction'),
    )
    ->withoutLoading()  // disable loading indicator
```

### method()

```php
ActionButton::make('Process')
    ->method(
        'updateSomething',
        params: fn(Model $item) => ['resourceItem' => $item->getKey()],
        message: 'Success!',
        selector: null,
        events: [],
        callback: null,
        page: null,
        resource: null,
    )

// With download
ActionButton::make('ZIP')->method('zip')->download()

// With selector params
ActionButton::make('Save')
    ->method('updateSomething')
    ->withSelectorsParams(['alias' => '[data-column="title"]', 'slug' => '#slug'])
```

### Other Methods

```php
// Dispatch JS event
ActionButton::make('Refresh')
    ->dispatchEvent(AlpineJs::event(JsEvent::TABLE_UPDATED, 'index-table'))

// Bulk action
ActionButton::make('Mass Delete', '/mass-delete')->bulk()

// Keyboard shortcut
ActionButton::make('Save')->hotKeys(['shift', 's'], withBadge: true)
```

---

## ActionGroup

```php
ActionGroup::make([
    ActionButton::make('Btn 1')->showInLine(),
    ActionButton::make('Btn 2')->showInDropdown(),
])->fill($data)
```

## FieldsGroup

```php
FieldsGroup::make([Text::make('Title'), Email::make('Email')])
    ->fill($raw, $casted, $index)
    ->previewMode()
    ->withoutWrappers()
```
