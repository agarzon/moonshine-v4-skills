# Color Palette System Reference (MoonShine v4)

Full color API: palette classes, ColorManager methods, color tokens, dark/light mode, and global overrides.

## CRITICAL v4 change: Palette system

MoonShine v4 introduces palette classes. The default palette is `PurplePalette`. Colors now use OKLCH format by default (HEX, RGB, and RGBA also accepted). Direct color arrays are replaced by palette classes implementing `PaletteContract`.

## Default color tokens

The base color set after installation (before any palette is applied):

```php
return [
    'body' => '0.99 0 0',
    'primary' => '0.25 0 0',
    'primary-text' => '1 0 0',
    'secondary' => '0.6 0 0',
    'secondary-text' => '1 0 0',
    'base' => [
        'text' => '0.25 0 0',
        'stroke' => '0.25 0 0 / 20%',
        'default' => '1 0 0',
        50 => '0.985 0 0',
        100 => '0.97 0 0',
        200 => '0.955 0 0',
        300 => '0.94 0 0',
        400 => '0.925 0 0',
        500 => '0.91 0 0',
        600 => '0.88 0 0',
        700 => '0.85 0 0',
        800 => '0.80 0 0',
        900 => '0.75 0 0',
    ],
    'success' => '0.64 0.22 142.49',
    'success-text' => '0.46 0.16 142.49',
    'warning' => '0.75 0.17 75.35',
    'warning-text' => '0.5 0.10 76.10',
    'error' => '0.58 0.21 26.855',
    'error-text' => '0.37 0.145 26.85',
    'info' => '0.60 0.219 257.63',
    'info-text' => '0.35 0.12 257.63',
];
```

## Palette system

Palettes encapsulate light and dark color schemes in dedicated classes.

### Available built-in palettes

All palettes are in the `MoonShine\ColorManager\Palettes` namespace:

| Palette | Description |
|---|---|
| `PurplePalette` | **Default.** Classic purple and magenta mix. |
| `CyanPalette` | True cyan blue-green. |
| `GrayPalette` | Cool neutral gray. |
| `GreenPalette` | Natural green tones. |
| `HalloweenPalette` | Orange and purple spooky theme. |
| `LimePalette` | Bright lime/chartreuse. |
| `NeutralPalette` | Neutral black and white classic. |
| `OrangePalette` | Classic orange. |
| `PinkPalette` | Bold hot pink shades. |
| `RetroPalette` | Vintage yellowish green. |
| `RosePalette` | Warm peachy-rose tones. |
| `SkyPalette` | Sky blue with a purple undertone. |
| `SpringPalette` | Fresh pastel mint green. |
| `TealPalette` | Pure cyan-teal blend. |
| `ValentinePalette` | Romantic red and pink duo. |
| `WinterPalette` | Cool icy blue tones. |
| `YellowPalette` | Greenish yellow. |

> Preview or build custom palettes at: https://getmoonshine.app/palette-generator

### Setting a palette

Three ways to activate a palette:

#### 1. In config/moonshine.php (global)

```php
// config/moonshine.php
'palette' => \MoonShine\ColorManager\Palettes\PurplePalette::class,
```

#### 2. In layout via $palette property (per-layout)

```php
use App\MoonShine\Palettes\CorporatePalette;
use MoonShine\Laravel\Layouts\AppLayout;

final class MoonShineLayout extends AppLayout
{
    protected ?string $palette = CorporatePalette::class;
}
```

Setting `$palette = null` falls back to the global config value.

#### 3. Programmatically via ColorManager

```php
$colorManager->palette(new CorporatePalette());
```

### Creating a custom palette

Implement `PaletteContract` with `getColors()` and `getDarkColors()` methods.

```php
namespace App\MoonShine\Palettes;

use MoonShine\Contracts\ColorManager\PaletteContract;

final class CorporatePalette implements PaletteContract
{
    public function getColors(): array
    {
        return [
            'body' => '0.99 0 0',
            'primary' => 'oklch(65% 0.18 264)',
            'primary-text' => '1 0 0',
            'secondary' => 'oklch(70% 0.14 230)',
            'secondary-text' => '1 0 0',
            'base' => [
                'text' => '0.25 0 0',
                'stroke' => '0.25 0 0 / 20%',
                'default' => '1 0 0',
                50 => 'oklch(98% 0.02 250)',
                100 => '0.97 0 0',
                200 => '0.955 0 0',
                300 => '0.94 0 0',
                400 => '0.925 0 0',
                500 => '0.91 0 0',
                600 => '0.88 0 0',
                700 => '0.85 0 0',
                800 => '0.80 0 0',
                900 => '0.75 0 0',
            ],
            'success' => 'oklch(63.9% 0.218 142.495)',
            'success-text' => '0.46 0.16 142.49',
            'warning' => 'oklch(80.88% 0.170358 75.3501)',
            'warning-text' => '0.5 0.10 76.10',
            'error' => 'oklch(58.9% 0.214 26.855)',
            'error-text' => '0.37 0.145 26.85',
            'info' => 'oklch(60.1% 0.219 257.63)',
            'info-text' => '0.35 0.12 257.63',
        ];
    }

    public function getDarkColors(): array
    {
        return [
            'body' => '0.2 0.0168 274.32',
            'primary' => 'oklch(60% 0.17 264)',
            'primary-text' => '1 0 0',
            'secondary' => 'oklch(65% 0.12 230)',
            'secondary-text' => '1 0 0',
            'base' => [
                'text' => '1 0 0',
                'stroke' => '1 0 0 / 10%',
                'default' => '0.24 0.0168 274.32',
                50 => '0.985 0 0',
                100 => '0.97 0 0',
                200 => '0.955 0 0',
                300 => '0.94 0 0',
                400 => '0.925 0 0',
                500 => '0.91 0 0',
                600 => '0.88 0 0',
                700 => '0.85 0 0',
                800 => '0.80 0 0',
                900 => '0.39 0.025 274.32',
            ],
            'success' => '0.639 0.218 142.495',
            'success-text' => '0.46 0.16 142.49',
            'warning' => '0.898 0.177 96.726',
            'warning-text' => '0.5 0.10 76.10',
            'error' => '0.589 0.214 26.855',
            'error-text' => '0.37 0.145 26.85',
            'info' => '0.601 0.219 257.63',
            'info-text' => '0.35 0.12 257.63',
        ];
    }
}
```

## ColorManager methods

### Setting colors

```php
// Set a single color (OKLCH, HEX, RGB, RGBA accepted)
$colorManager->set('primary', 'oklch(65% 0.18 264)');

// Set color for dark theme only
$colorManager->set('primary', 'oklch(60% 0.17 264)', dark: true);

// Apply to both light and dark themes at once
$colorManager->set('primary', '#7357ff', everything: true);

// Explicit helper that syncs both themes
$colorManager->setEverything('primary-text', '#ffffff');

// Set a specific shade via dot notation
$colorManager->set('base.500', 'oklch(70% 0.10 280)');
$colorManager->set('base.500', '0.36 0.023 274.32', dark: true);
```

### Bulk assignment

```php
$colorManager->bulkAssign([
    'base' => [
        'default' => '0 0 0',
        50 => '0.99 0 0',
        100 => '0.98 0 0',
    ],
], everything: true);
```

### Theme-level assignment

```php
$colorManager->theme([
    'body' => '1 0 0',
    'stroke' => '1 0 0 / 10%',
    'default' => '0.24 0.0168 274.32',
    900 => '0.39 0.025 274.32',
], dark: true);
```

### Getting colors

```php
// Get color (returns HEX by default)
$colorManager->get('primary');

// Get in stored format
$colorManager->get('primary', hex: false);

// Get a specific shade
$colorManager->get('base', 500);

// Get all colors
$colorManager->getAll();           // light theme
$colorManager->getAll(dark: true); // dark theme
```

### Applying a palette

```php
$colorManager->palette(new \App\MoonShine\Palettes\CorporatePalette());
```

## Component shortcuts (ColorShortcuts trait)

Dynamic methods for common color operations. Each accepts `dark` and `everything` flags.

### Primary and secondary

```php
$colorManager->primary('oklch(65% 0.18 264)', text: '#ffffff');
$colorManager->secondary('oklch(70% 0.14 230)', text: '#ffffff');
```

### Status colors

```php
$colorManager->success('oklch(63.9% 0.218 142.495)', text: '#194638', everything: true);
$colorManager->successBg('oklch(63.9% 0.218 142.495)');
$colorManager->warningBg('oklch(80.88% 0.170358 75.3501)');
$colorManager->errorBg('oklch(58.9% 0.214 26.855)');
$colorManager->infoBg('oklch(60.1% 0.219 257.63)');
```

### Background and text

```php
$colorManager->background('oklch(91% 0 0)', pageBg: 'oklch(98% 0 0)');
$colorManager->text('oklch(20% 0.04 274)');
```

### UI element shortcuts

```php
$colorManager->borders('oklch(72% 0.02 274)');

$colorManager->button(
    'oklch(65% 0.18 264)',
    text: '#ffffff',
    hoverBg: 'oklch(60% 0.18 264)',
    hoverText: '#f8fafc'
);

$colorManager->menu(
    'oklch(70% 0.14 230)',
    text: '#0f172a',
    hoverBg: 'oklch(80% 0.05 230)'
);

$colorManager->dropzone(
    'oklch(98% 0 0)',
    text: '#0f172a',
    icon: '#2563eb'
);

$colorManager->form(
    bg: 'oklch(100% 0 0)',
    text: '#0f172a',
    focus: '#2563eb',
    disabled: '#f1f5f9',
    disabledText: '#64748b'
);

$colorManager->collapse(
    'oklch(100% 0 0)',
    text: '#0f172a',
    bgOpen: 'oklch(96% 0 0)'
);

$colorManager->progress(
    bg: 'oklch(96% 0 0)',
    barBg: 'oklch(65% 0.18 264)',
    text: '#0f172a'
);

$colorManager->theme('oklch(95% 0.01 274)', 400);
$colorManager->theme('oklch(35% 0.18 274)', 800, dark: true);
```

## Using colors in layout

### Via palette property

```php
use App\MoonShine\Palettes\CorporatePalette;
use MoonShine\Laravel\Layouts\AppLayout;

final class MoonShineLayout extends AppLayout
{
    protected ?string $palette = CorporatePalette::class;
}
```

### Via colors() method (full control)

```php
use MoonShine\Contracts\ColorManager\ColorManagerContract;
use MoonShine\Laravel\Layouts\AppLayout;

final class MoonShineLayout extends AppLayout
{
    protected function colors(ColorManagerContract $colorManager): void
    {
        $colorManager
            ->primary('oklch(65% 0.18 264)')
            ->secondary('oklch(70% 0.14 230)')
            ->bulkAssign([
                'theme' => [
                    'body' => '0 0 0',
                    50 => '0.99 0 0',
                    100 => '0.98 0 0',
                    900 => '0.90 0 0',
                ],
            ])
            ->successBg('oklch(63.9% 0.218 142.495)')
            ->warningBg('oklch(80.88% 0.170358 75.3501)')
            ->errorBg('oklch(58.9% 0.214 26.855)')
            ->infoBg('oklch(60.1% 0.219 257.63)');

        // Dark theme
        $colorManager
            ->set('body', '0.2 0.0168 274.32', dark: true)
            ->theme([
                'body' => '1 0 0',
                'stroke' => '1 0 0 / 10%',
                'default' => '0.24 0.0168 274.32',
                900 => '0.39 0.025 274.32',
            ], dark: true)
            ->successBg('0.639 0.218 142.495', dark: true)
            ->warningBg('0.898 0.177 96.726', dark: true)
            ->errorBg('0.589 0.214 26.855', dark: true)
            ->infoBg('0.601 0.219 257.63', dark: true);
    }
}
```

## Global override via ServiceProvider

Override colors for all layouts from `MoonShineServiceProvider`.

```php
use Illuminate\Support\ServiceProvider;
use MoonShine\ColorManager\ColorManager;
use MoonShine\Contracts\ColorManager\ColorManagerContract;
use MoonShine\Contracts\Core\DependencyInjection\CoreContract;
use MoonShine\Laravel\DependencyInjection\ConfiguratorContract;
use MoonShine\Laravel\DependencyInjection\MoonShine;
use MoonShine\Laravel\DependencyInjection\MoonShineConfigurator;

class MoonShineServiceProvider extends ServiceProvider
{
    public function boot(
        CoreContract $core,
        ConfiguratorContract $config,
        ColorManagerContract $colors,
    ): void
    {
        $colors->palette(new \App\MoonShine\Palettes\CorporatePalette());
        $colors->primary('oklch(65% 0.18 264)');
        $colors->successBg('oklch(70% 0.15 142)');
    }
}
```

> WARNING: Layout `colors()` loads after ServiceProvider and takes precedence. When using palettes globally, make sure the target layout does not override colors or provide its own `$palette`.

## HTML output

Generate CSS custom properties for manual use:

```php
$colorManager->toHtml()
```

Result:

```html
<style>
    :root {
        --primary:0.627 0.265 303.9;
        --secondary:0.746 0.16 232.661;
        /* other light theme variables */
    }
    :root.dark {
        /* dark theme variables */
    }
</style>
```

## Color conversion utility

`ColorMutator` converts between HEX, RGB, RGBA, and OKLCH.

```php
use MoonShine\ColorManager\ColorMutator;

// Convert to HEX
ColorMutator::toHEX('oklch(65% 0.18 264)'); // '#7357ff'

// Convert to RGB
ColorMutator::toRGB('#7357ff'); // 'rgb(115,87,255)'

// Convert to OKLCH
ColorMutator::toOKLCH('rgb(115,87,255)'); // 'oklch(65.07% 0.20052 282.513)'
```

## Color token reference

### Top-level tokens

| Token | Description |
|---|---|
| `body` | Page background |
| `primary` | Primary accent color |
| `primary-text` | Text on primary backgrounds |
| `secondary` | Secondary accent color |
| `secondary-text` | Text on secondary backgrounds |
| `success` | Success state background |
| `success-text` | Success state text |
| `warning` | Warning state background |
| `warning-text` | Warning state text |
| `error` | Error state background |
| `error-text` | Error state text |
| `info` | Info state background |
| `info-text` | Info state text |

### Base shade tokens

| Token | Description |
|---|---|
| `base.text` | Default text color |
| `base.stroke` | Border/stroke color |
| `base.default` | Default base color |
| `base.50` - `base.900` | Shade scale from lightest to darkest |
