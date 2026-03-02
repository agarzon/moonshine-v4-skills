# Layouts Reference (MoonShine v4)

Full layout API, component control, slots, overridable methods, and Blade integration.

## Layout hierarchy

```
BaseLayout (abstract)
  -> AppLayout (standard sidebar layout -- the only layout you extend)
  -> BlankLayout (bare bones -- Head + Body only)
  -> LoginLayout (authentication page)
```

> CompactLayout has been removed in v4. Use `$contentSimpled = true` on AppLayout instead.

## AppLayout structure

When MoonShine is installed, the default layout is published at `app/MoonShine/Layouts/MoonShineLayout.php` and registered in `config/moonshine.php`.

```php
namespace App\MoonShine\Layouts;

use MoonShine\Laravel\Layouts\AppLayout;
use MoonShine\UI\Components\Layout\Layout;

final class MoonShineLayout extends AppLayout
{
    public function build(): Layout
    {
        return parent::build();
    }
}
```

## Component control properties (NEW in v4)

Control layout components through protected boolean properties without overriding `build()`.

### Sidebar

Enabled by default. Disable by setting `$sidebar = false`.

```php
use MoonShine\Laravel\Layouts\AppLayout;

final class MoonShineLayout extends AppLayout
{
    protected bool $sidebar = false;
}
```

### TopBar

Disabled by default. Enable by setting `$topBar = true`.

```php
use MoonShine\Laravel\Layouts\AppLayout;

final class MoonShineLayout extends AppLayout
{
    protected bool $topBar = true;
}
```

> WARNING: If you use both Sidebar and TopBar, TopBar must come first in the `build()` method.

### BottomBar

Disabled by default. Enable by setting `$bottomBar = true`. The BottomBar component automatically displays the top menu inside it.

```php
use MoonShine\Laravel\Layouts\AppLayout;

final class MoonShineLayout extends AppLayout
{
    protected bool $bottomBar = true;
}
```

### Mobile mode

A special mode that optimizes the interface for mobile devices.

```php
use MoonShine\Laravel\Layouts\AppLayout;

final class MoonShineLayout extends AppLayout
{
    protected bool $mobileMode = true;
}
```

When mobile mode is enabled:
- Compact paddings and element sizes are applied.
- The Burger component is hidden.
- BottomBar with the top menu is automatically enabled.

### Content layout properties

```php
// Remove border and background from content block
protected bool $contentSimpled = true; // default false

// Center content in a fixed-width container
protected bool $contentCentered = true; // default false
```

## Overridable component methods

Override individual sections without rewriting `build()`.

### Head component

```php
protected function getHeadComponent(bool $withAssetsFragment = true): Head
{
    return Head::make([
        // custom head content
    ]);
}
```

### Logo component

```php
protected function getLogoComponent(): Logo
{
    return Logo::make(
        $this->getHomeUrl(),
        $this->getLogo(),
        $this->getLogo(small: true),
    );
}
```

### Sidebar component

```php
protected function getSidebarComponent(): Sidebar
{
    return Sidebar::make([
        // custom sidebar content
    ]);
}
```

### Header component

```php
protected function getHeaderComponent(): Header
{
    return Header::make([
        // custom header content
    ]);
}
```

### TopBar component

```php
protected function getTopBarComponent(): Topbar
{
    return Topbar::make([
        // custom topbar content
    ]);
}
```

### Footer component

```php
protected function getFooterComponent(): Footer
{
    return Footer::make([
        // custom footer content
    ]);
}
```

### Profile component

```php
// NOTE: $withBorder parameter removed in v4
protected function getProfileComponent(): Profile
{
    return Profile::make();
}
```

### Content components

```php
protected function getContentComponents(): array
{
    return [
        // components rendered inside Content::make()
    ];
}
```

### Logo path

```php
protected function getLogo(bool $small = false): string
{
    return $small ? '/images/logo-small.png' : '/images/logo.png';
}
```

### Home URL

```php
protected function getHomeUrl(): string
{
    return route('moonshine.index');
}
```

### Footer customization

```php
protected function getFooterMenu(): array
{
    return [
        'https://example.com' => 'Custom link',
    ];
}

protected function getFooterCopyright(): string
{
    return sprintf('&copy; %d My Company', now()->year);
}
```

## Layout slots

Inject components into Sidebar or TopBar without overriding `build()`.

```php
protected function sidebarSlot(): array
{
    return [
        Search::make()->enabled(),
    ];
}

protected function sidebarTopSlot(): array
{
    return [
        Notifications::make(),
    ];
}

protected function topBarSlot(): array
{
    return [
        // custom components in top bar
    ];
}
```

## Creating a custom template

```shell
php artisan moonshine:layout MyLayout
```

Creates `app/MoonShine/Layouts/MyLayout.php`.

## Changing the page template

Assign a different layout to specific pages.

### Via property

```php
use App\MoonShine\Layouts\MyLayout;
use MoonShine\Laravel\Pages\Page;

class CustomPage extends Page
{
    protected ?string $layout = MyLayout::class;
}
```

### Via attribute

```php
use MoonShine\Core\Attributes\Layout;

#[Layout(MyLayout::class)]
class CustomPage extends Page {}
```

## Assets in layout

Each template can define its own styles and scripts.

```php
use MoonShine\AssetManager\Css;
use MoonShine\AssetManager\Js;
use MoonShine\AssetManager\InlineCss;

protected function assets(): array
{
    return [
        ...parent::assets(),
        Css::make('/vendor/moonshine/assets/minimalistic.css')->defer(),
        Js::make('/js/custom.js')->defer(),
        InlineCss::make(':root { --spacing: 0.15rem; }'),
    ];
}
```

### Vite integration

```php
use Illuminate\Support\Facades\Vite;
use MoonShine\AssetManager\Css;
use MoonShine\AssetManager\Js;

protected function assets(): array
{
    return [
        $this->getMainThemeJs(),
        Css::make(Vite::asset('resources/css/app.css')),
        Js::make(Vite::asset('resources/js/app.js')),
    ];
}
```

## Favicons

```php
protected function getFaviconComponent(): Favicon
{
    return parent::getFaviconComponent()->customAssets([
        'apple-touch' => '/favicon/apple-touch-icon.png',
        '32' => '/favicon/favicon-32x32.png',
        '16' => '/favicon/favicon-16x16.png',
        'safari-pinned-tab' => '/favicon/safari-pinned-tab.svg',
        'web-manifest' => '/favicon/site.webmanifest',
    ]);
}
```

## Menu in layout

```php
use MoonShine\MenuManager\MenuItem;

protected function menu(): array
{
    return [
        ...parent::menu(),
        MenuItem::make(ArticleResource::class),
    ];
}
```

## Themes

### Dark mode (always dark)

```php
protected function isAlwaysDark(): bool
{
    return true;
}
```

### Disable theme switcher (light only)

```php
protected function hasThemes(): bool
{
    return false;
}
```

### Force dark mode on custom components

Sidebar, TopBar, and MobileBar use dark colors by default. Force dark mode on custom additions:

```php
$this->getSidebarComponent()->class('dark'),
$this->getTopBarComponent()->class('dark'),

MobileBar::make([
    // ...
])->class('dark'),
```

## Colors in layout

Set a palette via property or override `colors()` for full control.

### Via palette property

```php
use App\MoonShine\Palettes\CorporatePalette;

final class MoonShineLayout extends AppLayout
{
    protected ?string $palette = CorporatePalette::class;
}
```

### Via colors() method

```php
use MoonShine\Contracts\ColorManager\ColorManagerContract;

protected function colors(ColorManagerContract $colorManager): void
{
    $colorManager
        ->primary('oklch(65% 0.18 264)')
        ->secondary('oklch(70% 0.14 230)')
        ->successBg('oklch(63.9% 0.218 142.495)')
        ->errorBg('oklch(58.9% 0.214 26.855)');

    // Dark theme overrides
    $colorManager
        ->set('body', '0.2 0.0168 274.32', dark: true)
        ->theme([
            'body' => '1 0 0',
            'stroke' => '1 0 0 / 10%',
            900 => '0.39 0.025 274.32',
        ], dark: true);
}
```

> Layout `colors()` loads after ServiceProvider and takes precedence.

## Fragments

Layout parts are wrapped in Fragment components for AJAX updates without page reload.

Available fragments:
- `sidebar-top` -- logo and theme switch
- `sidebar-content` -- side menu content
- `topbar-logo` -- logo in top menu
- `topbar-menu` -- top menu items
- `topbar-actions` -- action block in top menu
- `assets` -- styles and scripts

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

public function saveElement(CrudRequestContract $request): JsonResponse
{
    // ... save logic ...
    return JsonResponse::make()->events([
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'sidebar-top'),
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'sidebar-content'),
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'topbar-logo'),
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'topbar-menu'),
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'topbar-actions'),
        AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'assets'),
    ]);
}
```

## Blade layout

MoonShine v4 supports pure Blade templates using `<x-moonshine::layout.*>` components.

```blade
<x-moonshine::layout>
    <x-moonshine::layout.html :with-alpine-js="true" :with-themes="true">
        <x-moonshine::layout.head>
            <x-moonshine::layout.meta name="csrf-token" :content="csrf_token()"/>
            <x-moonshine::layout.favicon />
            <x-moonshine::layout.assets>
                @vite([
                    'resources/css/main.css',
                    'resources/js/app.js',
                ], 'vendor/moonshine')
            </x-moonshine::layout.assets>
        </x-moonshine::layout.head>
        <x-moonshine::layout.body>
            <x-moonshine::layout.wrapper>
                <x-moonshine::layout.sidebar :collapsed="true">
                    <x-moonshine::layout.div class="menu-header">
                        <x-moonshine::layout.div class="menu-logo">
                            <x-moonshine::layout.logo
                                href="/"
                                logo="/images/logo.png"
                                logo-small="/images/logo-small.png"
                                :minimized="true"
                            />
                        </x-moonshine::layout.div>
                        <x-moonshine::layout.div class="menu-actions">
                            <x-moonshine::layout.theme-switcher/>
                        </x-moonshine::layout.div>
                        <x-moonshine::layout.div class="menu-burger">
                            <x-moonshine::layout.burger/>
                        </x-moonshine::layout.div>
                    </x-moonshine::layout.div>
                    <x-moonshine::layout.div class="menu menu--vertical">
                        <x-moonshine::layout.menu :elements="[
                            ['label' => 'Dashboard', 'url' => '/'],
                            ['label' => 'Section', 'url' => '/section'],
                        ]"/>
                    </x-moonshine::layout.div>
                </x-moonshine::layout.sidebar>

                <x-moonshine::layout.div class="layout-main">
                    <x-moonshine::layout.div class="layout-page">
                        <x-moonshine::layout.header>
                            <x-moonshine::layout.div class="menu-burger">
                                <x-moonshine::layout.burger/>
                            </x-moonshine::layout.div>
                            <x-moonshine::breadcrumbs :items="['#' => 'Home']"/>
                            <x-moonshine::layout.search placeholder="Search" />
                            <x-moonshine::layout.locales :locales="collect()"/>
                        </x-moonshine::layout.header>
                        <x-moonshine::layout.content>
                            <article class="article">
                                Your content here
                            </article>
                        </x-moonshine::layout.content>
                    </x-moonshine::layout.div>
                </x-moonshine::layout.div>
            </x-moonshine::layout.wrapper>
        </x-moonshine::layout.body>
    </x-moonshine::layout.html>
</x-moonshine::layout>
```

### Available Blade layout components

- `<x-moonshine::layout>` -- root wrapper
- `<x-moonshine::layout.html>` -- HTML tag with Alpine.js and themes support
- `<x-moonshine::layout.head>` -- document head
- `<x-moonshine::layout.meta>` -- meta tags
- `<x-moonshine::layout.body>` -- document body
- `<x-moonshine::layout.wrapper>` -- main content wrapper
- `<x-moonshine::layout.sidebar>` -- sidebar navigation
- `<x-moonshine::layout.header>` -- page header
- `<x-moonshine::layout.content>` -- main content area
- `<x-moonshine::layout.menu>` -- menu rendering
- `<x-moonshine::layout.logo>` -- logo with small variant
- `<x-moonshine::layout.theme-switcher>` -- dark/light toggle
- `<x-moonshine::layout.burger>` -- mobile burger menu
- `<x-moonshine::layout.assets>` -- CSS/JS asset loading
- `<x-moonshine::layout.favicon>` -- favicon set
- `<x-moonshine::layout.search>` -- search component
- `<x-moonshine::layout.locales>` -- locale switcher
- `<x-moonshine::layout.div>` -- generic div wrapper
- `<x-moonshine::breadcrumbs>` -- breadcrumb navigation

## Full build() override example

For maximum control, override `build()` entirely.

```php
namespace App\MoonShine\Layouts;

use MoonShine\Laravel\Layouts\AppLayout;
use MoonShine\UI\Components\Layout\{Body, Content, Div, Flash, Html, Layout, Wrapper};
use MoonShine\UI\Components\Components;

final class TopBarOnlyLayout extends AppLayout
{
    protected bool $sidebar = false;
    protected bool $topBar = true;

    public function build(): Layout
    {
        return Layout::make([
            Html::make([
                $this->getHeadComponent(),
                Body::make([
                    Wrapper::make([
                        $this->getTopBarComponent(),
                        Div::make([
                            Flash::make(),
                            $this->getHeaderComponent(),
                            Content::make([
                                Components::make(
                                    $this->getPage()->getComponents()
                                ),
                            ]),
                            $this->getFooterComponent(),
                        ])->class('layout-page'),
                    ]),
                ]),
            ])
                ->customAttributes(['lang' => $this->getHeadLang()])
                ->withAlpineJs()
                ->withThemes(),
        ]);
    }
}
```
