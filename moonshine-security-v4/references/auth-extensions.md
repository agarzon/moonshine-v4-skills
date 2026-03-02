# Auth Extensions Reference (v4)

Detailed code examples for MoonShine v4 authentication and authorization extensions: custom auth pipelines, policies, Socialite, 2FA, custom auth responses, role-based middleware, authorization rules, and user field configuration.

---

## Custom Authentication Pipeline -- PhoneVerification

Complete pipeline class that redirects users with a phone number to an SMS challenge page before completing login:

```php
namespace App\MoonShine\AuthPipelines;

use Closure;
use Illuminate\Http\Request;
use MoonShine\Laravel\Models\MoonshineUser;

class PhoneVerification
{
    public function handle(Request $request, Closure $next)
    {
        $user = MoonshineUser::query()
            ->where('email', $request->get('username'))
            ->first();

        if (! is_null($user) && ! is_null($user->getAttribute('phone'))) {
            $request->session()->put([
                'login.id' => $user->getKey(),
                'login.remember' => $request->boolean('remember'),
            ]);

            return redirect()->route('sms-challenge');
        }

        return $next($request);
    }
}
```

### IP Whitelist Pipeline

```php
namespace App\MoonShine\AuthPipelines;

use Closure;
use Illuminate\Http\Request;

class CheckIpWhitelist
{
    public function handle(Request $request, Closure $next)
    {
        $allowedIps = config('moonshine.allowed_ips', []);

        if (! empty($allowedIps) && ! in_array($request->ip(), $allowedIps, true)) {
            abort(403, 'IP not whitelisted.');
        }

        return $next($request);
    }
}
```

### Audit Logging Pipeline

```php
namespace App\MoonShine\AuthPipelines;

use Closure;
use Illuminate\Http\Request;

class LogSuccessfulAttempt
{
    public function handle(Request $request, Closure $next)
    {
        logger()->info('MoonShine login attempt', [
            'username' => $request->get('username'),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
        ]);

        return $next($request);
    }
}
```

### Registering multiple pipelines

Pipeline order matters -- they execute sequentially:

```php
// config/moonshine.php
'auth' => [
    'pipelines' => [
        \App\MoonShine\AuthPipelines\CheckIpWhitelist::class,
        \MoonShine\TwoFactor\TwoFactorAuthPipe::class,
        \App\MoonShine\AuthPipelines\LogSuccessfulAttempt::class,
    ],
],
```

---

## Complete PostPolicy Example

A policy with all 8 MoonShine-compatible methods. The first type-hint on each method should match your configured auth model (default: `MoonshineUser`):

```php
namespace App\Policies;

use App\Models\Post;
use Illuminate\Auth\Access\HandlesAuthorization;
use MoonShine\Laravel\Models\MoonshineUser;

class PostPolicy
{
    use HandlesAuthorization;

    /**
     * Determine whether the user can view the index page.
     */
    public function viewAny(MoonshineUser $user): bool
    {
        return true;
    }

    /**
     * Determine whether the user can view the detail page.
     */
    public function view(MoonshineUser $user, Post $model): bool
    {
        return true;
    }

    /**
     * Determine whether the user can create records.
     */
    public function create(MoonshineUser $user): bool
    {
        return $user->hasRole('admin') || $user->hasRole('editor');
    }

    /**
     * Determine whether the user can edit records.
     */
    public function update(MoonshineUser $user, Post $model): bool
    {
        return $user->hasRole('admin') || $model->author_id === $user->id;
    }

    /**
     * Determine whether the user can delete records.
     */
    public function delete(MoonshineUser $user, Post $model): bool
    {
        return $user->hasRole('admin');
    }

    /**
     * Determine whether the user can mass delete records.
     */
    public function massDelete(MoonshineUser $user): bool
    {
        return $user->hasRole('admin');
    }

    /**
     * Determine whether the user can restore soft-deleted records.
     */
    public function restore(MoonshineUser $user, Post $model): bool
    {
        return $user->hasRole('admin');
    }

    /**
     * Determine whether the user can permanently delete records.
     */
    public function forceDelete(MoonshineUser $user, Post $model): bool
    {
        return $user->hasRole('admin');
    }
}
```

Enable on the resource:

```php
namespace App\MoonShine\Resources;

use MoonShine\Laravel\Resources\ModelResource;

class PostResource extends ModelResource
{
    protected bool $withPolicy = true;

    protected string $model = \App\Models\Post::class;
}
```

Generate a policy scaffold:

```shell
php artisan moonshine:policy PostPolicy
```

---

## Complete Socialite Setup

### Step 1: Install packages

```shell
composer require laravel/socialite
composer require moonshine/socialite
php artisan migrate
php artisan vendor:publish --provider="MoonShine\Socialite\Providers\SocialiteServiceProvider"
```

### Step 2: Configure Laravel Socialite drivers

```php
// config/services.php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => env('GITHUB_REDIRECT_URL'),
],
'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_REDIRECT_URL'),
],
```

### Step 3: Configure MoonShine Socialite drivers

```php
// config/moonshine-socialite.php
return [
    'drivers' => [
        'github' => '/images/github.png',
        'google' => '/images/google.svg',
    ],
];
```

### Step 4: Publish and extend the user model with the trait

```php
namespace App\Models;

use MoonShine\Socialite\Traits\HasMoonShineSocialite;

final class MoonshineUser extends \MoonShine\Laravel\Models\MoonshineUser
{
    use HasMoonShineSocialite;
}
```

### Step 5: Update config to use custom model

```php
// config/moonshine.php
'auth' => [
    'model' => \App\Models\MoonshineUser::class,
],
```

### Step 6: Add SocialAuth component to custom pages (if applicable)

The component is automatically added to the default profile page and LoginLayout. Only add manually if you have overridden those pages:

```php
use MoonShine\Socialite\Components\SocialAuth;

// In a custom login page
protected function components(): iterable
{
    return [
        // ... other components
        SocialAuth::make(), // login mode (default)
    ];
}

// In a custom profile page
protected function components(): iterable
{
    return [
        // ... other components
        SocialAuth::make(profileMode: true), // shows link/unlink buttons
    ];
}
```

---

## Complete 2FA Setup

### Step 1: Install the package

```shell
composer require moonshine/two-factor
php artisan migrate
```

### Step 2: Register the pipeline

Via config:

```php
// config/moonshine.php
return [
    'auth' => [
        'pipelines' => [
            MoonShine\TwoFactor\TwoFactorAuthPipe::class,
        ],
    ],
];
```

Via service provider:

```php
$config->authPipelines([
    MoonShine\TwoFactor\TwoFactorAuthPipe::class,
]);
```

### Step 3: Add the trait to the user model

```php
namespace App\Models;

use MoonShine\TwoFactor\Traits\TwoFactorAuthenticatable;

final class MoonshineUser extends \MoonShine\Laravel\Models\MoonshineUser
{
    use TwoFactorAuthenticatable;
}
```

### Step 4: Update config to use custom model

```php
// config/moonshine.php
'auth' => [
    'model' => App\Models\MoonshineUser::class,
],
```

### Step 5: Add TwoFactor component to custom profile page (if applicable)

The component is automatically added to the default profile page. Only add manually if you have a custom profile page:

```php
use MoonShine\TwoFactor\ComponentSets\TwoFactor;

protected function components(): iterable
{
    return [
        // ... other components
        TwoFactor::make(),
    ];
}
```

### Combining Socialite and 2FA

When using both packages, add both traits (`HasMoonShineSocialite` + `TwoFactorAuthenticatable`) to the user model.

---

## authenticateUsing / logoutUsing (NEW in v4)

Configured in `MoonShineServiceProvider`. Callback receives `$default` closure for standard behavior.

```php
use Closure;
use Symfony\Component\HttpFoundation\Response;

$config->authenticateUsing(function (Closure $default): Response {
    logger()->info('Admin logged in', ['user_id' => auth()->id(), 'ip' => request()->ip()]);
    return $default(); // or redirect()->to('/custom-dashboard')
});

$config->logoutUsing(function (Closure $default): Response {
    logger()->info('Admin logged out');
    return redirect()->to('/'); // redirect to main site instead of login
});
```

---

## authorizationRules -- Full Example

Global authorization rules apply across all resources. They run alongside (not instead of) resource-level policies.

```php
// app/Providers/MoonShineServiceProvider.php
namespace App\Providers;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\ServiceProvider;
use MoonShine\Contracts\Core\DependencyInjection\ConfiguratorContract;
use MoonShine\Contracts\Core\DependencyInjection\CoreContract;
use MoonShine\Contracts\Core\ResourceContract;
use MoonShine\Laravel\DependencyInjection\MoonShine;
use MoonShine\Laravel\DependencyInjection\MoonShineConfigurator;
use MoonShine\Support\Enums\Ability;

class MoonShineServiceProvider extends ServiceProvider
{
    /**
     * @param  MoonShine  $core
     * @param  MoonShineConfigurator  $config
     */
    public function boot(
        CoreContract $core,
        ConfiguratorContract $config,
    ): void {
        // Deny all delete operations for non-superadmins
        $config->authorizationRules(
            static function (ResourceContract $resource, Model $user, Ability $ability, Model $item): bool {
                if ($ability === Ability::DELETE && ! $user->is_superadmin) {
                    return false;
                }

                if ($ability === Ability::FORCE_DELETE && ! $user->is_superadmin) {
                    return false;
                }

                if ($ability === Ability::MASS_DELETE && ! $user->is_superadmin) {
                    return false;
                }

                return true;
            }
        );
    }
}
```

### Ability enum values (v4 namespace)

```php
use MoonShine\Support\Enums\Ability;

// Available abilities:
Ability::CREATE;       // 'create'
Ability::VIEW;         // 'view'
Ability::VIEW_ANY;     // 'viewAny'
Ability::UPDATE;       // 'update'
Ability::DELETE;       // 'delete'
Ability::MASS_DELETE;  // 'massDelete'
Ability::RESTORE;      // 'restore'
Ability::FORCE_DELETE; // 'forceDelete'
```

---

## Custom CheckAdminRole Middleware -- Full Example

### Basic role check

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAdminRole
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->hasRole('admin')) {
            abort(403, 'Access denied.');
        }

        return $next($request);
    }
}
```

### Multi-role access middleware

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckPanelAccess
{
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();

        if (! $user || ! $user->hasAnyRole(['admin', 'editor', 'moderator'])) {
            abort(403, 'Access denied.');
        }

        return $next($request);
    }
}
```

### Registering middleware in config

```php
// config/moonshine.php
'middleware' => [
    \App\Http\Middleware\CheckAdminRole::class,
],
```

---

## Custom User Field Mapping Examples

### Via config

```php
// config/moonshine.php
'user_fields' => [
    'username' => 'login',      // Login field column name
    'password' => 'pass',       // Password field column name
    'name' => 'full_name',      // Display name column name
    'avatar' => 'profile_image', // Avatar image column name
],
```

### Via MoonShineServiceProvider

```php
$config
    ->userField('username', 'login')
    ->userField('password', 'pass')
    ->userField('name', 'full_name')
    ->userField('avatar', 'profile_image');
```

### Complete custom user model example

```php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class Admin extends Authenticatable
{
    protected $table = 'admins';
    protected $fillable = ['login', 'pass', 'full_name', 'profile_image'];
    protected $hidden = ['pass', 'remember_token'];

    public function getAuthPassword(): string
    {
        return $this->pass;
    }
}
```

With matching config:

```php
// config/moonshine.php
'auth' => ['model' => App\Models\Admin::class],
'user_fields' => [
    'username' => 'login',
    'password' => 'pass',
    'name' => 'full_name',
    'avatar' => 'profile_image',
],
```

---

## Custom Profile Page Replacement

```php
// config/moonshine.php
'pages' => [
    'profile' => App\MoonShine\Pages\CustomProfile::class,
],

// Or via ServiceProvider:
$config->changePage(
    \MoonShine\Laravel\Pages\ProfilePage::class,
    \App\MoonShine\Pages\CustomProfile::class
);
```
