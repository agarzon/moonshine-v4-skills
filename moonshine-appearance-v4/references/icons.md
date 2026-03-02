# Icons Reference (MoonShine v4)

Heroicons integration, icon sets, custom icons, and icon component usage.

## Basics

MoonShine v4 uses [Heroicons](https://heroicons.com) as the default icon set. All entities that support the `icon()` method can use Heroicons or custom icon sets.

```php
icon(
    string $icon,
    bool $custom = false,
    ?string $path = null,
)
```

- `$icon` -- name of the icon or HTML (if custom mode is used).
- `$custom` -- enables custom mode (accepts raw SVG/HTML).
- `$path` -- path to the directory where Blade templates for icons are located.

## Icon sets

MoonShine v4 provides four Heroicon sets:

### Outline (default)

The default set. No prefix needed.

```php
->icon('academic-cap')
->icon('cog')
->icon('users')
->icon('home')
```

### Solid

Prefix with `s.` for solid icons.

```php
->icon('s.academic-cap')
->icon('s.cog')
->icon('s.users')
->icon('s.home')
```

### Mini

Prefix with `m.` for mini (20x20) icons.

```php
->icon('m.academic-cap')
->icon('m.cog')
->icon('m.users')
->icon('m.home')
```

### Compact

Prefix with `c.` for compact (16x16) icons.

```php
->icon('c.academic-cap')
->icon('c.cog')
->icon('c.users')
->icon('c.home')
```

## Using icons

### On menu items

```php
use MoonShine\MenuManager\MenuItem;

MenuItem::make(ArticleResource::class, icon: 'document-text')

// Or via method
MenuItem::make(ArticleResource::class)
    ->icon('document-text')
```

### On menu groups

```php
use MoonShine\MenuManager\MenuGroup;

MenuGroup::make('Content', [/* items */], icon: 'newspaper')

// Or via method
MenuGroup::make('Content', [/* items */])
    ->icon('newspaper')
```

### On resources via attribute

```php
use MoonShine\Support\Attributes\Icon;

#[Icon('users')]
class MoonShineUserResource extends ModelResource
{
    // The icon will appear in the menu automatically
}
```

### On pages via attribute

```php
use MoonShine\Support\Attributes\Icon;

#[Icon('home')]
class DashboardPage extends Page
{
    // ...
}
```

### On fields

```php
use MoonShine\UI\Fields\Text;

Text::make('Title')
    ->icon('pencil')
```

### On buttons

```php
use MoonShine\UI\Components\ActionButton;

ActionButton::make('Edit', '/edit')
    ->icon('pencil-square')
```

## Custom icons

### Raw HTML/SVG

Pass raw SVG or HTML as the icon with `custom: true`.

```php
->icon(
    svg('path-to-icon-pack')->toHtml(),
    custom: true
)
```

> The `svg()` function is from the `Blade Icons` package.

### Custom SVG as HTML string

```php
->icon(
    '<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M12 6v12m6-6H6"/></svg>',
    custom: true
)
```

### Custom icon directory

Point to a directory containing Blade templates for icons. Each icon is a `.blade.php` file containing SVG markup.

```php
->icon('cog', path: 'icons')
```

This looks for the Blade template at `resources/views/icons/cog.blade.php`.

#### Example icon template

```blade
{{-- resources/views/icons/cog.blade.php --}}
<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" {{ $attributes }}>
    <path stroke-linecap="round" stroke-linejoin="round" d="M9.594 3.94c.09-.542.56-.94 1.11-.94h2.593c.55 0 1.02.398 1.11.94l.213 1.281c.063.374.313.686.645.87.074.04.147.083.22.127.325.196.72.257 1.075.124l1.217-.456a1.125 1.125 0 0 1 1.37.49l1.296 2.247a1.125 1.125 0 0 1-.26 1.431l-1.003.827c-.293.241-.438.613-.43.992a7.723 7.723 0 0 1 0 .255c-.008.378.137.75.43.991l1.004.827c.424.35.534.955.26 1.43l-1.298 2.247a1.125 1.125 0 0 1-1.369.491l-1.217-.456c-.355-.133-.75-.072-1.076.124a6.47 6.47 0 0 1-.22.128c-.331.183-.581.495-.644.869l-.213 1.281c-.09.543-.56.94-1.11.94h-2.594c-.55 0-1.019-.398-1.11-.94l-.213-1.281c-.062-.374-.312-.686-.644-.87a6.52 6.52 0 0 1-.22-.127c-.325-.196-.72-.257-1.076-.124l-1.217.456a1.125 1.125 0 0 1-1.369-.49l-1.297-2.247a1.125 1.125 0 0 1 .26-1.431l1.004-.827c.292-.24.437-.613.43-.991a6.932 6.932 0 0 1 0-.255c.007-.38-.138-.751-.43-.992l-1.004-.827a1.125 1.125 0 0 1-.26-1.43l1.297-2.247a1.125 1.125 0 0 1 1.37-.491l1.216.456c.356.133.751.072 1.076-.124.072-.044.146-.086.22-.128.332-.183.582-.495.644-.869l.214-1.28Z"/>
    <path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z"/>
</svg>
```

### Using Blade Icons package

For large custom icon sets, consider using the [Blade Icons](https://github.com/blade-ui-kit/blade-icons) package.

```php
// After installing and configuring blade-icons
->icon(svg('heroicon-o-cog-6-tooth')->toHtml(), custom: true)
```

## Commonly used Heroicon names

### Navigation and layout

| Icon name | Description |
|---|---|
| `home` | Home/dashboard |
| `bars-3` | Hamburger menu |
| `x-mark` | Close/dismiss |
| `chevron-left` | Back navigation |
| `chevron-right` | Forward navigation |
| `chevron-down` | Dropdown indicator |
| `chevron-up` | Collapse indicator |
| `arrow-left` | Left arrow |
| `arrow-right` | Right arrow |
| `arrows-pointing-out` | Fullscreen |

### Content and documents

| Icon name | Description |
|---|---|
| `document-text` | Article/document |
| `newspaper` | News/blog |
| `book-open` | Documentation |
| `folder` | Folder/category |
| `photo` | Image |
| `film` | Video |
| `musical-note` | Audio |
| `paper-clip` | Attachment |
| `link` | Link |
| `tag` | Tag/category |
| `bookmark` | Bookmark/save |

### Actions

| Icon name | Description |
|---|---|
| `plus` | Add/create |
| `pencil` | Edit |
| `pencil-square` | Edit (boxed) |
| `trash` | Delete |
| `eye` | View/preview |
| `eye-slash` | Hide |
| `magnifying-glass` | Search |
| `funnel` | Filter |
| `arrow-down-tray` | Download |
| `arrow-up-tray` | Upload |
| `clipboard` | Copy |
| `check` | Confirm/done |
| `x-mark` | Cancel/remove |

### Users and communication

| Icon name | Description |
|---|---|
| `user` | Single user |
| `users` | User group |
| `user-plus` | Add user |
| `user-circle` | User avatar |
| `chat-bubble-left` | Comment/message |
| `chat-bubble-left-right` | Conversation |
| `envelope` | Email |
| `bell` | Notification |
| `phone` | Phone |

### System and settings

| Icon name | Description |
|---|---|
| `cog` | Settings (outline) |
| `cog-6-tooth` | Settings (detailed) |
| `wrench` | Configuration |
| `adjustments-horizontal` | Adjustments |
| `shield-check` | Security |
| `lock-closed` | Locked/private |
| `lock-open` | Unlocked/public |
| `key` | Authentication |
| `server` | Server/database |
| `circle-stack` | Database |

### Status and indicators

| Icon name | Description |
|---|---|
| `check-circle` | Success |
| `exclamation-triangle` | Warning |
| `x-circle` | Error |
| `information-circle` | Info |
| `clock` | Pending/time |
| `arrow-path` | Refresh/sync |
| `signal` | Active/online |
| `signal-slash` | Inactive/offline |

### Commerce

| Icon name | Description |
|---|---|
| `shopping-cart` | Cart |
| `shopping-bag` | Order |
| `credit-card` | Payment |
| `currency-dollar` | Finance |
| `receipt-percent` | Discount |
| `gift` | Promotion |
| `truck` | Shipping |

### Data and charts

| Icon name | Description |
|---|---|
| `chart-bar` | Bar chart |
| `chart-pie` | Pie chart |
| `presentation-chart-line` | Line chart |
| `table-cells` | Table/grid |
| `squares-2x2` | Grid view (default when no icon set) |
| `list-bullet` | List view |
| `calendar` | Calendar |
| `calculator` | Calculator |

## Icon in onlyIcon mode

When `onlyIcon()` is used on a menu item, only the icon displays and a tooltip shows the label text on hover.

```php
MenuItem::make(DashboardPage::class, 'Dashboard', 'home')
    ->onlyIcon()
```

If no icon is set, the `squares-2x2` icon is used as the default fallback.
