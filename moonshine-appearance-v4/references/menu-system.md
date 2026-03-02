# Menu System Reference (MoonShine v4)

Full menu API including MenuItem, MenuGroup, MenuDivider, autoload, attributes, and authorization.

## CRITICAL v4 change: MenuItem parameter order

In v4 the `$filler` is the FIRST parameter and `$label` is the SECOND (optional) parameter. The label is automatically derived from the filler's `getTitle()` method when omitted.

```php
MenuItem::make(
    Closure|MenuFillerContract|string $filler,  // FIRST: resource class, page class, URL, or closure
    Closure|string $label = null,               // SECOND: optional label
    string $icon = null,                        // icon name
    Closure|bool $blank = false                 // open in new tab
)
```

Any class implementing `MenuFillerContract` can be passed as `$filler`. By default, `ModelResource` and `Page` implement this interface.

## Menu basics

The menu is declared in the layout's `menu()` method.

```php
namespace App\MoonShine\Layouts;

use MoonShine\Laravel\Layouts\AppLayout;
use MoonShine\MenuManager\MenuItem;
use MoonShine\MenuManager\MenuGroup;
use MoonShine\MenuManager\MenuDivider;

final class MoonShineLayout extends AppLayout
{
    protected function menu(): array
    {
        return [
            MenuItem::make(MoonShineUserResource::class),
            MenuItem::make(fn() => route('home'), 'Home'),
            MenuItem::make('https://moonshine-laravel.com/docs', 'Docs', blank: true),
            MenuItem::make('https://laravel.com/docs', 'Laravel Docs', blank: true),
        ];
    }
}
```

> If the menu is created for ModelResource or CrudResource, the first page declared in the `pages()` method is used for the menu item URL.

## MenuItem examples

### Resource-based (label auto-derived)

```php
MenuItem::make(ArticleResource::class)
```

### Resource with custom label

```php
MenuItem::make(ArticleResource::class, 'Articles')
```

### Resource with icon

```php
MenuItem::make(ArticleResource::class, icon: 'document-text')
```

### Page-based

```php
MenuItem::make(DashboardPage::class, 'Dashboard', 'home')
```

### URL-based

```php
MenuItem::make('https://example.com', 'External Link', blank: true)
```

### Closure-based URL

```php
MenuItem::make(fn() => route('admin.reports'), 'Reports')
```

## MenuGroup

Groups menu items together. Supports nesting for multi-level menus.

```php
MenuGroup::make(
    Closure|string $label,
    iterable $items,
    string|null $icon = null,
)
```

### Basic group

```php
MenuGroup::make('System', [
    MenuItem::make(MoonShineUserResource::class),
    MenuItem::make(MoonShineUserRoleResource::class),
])
```

### Group with icon

```php
MenuGroup::make('Content', [
    MenuItem::make(ArticleResource::class),
    MenuItem::make(CategoryResource::class),
])->icon('newspaper')
```

### Using setItems()

```php
MenuGroup::make('System')->setItems([
    MenuItem::make(MoonShineUserResource::class),
    MenuItem::make(MoonShineUserRoleResource::class),
])
```

### Nested groups (multi-level menu)

```php
MenuGroup::make('Administration', [
    MenuGroup::make('Users', [
        MenuItem::make(MoonShineUserResource::class),
        MenuItem::make(MoonShineUserRoleResource::class),
    ])->icon('users'),
    MenuGroup::make('Settings', [
        MenuItem::make(SettingsPage::class),
    ])->icon('cog'),
])->icon('shield-check')
```

## MenuDivider

Visual separator between menu items. Accepts an optional label.

```php
MenuDivider::make(Closure|string $label = '')
```

```php
MenuItem::make(MoonShineUserResource::class),
MenuDivider::make(),
MenuItem::make(MoonShineUserRoleResource::class),

// With label
MenuDivider::make('Section Title'),
```

## Icons

Icons can be set via parameter, method, or attribute.

### Via parameter

```php
MenuItem::make(MoonShineUserResource::class, icon: 'users')
```

### Via method

```php
MenuItem::make(MoonShineUserResource::class)
    ->icon('users')

// Custom SVG icon
MenuItem::make(ArticleResource::class)
    ->icon(svg('path-to-icon-pack')->toHtml(), custom: true)

// Custom icon directory
MenuGroup::make('System', [/* ... */])
    ->icon('cog', path: 'icons')
```

### Via attribute on resource/page

```php
use MoonShine\Support\Attributes\Icon;

#[Icon('users')]
class MoonShineUserResource extends ModelResource
{
    // ...
}
```

## Only icons mode

Display menu items with only icons (no text labels). Tooltip shown on hover.

### Individual items

```php
MenuItem::make(DashboardPage::class, 'Dashboard', 'home')
    ->onlyIcon()

// Conditional
MenuItem::make(DashboardPage::class, 'Dashboard', 'home')
    ->onlyIcon(fn() => $someCondition)
```

### Entire menu via autoload

```php
protected function menu(): array
{
    return $this->autoloadMenu(onlyIcons: true);
}
```

> Items without an icon set will use the `squares-2x2` icon by default.

## Badge

Add count badges to menu items.

```php
MenuItem::make(CommentResource::class)
    ->badge(fn() => Comment::count())

// Static badge
MenuItem::make(ArticleResource::class)
    ->badge('new')
```

## Translation

Pass translation keys as labels and use `translatable()`.

```php
// Using translation file key
MenuItem::make(CommentResource::class, 'menu.Comments')
    ->translatable()

// With translation namespace
MenuItem::make(CommentResource::class, 'Comments')
    ->translatable('menu')
```

```php
// lang/en/menu.php
return [
    'Comments' => 'Comments',
];
```

### Translatable badges

```php
MenuItem::make(CommentResource::class)
    ->badge(fn() => __('menu.badge.new'))
```

### Translatable groups

```php
MenuGroup::make('menu.system', [/* ... */])
    ->translatable()
```

## Open in new tab

### Via parameter

```php
MenuItem::make('https://moonshine-laravel.com/docs', 'MoonShine Docs', 'arrow-up', true)
MenuItem::make('https://laravel.com/docs', 'Laravel Docs', blank: fn() => true)
```

### Via method

```php
MenuItem::make('https://example.com', 'External')
    ->blank()

// Conditional
MenuItem::make('https://example.com', 'External')
    ->blank(fn() => $shouldOpenInNewTab)
```

## Conditional display (canSee)

Show or hide menu items based on conditions.

```php
MenuGroup::make('System', [
    MenuItem::make(MoonShineUserResource::class),
    MenuDivider::make()
        ->canSee(fn() => true),
    MenuItem::make(MoonShineUserRoleResource::class)
        ->canSee(fn() => false)
])
    ->canSee(static fn(): bool => request()->user('moonshine')?->id === 1)
```

## Active item

Force a menu item active with `whenActive()`.

```php
MenuItem::make('/endpoint', 'Label')
    ->whenActive(
        fn() => request()->fullUrlIs('*admin/endpoint/*')
    )
```

## Custom attributes

```php
MenuGroup::make('System')->setItems([
    MenuItem::make(MoonShineUserResource::class),
    MenuItem::make(MoonShineUserRoleResource::class)
        ->customAttributes(['class' => 'group-li-custom-class'])
])
    ->setAttribute('data-id', '123')
    ->class('group-li-custom-class')
```

## Change button

Menu items are ActionButton components. Modify them via `changeButton()`.

```php
use MoonShine\UI\Components\ActionButton;

MenuItem::make('/endpoint', 'Label')
    ->changeButton(static fn(ActionButton $button) => $button->class('new-item'))
```

> WARNING: Some ActionButton parameters (url, badge, icon) are overridden by the system. Use the corresponding MenuItem methods instead.

## Custom view

Override the Blade view for individual items or groups.

```php
MenuGroup::make('Group', [
    MenuItem::make('/endpoint', 'Label')
        ->customView('admin.custom-menu-item'),
])
    ->customView('admin.custom-menu-group')
```

## Menu autoload

Replace manual menu arrays with automatic discovery from registered resources and pages.

> Requires autoloading of resources and pages to be enabled.

```php
protected function menu(): array
{
    return $this->autoloadMenu();
}
```

### Autoload attributes

#### SkipMenu -- exclude from menu

```php
use MoonShine\MenuManager\Attributes\SkipMenu;

#[SkipMenu]
class ProfilePage extends Page {}
```

#### Group -- assign to menu group

```php
use MoonShine\MenuManager\Attributes\Group;

#[Group('moonshine::ui.profile', 'users', translatable: true)]
class ProfilePage extends Page {}
```

#### Order -- set display order

```php
use MoonShine\MenuManager\Attributes\Order;

#[Order(1)]
class ArticleResource extends ModelResource {}
```

#### CanSee -- conditional display

```php
use MoonShine\MenuManager\Attributes\CanSee;

#[CanSee(method: 'someMethod')]
class ArticleResource extends ModelResource
{
    public function someMethod(): bool
    {
        return auth()->user()->isAdmin();
    }
}
```

#### Badge -- add badge to menu item

```php
use MoonShine\MenuManager\Attributes\Badge;

#[Badge('new')]
class ArticleResource extends ModelResource {}
```

## Top menu

By default, MoonShine has a top menu component. Use it instead of or alongside the Sidebar.

### Replace Sidebar with TopBar

Set `$sidebar = false` and `$topBar = true` in your layout, or override `build()` to replace `getSidebarComponent()` with `getTopBarComponent()`.

```php
final class MoonShineLayout extends AppLayout
{
    protected bool $sidebar = false;
    protected bool $topBar = true;
}
```

### Both Sidebar and TopBar

When using both, TopBar must come first in the component tree.

```php
final class MoonShineLayout extends AppLayout
{
    protected bool $sidebar = true;
    protected bool $topBar = true;
}
```

## Complete menu example

```php
namespace App\MoonShine\Layouts;

use App\MoonShine\Resources\ArticleResource;
use App\MoonShine\Resources\CategoryResource;
use App\MoonShine\Resources\CommentResource;
use App\MoonShine\Pages\DashboardPage;
use MoonShine\Laravel\Layouts\AppLayout;
use MoonShine\MenuManager\MenuItem;
use MoonShine\MenuManager\MenuGroup;
use MoonShine\MenuManager\MenuDivider;

final class MoonShineLayout extends AppLayout
{
    protected function menu(): array
    {
        return [
            MenuItem::make(DashboardPage::class, 'Dashboard', 'home'),

            MenuDivider::make('Content'),

            MenuGroup::make('Blog', [
                MenuItem::make(ArticleResource::class)
                    ->icon('document-text')
                    ->badge(fn() => \App\Models\Article::where('status', 'draft')->count()),
                MenuItem::make(CategoryResource::class)
                    ->icon('tag'),
                MenuItem::make(CommentResource::class)
                    ->icon('chat-bubble-left')
                    ->badge(fn() => \App\Models\Comment::where('approved', false)->count()),
            ])->icon('newspaper'),

            MenuDivider::make(),

            MenuItem::make('https://docs.example.com', 'Documentation', 'book-open')
                ->blank()
                ->canSee(fn() => auth()->user()->isAdmin()),
        ];
    }
}
```
