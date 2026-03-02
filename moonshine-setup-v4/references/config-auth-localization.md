# Artisan Commands, Localization & Troubleshooting Reference

MoonShine v4 artisan command signatures, localization setup, and common troubleshooting solutions.

---

## Table of Contents

- [Artisan commands](#artisan-commands)
  - [install](#install)
  - [user](#user)
  - [resource](#resource)
  - [page](#page)
  - [layout](#layout)
  - [component](#component)
  - [field](#field)
  - [controller](#controller)
  - [handler](#handler)
  - [policy](#policy)
  - [type-cast](#type-cast)
  - [apply](#apply)
  - [publish](#publish)
  - [resources-list](#resources-list)
  - [pages-list](#pages-list)
- [Localization](#localization)
- [Troubleshooting](#troubleshooting)

---

## Artisan commands

> **Tip**: Use the `space` key to select options in interactive prompts.

### install

Command to install the MoonShine package in your Laravel project. Run once at the start.

```shell
php artisan moonshine:install
```

Signature:

```
moonshine:install {--u|without-user} {--m|without-migrations} {--l|default-layout} {--a|without-auth} {--d|without-notifications} {--t|tests-mode} {--Q|quick-mode}
```

| Option | Short | Effect |
|--------|-------|--------|
| `--without-user` | `-u` | Skip creating a superuser |
| `--without-migrations` | `-m` | Skip running migrations |
| `--default-layout` | `-l` | Select the default template |
| `--without-auth` | `-a` | Disable authentication |
| `--without-notifications` | `-d` | Disable notifications |
| `--tests-mode` | `-t` | Test mode |
| `--quick-mode` | `-Q` | Skip all dialogs, use default settings |

Example (fully automated):

```shell
php artisan moonshine:install -Q
```

### user

Create a superuser for the admin panel.

```shell
php artisan moonshine:user
```

Signature:

```
moonshine:user {--u|username=} {--N|name=} {--p|password=}
```

| Option | Short | Effect |
|--------|-------|--------|
| `--username` | `-u` | User login/email |
| `--name` | `-N` | User display name |
| `--password` | `-p` | Password |

Example (non-interactive):

```shell
php artisan moonshine:user --username=admin@example.com --name=Admin --password=secret123
```

### resource

Generate a model resource with CRUD pages.

```shell
php artisan moonshine:resource
```

Signature:

```
moonshine:resource {className?} {--type=} {--m|model=} {--t|title=} {--test} {--pest} {--force} {--p|policy} {--base-dir=} {--base-namespace=}
```

| Option | Short | Effect |
|--------|-------|--------|
| `--type` | | Resource type: 1=ModelResource (default), 2=CrudResource, 3=Blank Resource |
| `--model` | `-m` | Eloquent model class for ModelResource |
| `--title` | `-t` | Section title |
| `--test` | | Generate a PHPUnit test class |
| `--pest` | | Generate a Pest test class |
| `--force` | | Overwrite existing resource without confirmation |
| `--policy` | `-p` | Also create a Policy class |
| `--base-dir` | | Change the base directory of the class |
| `--base-namespace` | | Change the base namespace of the class |

Resource types:
- **ModelResource** -- standard resource for managing Eloquent models.
- **CrudResource** -- resource without dependence on Eloquent.
- **Blank Resource** -- empty resource for custom implementations.

Examples:

```shell
php artisan moonshine:resource Post --model=CustomPost --title="Articles"

php artisan moonshine:resource Post --model="App\Models\CustomPost" --policy

php artisan moonshine:resource Post --type=2
```

After execution, a resource file with CRUD pages (`{Name}IndexPage`, `{Name}FormPage`, `{Name}DetailPage`) is created in `app/MoonShine/Resources/`.

### page

Generate a custom page.

```shell
php artisan moonshine:page
```

Signature:

```
moonshine:page {className?} {--force} {--without-register} {--skip-menu} {--crud} {--dir=} {--extends=} {--base-dir=} {--base-namespace=} {--resource=}
```

| Option | Effect |
|--------|--------|
| `--force` | Do not ask for the page type |
| `--without-register` | Skip automatic registration in the provider |
| `--skip-menu` | Do not add this page to the menu when using autoload menu |
| `--crud` | Create a group of pages: index, detail, and form |
| `--dir` | Directory relative to `app/MoonShine` (default: `Pages`) |
| `--extends` | Base class to extend: `IndexPage`, `FormPage`, or `DetailPage` |
| `--base-dir` | Change the base directory of the class |
| `--base-namespace` | Change the base namespace of the class |
| `--resource` | Resource for which pages are created |

Examples:

```shell
php artisan moonshine:page Dashboard

php artisan moonshine:page OrderPage --extends=FormPage

php artisan moonshine:page Order --crud --resource=OrderResource
```

### layout

Generate a layout template.

```shell
php artisan moonshine:layout
```

Signature:

```
moonshine:layout {className?} {--default} {--palette=} {--dir=} {--base-dir=} {--base-namespace=}
```

| Option | Effect |
|--------|--------|
| `--default` | Set as the default template in the config |
| `--palette` | Preselect a palette class for the layout |
| `--dir` | Directory relative to `app/MoonShine` (default: `Layouts`) |
| `--base-dir` | Change the base directory of the class |
| `--base-namespace` | Change the base namespace of the class |

Example:

```shell
php artisan moonshine:layout AdminLayout --default

php artisan moonshine:layout DarkLayout --palette="App\MoonShine\Palettes\DarkPalette"
```

### component

Generate a custom component (PHP class + Blade file).

```shell
php artisan moonshine:component
```

Signature:

```
moonshine:component {className?} {--base-dir=} {--base-namespace=}
```

| Option | Effect |
|--------|--------|
| `--base-dir` | Change the base directory of the class |
| `--base-namespace` | Change the base namespace of the class |

Creates a class in `app/MoonShine/Components/` and a Blade file in `resources/views/admin/components/`.

### field

Generate a custom field (PHP class + Blade file).

```shell
php artisan moonshine:field
```

Signature:

```
moonshine:field {className?} {--base-dir=} {--base-namespace=}
```

| Option | Effect |
|--------|--------|
| `--base-dir` | Change the base directory of the class |
| `--base-namespace` | Change the base namespace of the class |

When executing the command, you can specify whether the field extends the base class or another field. Creates a class in `app/MoonShine/Fields/` and a Blade file in `resources/views/admin/fields/`.

### controller

Generate a controller for admin panel routes.

```shell
php artisan moonshine:controller
```

Signature:

```
moonshine:controller {className?} {--base-dir=} {--base-namespace=}
```

Creates a controller class in `app/MoonShine/Controllers/`.

### handler

Generate a handler class.

```shell
php artisan moonshine:handler
```

Signature:

```
moonshine:handler {className?} {--base-dir=} {--base-namespace=}
```

Creates a handler class in `app/MoonShine/Handlers/`.

### policy

Generate a Policy class tied to the admin panel user.

```shell
php artisan moonshine:policy
```

Creates a class in `app/Policies/`. No additional options.

### type-cast

Generate a TypeCast class for working with data.

```shell
php artisan moonshine:type-cast
```

Signature:

```
moonshine:type-cast {className?} {--base-dir=} {--base-namespace=}
```

Creates a file in `app/MoonShine/TypeCasts/`.

### apply

Generate an apply class for custom field apply logic.

```shell
php artisan moonshine:apply
```

Signature:

```
moonshine:apply {className?} {--base-dir=} {--base-namespace=}
```

Creates a file in `app/MoonShine/Applies/`. The created class needs to be registered in the service provider.

### publish

Publish MoonShine assets, templates, or system classes.

```shell
php artisan moonshine:publish
```

Interactive options:
- **Assets** -- frontend assets for the admin panel.
- **Assets template** -- template for adding custom styles or creating a custom theme.
- **System Resources** -- `MoonShineUserResource`, `MoonShineUserRoleResource` for customization.
- **System Forms** -- `LoginForm`, `FiltersForm` for customization.
- **System Pages** -- `ProfilePage`, `LoginPage`, `ErrorPage` for customization.

You can specify the publication type directly:

```shell
php artisan moonshine:publish assets
php artisan moonshine:publish assets-template
php artisan moonshine:publish resources
php artisan moonshine:publish forms
php artisan moonshine:publish pages
```

### resources-list

Display all registered MoonShine resources.

```shell
php artisan moonshine:resources
```

Signature:

```
moonshine:resources {--json}
```

| Option | Effect |
|--------|--------|
| `--json` | Output in JSON format for automation |

Displays all registered resources along with their pages in a readable format.

### pages-list

Display all registered standalone MoonShine pages (excluding pages belonging to resources).

```shell
php artisan moonshine:pages
```

Signature:

```
moonshine:pages {--json}
```

| Option | Effect |
|--------|--------|
| `--json` | Output in JSON format for automation |

---

## Localization

### Language files

After installing MoonShine, a directory `lang/vendor/moonshine/` is created for translation files. By default, only English is included.

```
lang/
  vendor/
    moonshine/
      en/
        ui.php
        validation.php
      ru/
        ui.php
        validation.php
```

Additional languages can be found in the MoonShine plugins section, or use the third-party package [laravel-lang/moonshine](https://laravel-lang.com/packages-moonshine.html) which provides many localizations in one package.

### Publishing language files

To publish language files for customization:

```shell
php artisan vendor:publish --tag=moonshine-lang
```

### Default language configuration

Set the default locale via config file or service provider:

```php
// config/moonshine.php
'locale' => 'ru',
```

```php
// MoonShineServiceProvider
$config->locale('ru');
```

### Available languages

Configure which locales are available for switching:

```php
// config/moonshine.php
'locales' => ['en', 'ru'],

// Key-value format with display names
'locales' => [
    'en' => 'English',
    'ru' => 'Russian',
],
```

```php
// MoonShineServiceProvider
$config->locales(['en', 'ru']);

// With translated labels
$config->locales(
    fn() => [
        'ru' => __('lang.russian'),
        'en' => __('lang.english'),
    ]
);
```

> **Warning**: If you change the language in the panel interface, the selection is saved in sessions and will take precedence over the configuration.

### Locale query parameter

The parameter name used for locale switching (default `_lang`):

```php
// config/moonshine.php
'locale_key' => '_lang',
```

```php
// MoonShineServiceProvider
$config->localeKey('_lang');
```

### ChangeLocale middleware

The `MoonShine\Laravel\Http\Middleware\ChangeLocale` middleware is included in the default middleware stack. It:

1. Reads the locale from the `locale_key` query parameter.
2. Validates it against the configured `locales` list.
3. Stores the selected locale in the session.
4. Applies the locale for the current request.

No additional setup is needed for locale switching to work. If you want custom locale switching logic, replace this middleware with your own in the `middleware` config array.

### Using translations in your code

Use Laravel's standard translation helpers:

```php
// In resource title
public function getTitle(): string
{
    return __('Clients');
}

// In pages or fields
Text::make(__('moonshine::ui.name'), 'name')
```

Translation files follow the standard Laravel vendor structure at `lang/vendor/moonshine/{locale}/`.

---

## Troubleshooting

### Images are not displayed

1. Make sure you ran the command `php artisan storage:link`.
2. Ensure that the default disk is set to `public`, not `local`.
3. Check that `APP_URL` in the `.env` file is set correctly.

```ini
APP_URL=https://moonshine.test:8080
```

### Problems with HTTPS

If forms use URLs with `http` but expect `https`:

1. Make sure you have a valid SSL certificate.
2. In the `TrustProxies` middleware, set `protected $proxies = ['*']`.

### Error "Page not found"

1. Check for the presence of `MoonShineServiceProvider` in `bootstrap/providers.php` or in `config/app.php`. Some packages (e.g., Apiato) change the provider structure, so MoonShine cannot be added automatically -- add it manually.
2. Ensure that the resource or page is declared in `MoonShineServiceProvider`.
3. Clear the cache:

```shell
php artisan cache:clear
php artisan config:clear
php artisan route:clear
```

### Resource not showing in menu

1. Verify the resource is registered in `MoonShineServiceProvider`:

```php
$core->resources([
    PostResource::class,
]);
```

2. If using auto-menu loading, ensure the resource's `getTitle()` method returns a non-empty string.

### Migrations not running

1. Check that `use_migrations` is set to `true` in `config/moonshine.php` or via `$config->useMigrations()` in the service provider.
2. Run migrations manually:

```shell
php artisan migrate
```

### Assets not loading

1. Re-publish assets:

```shell
php artisan moonshine:publish assets
```

2. Ensure the `public/vendor/moonshine/` directory exists and is accessible.
3. Check that `storage:link` has been executed:

```shell
php artisan storage:link
```
