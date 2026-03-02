# Dashboard, Profile, Menu, and Breadcrumbs Recipes (MoonShine v4)

## Dashboard Settings Form

Save settings directly from the dashboard using `asyncMethod`.

```php
use MoonShine\Crud\JsonResponse;
use MoonShine\Laravel\TypeCasts\ModelCaster;
use MoonShine\UI\Components\FormBuilder;

private function getSetting(): Setting
{
    return Setting::query()->find(1);
}

public function store(): JsonResponse
{
    $this->form()->apply(fn(Setting $item) => $item->save());

    return JsonResponse::make()->toast('Saved');
}

private function form(): FormBuilder
{
    return FormBuilder::make()
        ->asyncMethod('store')
        ->fillCast($this->getSetting(), new ModelCaster(Setting::class))
        ->fields([
            // Your fields here
        ]);
}

protected function components(): iterable
{
    yield $this->form();
}
```

---

## Async Metrics with Date Filters

Metrics that update based on form date parameters, using Fragment for async reloading.

```php
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Components\Fragment;
use MoonShine\UI\Components\Layout\Flex;
use MoonShine\UI\Components\Metrics\Wrapped\LineChartMetric;
use MoonShine\UI\Fields\Date;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

protected function components(): iterable
{
    $startDate = request()->date('_data.start_date');
    $endDate = request()->date('_data.end_date');

    return [
        FormBuilder::make()
            ->dispatchEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'metrics'))
            ->fields([
                Flex::make([
                    Date::make('Start date'),
                    Date::make('End date'),
                ]),
            ]),

        Fragment::make([
            FlexibleRender::make("$startDate - $endDate"),

            LineChartMetric::make('Orders')
                ->line([
                    'Profit' => Order::query()
                        ->selectRaw('SUM(price) as sum, DATE_FORMAT(created_at, "%d.%m.%Y") as date')
                        ->whereBetween('created_at', [$startDate, $endDate])
                        ->groupBy('date')
                        ->pluck('sum', 'date')
                        ->toArray(),
                ])
                ->line([
                    'Avg' => Order::query()
                        ->selectRaw('AVG(price) as avg, DATE_FORMAT(created_at, "%d.%m.%Y") as date')
                        ->whereBetween('created_at', [$startDate, $endDate])
                        ->groupBy('date')
                        ->pluck('avg', 'date')
                        ->toArray(),
                ], '#EC4176'),
        ])->name('metrics'),
    ];
}
```

---

## Custom Profile (Personal Account with User Model)

This recipe shows MoonShine used not as an admin panel, but as a personal account with login, registration, password recovery, and profile.

### Routes

```php
use App\Http\Controllers\AuthenticateController;
use App\Http\Controllers\ForgotController;
use App\Http\Controllers\ProfileController;
use App\Http\Controllers\RegisterController;
use App\MoonShine\Pages\ResetPasswordPage;
use Illuminate\Support\Facades\Route;

Route::controller(AuthenticateController::class)->group(function () {
    Route::get('/login', 'form')->middleware('guest')->name('login');
    Route::post('/login', 'authenticate')->middleware('guest')->name('authenticate');
    Route::delete('/logout', 'logout')->middleware('auth')->name('logout');
});

Route::controller(ForgotController::class)->middleware('guest')->group(function () {
    Route::get('/forgot', 'form')->name('forgot');
    Route::post('/forgot', 'reset');
    Route::get('/reset-password/{token}', static fn (ResetPasswordPage $page) => $page)->name('password.reset');
    Route::post('/reset-password', 'updatePassword')->name('password.update');
});

Route::controller(RegisterController::class)->middleware('guest')->group(function () {
    Route::get('/register', 'form')->name('register');
    Route::post('/register', 'store')->name('register.store');
});

Route::controller(ProfileController::class)->middleware('auth')->prefix('profile')->group(function () {
    Route::get('/', 'index')->name('profile');
    Route::post('/', 'update')->name('profile.update');
});
```

### FormLayout (for auth pages)

```php
namespace App\MoonShine\Layouts;

use MoonShine\Laravel\Layouts\AppLayout;
use MoonShine\UI\Components\{Components, Layout\Div, Layout\Body, Layout\Html, Layout\Layout};

final class FormLayout extends AppLayout
{
    protected function getHomeUrl(): string
    {
        return route('home');
    }

    public function build(): Layout
    {
        return Layout::make([
            Html::make([
                $this->getHeadComponent(),
                Body::make([
                    Div::make([
                        Div::make([$this->getLogoComponent()])->class('authentication-logo'),
                        Div::make([
                            Flash::make(),
                            Components::make($this->getPage()->getComponents()),
                        ])->class('authentication-content'),
                    ])->class('authentication'),
                ]),
            ])
                ->customAttributes(['lang' => $this->getHeadLang()])
                ->withAlpineJs()
                ->withThemes(),
        ]);
    }
}
```

### LoginPage

```php
namespace App\MoonShine\Pages;

use App\MoonShine\Layouts\FormLayout;
use MoonShine\Laravel\Pages\Page;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Components\FormBuilder;
use MoonShine\UI\Components\Layout\{Divider, Flex};
use MoonShine\UI\Components\Link;
use MoonShine\UI\Fields\{Password, Switcher, Text};

class LoginPage extends Page
{
    protected ?string $layout = FormLayout::class;

    protected function components(): iterable
    {
        return [
            FormBuilder::make()
                ->class('authentication-form')
                ->action(route('authenticate'))
                ->fields([
                    Text::make('E-mail', 'email')->required()
                        ->customAttributes(['autofocus' => true, 'autocomplete' => 'username']),
                    Password::make(__('Password'), 'password')->required(),
                    Switcher::make(__('Remember me'), 'remember'),
                ])->submit(__('Log in'), ['class' => 'btn-primary btn-lg w-full']),

            Divider::make(),

            Flex::make([
                ActionButton::make(__('Create account'), route('register'))->primary(),
                Link::make(route('forgot'), __('Forgot password'))
            ])->justifyAlign('start'),
        ];
    }
}
```

### AuthenticateController

```php
namespace App\Http\Controllers;

use App\MoonShine\Pages\LoginPage;
use Illuminate\Http\RedirectResponse;

final class AuthenticateController extends Controller
{
    public function form(LoginPage $page): LoginPage
    {
        return $page;
    }

    public function authenticate(AuthenticateFormRequest $request): RedirectResponse
    {
        if (!auth()->attempt($request->validated())) {
            return back()->withErrors(['email' => __('moonshine::auth.failed')]);
        }

        return redirect()->intended(route('profile'));
    }

    public function logout(Guard $guard, Request $request): RedirectResponse
    {
        $guard->logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect()->intended(url()->previous() ?? route('home'));
    }
}
```

### ProfileController

```php
namespace App\Http\Controllers;

use App\MoonShine\Pages\ProfilePage;
use Illuminate\Container\Attributes\CurrentUser;
use Illuminate\Support\Facades\Hash;

final class ProfileController extends Controller
{
    public function index(ProfilePage $page): ProfilePage
    {
        return $page;
    }

    public function update(ProfileFormRequest $request, #[CurrentUser] User $user): RedirectResponse
    {
        $data = $request->only(['email', 'name']);

        if ($request->filled('password')) {
            $data['password'] = Hash::make($request->input('password'));
        }

        $user->update($data);

        return to_route('profile');
    }
}
```

---

## Menu Authorization

### 1. Using Gate Facade

```php
use Illuminate\Support\Facades\Gate;
use MoonShine\Support\Enums\Ability;
use MoonShine\MenuManager\MenuItem;

protected function menu(): array
{
    return [
        MenuItem::make(MoonShineUserRoleResource::class)
            ->canSee(fn() => Gate::check(Ability::VIEW_ANY, MoonshineUserRole::class)),
    ];
}
```

### 2. Using Resource

```php
use MoonShine\Support\Enums\Ability;

protected function menu(): array
{
    return [
        MenuItem::make(MoonShineUserRoleResource::class)
            ->canSee(fn(MenuItem $item) => $item->getFiller()->can(Ability::VIEW_ANY)),
    ];
}
```

### 3. Without Policy (role check)

```php
protected function menu(): array
{
    $menu = [
        MenuItem::make(ArticleResource::class),
    ];

    if (request()->user()->isSuperUser()) {
        $menu[] = MenuItem::make(MoonShineUserResource::class);
    }

    return $menu;
}
```

---

## Custom Breadcrumbs from Resource

Change breadcrumbs of a page directly from the resource:

```php
class MoonShineUserResource extends ModelResource
{
    protected function onLoad(): void
    {
        parent::onLoad();

        $this->getFormPage()->breadcrumbs([
            '/custom' => 'Custom',
            '#' => $this->getTitle(),
        ]);
    }
}
```
