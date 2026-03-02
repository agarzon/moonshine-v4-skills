# Configuration Options Reference

Complete reference for MoonShine v4 configuration options, feature flags, routing, middleware, storage, layout, palette, pages, forms, and home page settings.

---

## Table of Contents

- [Full default config/moonshine.php](#full-default-config)
- [Options (feature flags)](#options)
- [Title, Logo, Favicons](#title-logo-favicons)
- [Middleware](#middleware)
- [Routing](#routing)
- [Authentication](#authentication)
- [Localization](#localization)
- [Storage and cache](#storage-and-cache)
- [Layout and Palette](#layout-and-palette)
- [Forms, Pages, Home page](#forms-pages-home-page)
- [Getting pages and forms](#getting-pages-and-forms)
- [MoonShineConfigurator methods](#moonshineconfigurator-methods)
- [Configuration priority](#configuration-priority)

---

## Full default config/moonshine.php

```php
<?php

use Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse;
use Illuminate\Cookie\Middleware\EncryptCookies;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;
use Illuminate\Routing\Middleware\SubstituteBindings;
use Illuminate\Session\Middleware\AuthenticateSession;
use Illuminate\Session\Middleware\StartSession;
use Illuminate\View\Middleware\ShareErrorsFromSession;
use MoonShine\Laravel\Exceptions\MoonShineNotFoundException;
use MoonShine\Crud\Forms\FiltersForm;
use MoonShine\Crud\Forms\LoginForm;
use MoonShine\Laravel\Http\Middleware\Authenticate;
use MoonShine\Laravel\Http\Middleware\ChangeLocale;
use MoonShine\Laravel\Layouts\AppLayout;
use MoonShine\Laravel\Models\MoonshineUser;
use MoonShine\Laravel\Pages\Dashboard;
use MoonShine\Laravel\Pages\ErrorPage;
use MoonShine\Laravel\Pages\LoginPage;
use MoonShine\Laravel\Pages\ProfilePage;
use MoonShine\ColorManager\Palettes\PurplePalette;

return [
    'title' => env('MOONSHINE_TITLE', 'MoonShine'),
    'logo' => 'vendor/moonshine/logo.svg',
    'logo_small' => 'vendor/moonshine/logo-small.svg',
    'favicons' => [
        'apple-touch' => '/vendor/moonshine/apple-touch-icon.png',
        '32' => '/vendor/moonshine/favicon-32x32.png',
        '16' => '/vendor/moonshine/favicon-16x16.png',
        'safari-pinned-tab' => '/vendor/moonshine/safari-pinned-tab.svg',
    ],
    'use_migrations' => true,
    'use_notifications' => true,
    'use_database_notifications' => true,
    'use_routes' => true,
    'use_profile' => true,
    'dir' => 'app/MoonShine',
    'namespace' => 'App\MoonShine',
    'domain' => env('MOONSHINE_DOMAIN'),
    'prefix' => env('MOONSHINE_ROUTE_PREFIX', 'admin'),
    'page_prefix' => env('MOONSHINE_PAGE_PREFIX', 'page'),
    'resource_prefix' => env('MOONSHINE_RESOURCE_PREFIX', 'resource'),
    'home_route' => 'moonshine.index',
    'not_found_exception' => MoonShineNotFoundException::class,
    'middleware' => [
        EncryptCookies::class,
        AddQueuedCookiesToResponse::class,
        StartSession::class,
        AuthenticateSession::class,
        ShareErrorsFromSession::class,
        VerifyCsrfToken::class,
        SubstituteBindings::class,
        ChangeLocale::class,
    ],
    'disk' => 'public',
    'disk_options' => [],
    'user_avatars_dir' => 'moonshine_users',
    'cache' => 'file',
    'auth' => [
        'enabled' => true,
        'guard' => 'moonshine',
        'model' => MoonshineUser::class,
        'middleware' => [Authenticate::class],
        'pipelines' => [],
    ],
    'user_fields' => [
        'username' => 'email',
        'password' => 'password',
        'name' => 'name',
        'avatar' => 'avatar',
    ],
    'layout' => AppLayout::class,
    'palette' => PurplePalette::class,
    'forms' => [
        'login' => LoginForm::class,
        'filters' => FiltersForm::class,
    ],
    'pages' => [
        'dashboard' => Dashboard::class,
        'profile' => ProfilePage::class,
        'login' => LoginPage::class,
        'error' => ErrorPage::class,
    ],
    'locale' => 'en',
    'locale_key' => ChangeLocale::KEY,
    'locales' => [],
];
```

---

## Options

Feature flags controlling MoonShine subsystems. Must always be present in either `config/moonshine.php` or `MoonShineServiceProvider`.

| Config key | Default | Provider method | Description |
|-----------|---------|----------------|-------------|
| `use_migrations` | `true` | `$config->useMigrations()` | Enable built-in migrations (`moonshine_users`, `moonshine_user_roles`) |
| `use_notifications` | `true` | `$config->useNotifications()` | Enable notification system |
| `use_database_notifications` | `true` | `$config->useDatabaseNotifications()` | Persist notifications via database driver |
| `use_routes` | `true` | N/A | Enable built-in route registration |
| `use_profile` | `true` | N/A | Enable built-in profile page |
| `dir` | `'app/MoonShine'` | `$config->dir('app/MoonShine', 'App\MoonShine')` | Directory for artisan generators |
| `namespace` | `'App\MoonShine'` | (second param of `dir()`) | Namespace for generated classes |

---

## Title, Logo, Favicons

### title

**Default:** `env('MOONSHINE_TITLE', 'MoonShine')` | **Provider:** `$config->title('My Application')`

```php
// config/moonshine.php
'title' => 'My Admin Panel',
```

### logo / logo_small

**Defaults:** `'vendor/moonshine/logo.svg'`, `'vendor/moonshine/logo-small.svg'`
**Provider:** `$config->logo('/path.png')` and `$config->logo('/path-small.png', small: true)`

```php
// config/moonshine.php
'logo' => '/assets/logo.png',
'logo_small' => '/assets/logo-small.png',
```

```php
// MoonShineServiceProvider
$config->logo('/assets/logo.png')->logo('/assets/logo-small.png', small: true);
```

### favicons

```php
'favicons' => [
    'apple-touch' => '/vendor/moonshine/apple-touch-icon.png',
    '32' => '/vendor/moonshine/favicon-32x32.png',
    '16' => '/vendor/moonshine/favicon-16x16.png',
    'safari-pinned-tab' => '/vendor/moonshine/safari-pinned-tab.svg',
],
```

---

## Middleware

**Config key:** `middleware`

Default middleware stack applied to all MoonShine routes (independent of `web` group):

| Middleware | Purpose |
|-----------|---------|
| `EncryptCookies` | Encrypts/decrypts cookies |
| `AddQueuedCookiesToResponse` | Attaches queued cookies |
| `StartSession` | Initializes session |
| `AuthenticateSession` | Validates authenticated session |
| `ShareErrorsFromSession` | Shares validation errors with views |
| `VerifyCsrfToken` | CSRF protection |
| `SubstituteBindings` | Route model binding |
| `ChangeLocale` | Admin panel locale switching |

Append custom middleware by adding to the array in `config/moonshine.php`.

---

## Routing

> **Warning**: Route settings must be in `config/moonshine.php` -- routes load before the provider's `boot` method.

| Config key | Default | Description |
|-----------|---------|-------------|
| `domain` | `env('MOONSHINE_DOMAIN')` (null) | Restrict routes to a specific domain |
| `prefix` | `env('MOONSHINE_ROUTE_PREFIX', 'admin')` | URL prefix for all routes |
| `page_prefix` | `env('MOONSHINE_PAGE_PREFIX', 'page')` | URL segment for standalone pages |
| `resource_prefix` | `env('MOONSHINE_RESOURCE_PREFIX', 'resource')` | URL segment for resources |
| `not_found_exception` | `MoonShineNotFoundException::class` | 404 exception class |

URL pattern: `/{prefix}/{resource_prefix}/{resourceUri}/{pageUri}`

> **Warning**: Leaving `resource_prefix` empty creates URLs like `/admin/{resourceUri}/{pageUri}` but may conflict with package routes.

**Provider method for 404:** `$config->notFoundException(MyNotFoundException::class)`

---

## Authentication

### Config file settings

```php
'auth' => [
    'enabled' => true,               // false to disable built-in auth
    'guard' => 'moonshine',          // Laravel auth guard name
    'model' => MoonshineUser::class, // config file only (read at bootstrap)
    'middleware' => [Authenticate::class],
    'pipelines' => [],               // e.g., [TwoFactor::class]
],
'user_fields' => [
    'username' => 'email',
    'password' => 'password',
    'name' => 'name',
    'avatar' => 'avatar',
],
```

### Provider methods

```php
$config->guard('admin');
$config->authPipelines([\App\MoonShine\Pipelines\TwoFactor::class]);
$config->userField('username', 'login');
$config->userField('name', 'full_name');
$config->authorizationRules(
    function(ResourceContract $ctx, mixed $user, Ability $ability, mixed $data): bool {
        return true;
    }
);
```

> **Note**: `auth.model` must be in the config file (read at bootstrap, not in provider). When changing the model, also update `user_fields` and configure the guard in `config/auth.php`.

---

## Localization

| Config key | Default | Provider method |
|-----------|---------|----------------|
| `locale` | `'en'` | `$config->locale('en')` |
| `locales` | `[]` | `$config->locales(['en', 'ru'])` |
| `locale_key` | `'_lang'` | `$config->localeKey('_lang')` |

```php
// Key-value format with display names
'locales' => ['en' => 'English', 'ru' => 'Russian'],
```

```php
// Provider with translated labels
$config->locales(fn() => ['ru' => __('lang.russian'), 'en' => __('lang.english')]);
```

The `ChangeLocale` middleware reads the `locale_key` query parameter, validates against `locales`, stores in session, and applies it.

---

## Storage and cache

| Config key | Default | Provider method | Description |
|-----------|---------|----------------|-------------|
| `disk` | `'public'` | `$config->disk('public')` | Filesystem disk for uploads |
| `disk_options` | `[]` | `$config->disk('public', options: [...])` | Additional disk options |
| `user_avatars_dir` | `'moonshine_users'` | `$config->userAvatarsDir('images/avatars')` | Avatar storage directory |
| `cache` | `'file'` | `$config->cacheDriver('redis')` | Cache driver for MoonShine |

```php
// S3 example
'disk' => 's3',
'disk_options' => ['visibility' => 'public'],
```

---

## Layout and Palette

### layout

**Default:** `AppLayout::class` | **Provider:** `$config->layout(CustomLayout::class)`

> **Note**: In v4, `CompactLayout` was removed. Extend `AppLayout` instead.

```php
// config/moonshine.php
'layout' => \App\MoonShine\Layouts\CustomLayout::class,
```

### palette

**Default:** `PurplePalette::class` | **Provider:** `$config->set('palette', CorporatePalette::class)`

The default palette used when a layout does not define its own.

```php
'palette' => \App\MoonShine\Palettes\CorporatePalette::class,
```

---

## Forms, Pages, Home page

### forms

Maps form names to classes. In v4, defaults moved to `MoonShine\Crud\Forms\` namespace.

```php
'forms' => [
    'login' => \App\MoonShine\Forms\MyLoginForm::class,
    'filters' => \MoonShine\Crud\Forms\FiltersForm::class,
],
```

Provider: `$config->set('forms.login', MyLoginForm::class);`

### pages

Maps page names to classes.

```php
'pages' => [
    'dashboard' => \App\MoonShine\Pages\DashboardPage::class,
    'profile' => ProfilePage::class,
    'login' => LoginPage::class,
    'error' => ErrorPage::class,
],
```

Provider: `$config->changePage(LoginPage::class, MyLoginPage::class);`

### home_route / home_url

**Default:** `'moonshine.index'`

Used for post-login redirects, logo link, and 404 "go home" link.

```php
'home_route' => 'moonshine.index',   // by route name
'home_url' => '/admin/page/some-page', // by direct URL
```

Provider: `$config->homeRoute('moonshine.index')` or `$config->homeUrl('/admin/page/some-page')`

---

## Getting pages and forms

### getPage

```php
$page = moonshineConfig()->getPage('dashboard');

// Via DI
public function index(ConfiguratorContract $config)
{
    $page = $config->getPage('dashboard');
}
```

Signature: `getPage(string $name, string $default, mixed ...$parameters)`

### getForm

```php
$form = moonshineConfig()->getForm('login');

// Via DI
public function index(ConfiguratorContract $config)
{
    $form = $config->getForm('login');
}
```

Signature: `getForm(string $name, string $default, mixed ...$parameters)`

---

## MoonShineConfigurator methods

| Method | Description |
|--------|-------------|
| `title(string $title)` | Set the page `<title>` |
| `logo(string $path, bool $small = false)` | Set logo (call twice for full and small) |
| `dir(string $dir, string $namespace)` | Set MoonShine directory and namespace |
| `useMigrations()` | Enable built-in migrations |
| `useNotifications()` | Enable notification system |
| `useDatabaseNotifications()` | Enable database notification driver |
| `guard(string $guard)` | Set the auth guard |
| `authPipelines(array $pipelines)` | Set authentication pipelines |
| `authorizationRules(Closure $rules)` | Define global authorization rules |
| `layout(string $class)` | Set the default layout class |
| `locale(string $locale)` | Set the default locale |
| `locales(array $locales)` | Set available locales |
| `localeKey(string $key)` | Set the locale query parameter name |
| `disk(string $disk, array $options = [])` | Set the filesystem disk |
| `userAvatarsDir(string $dir)` | Set the user avatars directory |
| `cacheDriver(string $driver)` | Set the cache driver |
| `homeRoute(string $route)` | Set the home page by route name |
| `homeUrl(string $url)` | Set the home page by URL |
| `notFoundException(string $class)` | Set the 404 exception class |
| `changePage(string $old, string $new)` | Replace a default page class |
| `set(string $key, mixed $value)` | Set an arbitrary config value |
| `getPage(string $name, ...)` | Get a page instance by config name |
| `getForm(string $name, ...)` | Get a form instance by config name |
| `userField(string $field, string $column)` | Map a user field to a column |

---

## Configuration priority

Provider settings take precedence over `config/moonshine.php` values.

| Setting type | Where to configure |
|-------------|-------------------|
| Route settings (`prefix`, `domain`, `page_prefix`, `resource_prefix`) | `config/moonshine.php` only (loaded before boot) |
| `auth.model` | `config/moonshine.php` only (needed at bootstrap) |
| Feature flags (`use_migrations`, etc.) | Either location |
| Dynamic settings (conditional logic, environment-based) | `MoonShineServiceProvider` |
| Simple static values | `config/moonshine.php` |
