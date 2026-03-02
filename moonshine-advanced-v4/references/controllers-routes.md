# Controllers and Routes Reference (MoonShine v4)

## Controller Generation

```shell
php artisan moonshine:controller CustomController
```

Signature:

```
moonshine:controller {className?} {--base-dir=} {--base-namespace=}
```

Creates a controller in `app/MoonShine/Controllers/`. Inheriting from `MoonShineController` is optional but provides convenient helper methods.

---

## Show Blade View Inside MoonShine Layout

```php
namespace App\MoonShine\Controllers;

use MoonShine\Contracts\Core\PageContract;
use MoonShine\Laravel\Http\Controllers\MoonShineController;

final class CustomViewController extends MoonShineController
{
    public function __invoke(): PageContract
    {
        return $this->view(
            'path_to_blade',
            ['param' => 'value']
        );
    }
}
```

---

## Display a MoonShine Page

```php
namespace App\MoonShine\Controllers;

use App\MoonShine\Pages\MyPage;
use MoonShine\Laravel\Http\Controllers\MoonShineController;

final class CustomViewController extends MoonShineController
{
    public function __invoke(MyPage $page): MyPage
    {
        return $page;
    }
}
```

---

## Toast Notification from Controller

The `toast()` method triggers a standard toast notification. In v4, this uses the global `toast()` helper internally.

```php
namespace App\MoonShine\Controllers;

use MoonShine\Laravel\Http\Controllers\MoonShineController;
use MoonShine\Support\Enums\ToastType;
use Symfony\Component\HttpFoundation\Response;

final class CustomViewController extends MoonShineController
{
    public function __invoke(): Response
    {
        $this->toast('Hello world', ToastType::SUCCESS);

        return back();
    }
}
```

---

## Send Persistent Notification

```php
namespace App\MoonShine\Controllers;

use MoonShine\Laravel\Http\Controllers\MoonShineController;
use Symfony\Component\HttpFoundation\Response;

final class CustomViewController extends MoonShineController
{
    public function __invoke(): Response
    {
        $this->notification('Message');

        return back();
    }
}
```

---

## Access Page or Resource from Request

In v4, use `CrudRequestContract` instead of `MoonShineRequest`:

```php
namespace App\MoonShine\Controllers;

use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;
use MoonShine\Laravel\Http\Controllers\MoonShineController;
use Symfony\Component\HttpFoundation\Response;

final class CustomViewController extends MoonShineController
{
    public function __invoke(CrudRequestContract $request)
    {
        $page = $request->getPage();
        $resource = $request->getResource();
    }
}
```

---

## JSON Response from Controller

```php
namespace App\MoonShine\Controllers;

use MoonShine\Laravel\Http\Controllers\MoonShineController;
use Symfony\Component\HttpFoundation\Response;

final class CustomViewController extends MoonShineController
{
    public function __invoke(): Response
    {
        return $this->json(message: 'Message', data: []);
    }
}
```

---

## Routes Overview

MoonShine uses standard Laravel routing. All displayed pages are rendered through `PageController`:

```php
public function __invoke(CrudRequestContract $request): PageContract
{
    $request->getResource();
    $page = $request->getPage()->checkUrl();
    return $page;
}
```

For CRUD pages, route parameters `resourceUri` and `pageUri` are required. `resourceUri` is optional when the page has no resource.

---

## Standard Route Example

```php
Route::get('/admin/resource/{resourceUri}/{pageUri}', CustomController::class)
    ->middleware(['moonshine', \MoonShine\Laravel\Http\Middleware\Authenticate::class])
    ->name('moonshine.name');
```

The `resource` prefix can be changed or removed through configuration settings.

---

## Route::moonshine() Directive

For quick implementation, use the `Route::moonshine()` directive:

```php
Route::moonshine(static function (Router $router) {
    $router->post(
        'permissions/{resourceItem}',
        PermissionController::class
    )->name('permissions');
}, withResource: true, withPage: true, withAuthenticate: true);

// Result:
// POST /admin/resource/{resourceUri}/{pageUri}/permissions/{resourceItem}
// middleware: moonshine, Authenticate::class
```

---

## Route Retrieval

From resource context:

```php
$this->getRoute('permissions')
```

Outside resource:

```php
route('moonshine.permissions', [
    'resourceUri' => 'user-resource',
    'pageUri' => 'custom-page',
])
```

---

## Route Options

```php
Route::moonshine(static function (Router $router) {
    // ...
},
// add prefix {resourceUri}
withResource: false,
// add prefix {pageUri}
withPage: false,
// add middleware Authenticate::class
withAuthenticate: false
);
```

---

## Creating routes/moonshine.php

The best practice is to create `routes/moonshine.php` and declare your custom routes inside.

When creating this file, remember to register it in your application (e.g., via the service provider or `bootstrap/app.php`).

> WARNING: You cannot use middleware groups `web` and `moonshine` simultaneously, as they both start sessions at the same time.

---

## Complete Controller + Route Example

### Step 1: Create the controller

```shell
php artisan moonshine:controller OrderExportController
```

### Step 2: Implement the controller

```php
namespace App\MoonShine\Controllers;

use App\Exports\OrdersExport;
use MoonShine\Contracts\Core\DependencyInjection\CrudRequestContract;
use MoonShine\Laravel\Http\Controllers\MoonShineController;
use Symfony\Component\HttpFoundation\Response;

final class OrderExportController extends MoonShineController
{
    public function __invoke(CrudRequestContract $request): Response
    {
        $resource = $request->getResource();

        // Perform custom export logic
        // ...

        $this->toast('Export started', \MoonShine\Support\Enums\ToastType::SUCCESS);

        return back();
    }
}
```

### Step 3: Register the route

```php
// routes/moonshine.php
use App\MoonShine\Controllers\OrderExportController;

Route::moonshine(static function (Router $router) {
    $router->get('export', OrderExportController::class)
        ->name('order-export');
}, withResource: true, withPage: true, withAuthenticate: true);
```

### Step 4: Create an ActionButton to trigger it

```php
use MoonShine\UI\Components\ActionButton;

ActionButton::make('Export Orders', $this->getRoute('order-export'))
    ->icon('arrow-down-tray');
```

---

## Artisan Commands Quick Reference

| Command | Description |
|---------|-------------|
| `php artisan moonshine:controller` | Create a controller |
| `php artisan moonshine:handler` | Create a handler |
| `php artisan moonshine:resource` | Create a resource (ModelResource, CrudResource, or Blank) |
| `php artisan moonshine:page` | Create a page |
| `php artisan moonshine:layout` | Create a layout |
| `php artisan moonshine:component` | Create a custom component |
| `php artisan moonshine:field` | Create a custom field |
| `php artisan moonshine:type-cast` | Create a TypeCast class |
| `php artisan moonshine:policy` | Create a policy tied to admin user |
| `php artisan moonshine:apply` | Create an apply class |
| `php artisan moonshine:publish` | Publish assets, resources, forms, or pages |
| `php artisan moonshine:resources` | List all registered resources |
| `php artisan moonshine:pages` | List all standalone pages |
