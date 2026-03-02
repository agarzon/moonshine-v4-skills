# Assets, Branding & Custom Build Reference (v4)

## AssetManager

`MoonShine\Contracts\AssetManager\AssetManagerContract` — manages CSS and JavaScript assets.

### Asset Types

| Class | Output |
|---|---|
| `MoonShine\AssetManager\Js` | `<script src>` tag |
| `MoonShine\AssetManager\Css` | `<link>` tag |
| `MoonShine\AssetManager\InlineCss` | `<style>` tag |
| `MoonShine\AssetManager\InlineJs` | `<script>` inline tag |
| `MoonShine\AssetManager\Raw` | Arbitrary content in `<head>` |

### JavaScript

```php
use MoonShine\AssetManager\Js;

Js::make('/js/app.js')
Js::make('/js/app.js')->defer()
Js::make('/js/app.js')->customAttributes(['data-module' => 'main'])
```

### CSS

```php
use MoonShine\AssetManager\Css;

Css::make('/css/styles.css')
Css::make('/css/styles.css')->defer()
Css::make('/css/styles.css')->customAttributes(['media' => 'print'])
```

### Inline JS / CSS

```php
use MoonShine\AssetManager\InlineJs;
use MoonShine\AssetManager\InlineCss;

InlineJs::make('console.log("Loaded");')
InlineCss::make('.custom { color: red; }')
```

### Raw content

```php
use MoonShine\AssetManager\Raw;

Raw::make('<link rel="preconnect" href="https://fonts.googleapis.com">')
```

### Asset Collections (ordering)

```php
// Add assets to the end (before main list)
$assetManager->append([Js::make('/js/last.js')]);

// Add assets to the beginning (after main list)
$assetManager->prepend([Js::make('/js/first.js')]);

// Add in order of registration
$assetManager->add([Js::make('/js/middle.js')]);
```

### Versioning

```php
Js::make('/js/app.js')->version('1.0.0')
// Result: /js/app.js?v=1.0.0
```

Default version is the MoonShine version.

### Modify assets collection

```php
$assetManager->modifyAssets(function($assets) {
    return array_filter($assets, fn($asset) =>
        !str_contains($asset->getLink(), 'remove-this')
    );
});
```

---

## Adding Assets at Different Levels

### Global (ServiceProvider)

```php
use MoonShine\AssetManager\Js;
use MoonShine\Contracts\AssetManager\AssetManagerContract;

public function boot(CoreContract $core, ConfiguratorContract $config, AssetManagerContract $assets): void
{
    $assets->add(Js::make('/js/app.js'));
}
```

### Layout

```php
use MoonShine\AssetManager\Js;

final class MoonShineLayout extends AppLayout
{
    protected function assets(): array
    {
        return [
            Js::make(Vite::asset('resources/js/app.js'))
        ];
    }
}
```

### CrudResource

```php
protected function onLoad(): void
{
    $this->getAssetManager()
        ->prepend(InlineJs::make('alert(1)'))
        ->append(Js::make('/js/app.js'));
}
```

### Page

```php
protected function onLoad(): void
{
    parent::onLoad();
    $this->getAssetManager()
        ->add(Css::make('/css/app.css'))
        ->append(Js::make('/js/app.js'));
}
```

### Component / Field (on the fly)

```php
Box::make()->addAssets([
    Js::make('/js/custom.js'),
    Css::make('/css/styles.css'),
])
```

### Component class (assets method)

```php
final class MyComponent extends MoonShineComponent
{
    protected function assets(): array
    {
        return [
            Js::make('/js/custom.js'),
            Css::make('/css/styles.css'),
        ];
    }
}
```

### Blade

```blade
<x-moonshine::layout.assets>
    @vite(['resources/css/main.css', 'resources/js/app.js'], 'vendor/moonshine')
</x-moonshine::layout.assets>
```

---

## Logo

`MoonShine\UI\Components\Layout\Logo` — admin panel logo.

```php
use MoonShine\UI\Components\Layout\Logo;

Logo::make(
    href: '/admin',
    logo: '/vendor/moonshine/logo.svg',
    logoSmall: '/vendor/moonshine/logo-small.svg',
    title: 'My Admin',
    minimized: false, // auto-select small logo when sidebar is minimized
)
```

**Logo attributes:**

```php
Logo::make('/admin', '/logo.svg', '/logo-sm.svg')
    ->logoAttributes(['class' => 'custom-logo'])
    ->logoSmallAttributes(['class' => 'custom-logo-sm'])
```

**Dark mode logos:**

```php
protected function getLogoComponent(): Logo
{
    return parent::getLogoComponent()
        ->darkMode(
            asset('logo-dark.svg'),
            asset('logo-dark-small.svg'),
        );
}
```

**Via config:**

```php
// config/moonshine.php
'logo' => '/assets/logo.png',
'logo_small' => '/assets/logo-small.svg',
```

**Blade:**

```blade
<x-moonshine::layout.logo
    :href="'/admin'"
    :logo="'/vendor/moonshine/logo.svg'"
    :logoSmall="'/vendor/moonshine/logo-small.svg'"
/>
```

---

## Favicon

`MoonShine\UI\Components\Layout\Favicon` — favicon management.

**Via config (simplest):**

```php
// config/moonshine.php
'favicons' => [
    'apple-touch' => '/images/apple-touch-icon.png',
    '32' => '/images/favicon-32x32.png',
    '16' => '/images/favicon-16x16.png',
    'safari-pinned-tab' => '/images/safari-pinned-tab.svg',
],
```

**Custom assets:**

```php
Favicon::make()
    ->customAssets([
        'apple-touch' => Vite::asset('favicons/apple-touch-icon.png'),
        '32' => Vite::asset('favicons/favicon-32x32.png'),
        '16' => Vite::asset('favicons/favicon-16x16.png'),
        'safari-pinned-tab' => Vite::asset('favicons/safari-pinned-tab.svg'),
    ])
```

**Safari pinned tab color:**

```php
Favicon::make()->bodyColor('#7843e9')
```

---

## Footer

`MoonShine\UI\Components\Layout\Footer` — footer block.

```php
use MoonShine\UI\Components\Layout\Footer;

Footer::make()
    ->copyright(fn(): string => 'Your Brand')
    ->menu([
        'https://example.com/docs' => 'Documentation',
        'https://example.com/support' => 'Support',
    ])
```

**Override in layout:**

```php
protected function getFooterComponent(): Footer
{
    return parent::getFooterComponent()
        ->copyright('My App')
        ->menu(['/' => 'Home']);
}
```

**Blade:**

```blade
<x-moonshine::layout.footer copyright="Your brand" :menu="['/' => 'Home']" />
```

---

## Theme Switcher

`MoonShine\UI\Components\Layout\ThemeSwitcher` — light/dark mode toggle.

```php
use MoonShine\UI\Components\Layout\ThemeSwitcher;

ThemeSwitcher::make()
```

**Blade:**

```blade
<x-moonshine::layout.theme-switcher />
```

**Layout dark mode control:**

```php
final class MoonShineLayout extends AppLayout
{
    // Always dark theme (no toggle)
    protected function isAlwaysDark(): bool { return true; }

    // Show/hide theme switcher
    protected function hasThemes(): bool { return true; }
}
```

---

## Custom Build (Vite + TailwindCSS)

For additional TailwindCSS classes not included in default MoonShine build.

### Automatic publishing

```shell
php artisan moonshine:publish
# Select "Assets Template"
```

Publishes `vite.config.js`, `postcss.config.js`, `resources/css/app.css`. Requires TailwindCSS 4+ and Laravel 12+.

### Implementation via Layout

```php
final class MoonShineLayout extends AppLayout
{
    protected function assets(): array
    {
        return [
            $this->getMainThemeJs(),
            Css::make(Vite::asset('resources/css/app.css')),
            Js::make(Vite::asset('resources/js/app.js')),
        ];
    }
}
```

### Manual vite.config.js

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    resolve: {
        alias: {
            '@moonshine-resources': '/vendor/moonshine/moonshine/src/UI/resources',
        }
    },
});
```

### postcss.config.js

```js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

### resources/css/app.css

```css
@import '../../vendor/moonshine/moonshine/src/UI/resources/css/main.css';

@source '../../vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php';
@source '../../storage/framework/views/*.php';
@source '../**/*.blade.php';
@source '../**/*.js';
```
