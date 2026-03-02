# API Mode, JWT Authentication & OpenAPI Generator -- Reference (v4)

This reference provides complete setup details for MoonShine v4 API mode, JWT token authentication, and the OpenAPI Generator package.

---

## API Mode

### Concept

MoonShine can operate as a pure JSON API backend. Adding the `Accept: application/json` header to any request causes MoonShine CRUD operations to return JSON responses instead of rendered HTML.

### Available CRUD Routes

All resource CRUD operations are available as API endpoints:

```
DELETE /admin/resource/{resourceUri}/crud                    -- Mass delete
GET    /admin/resource/{resourceUri}/crud                    -- Listing
POST   /admin/resource/{resourceUri}/crud                    -- Create
PUT    /admin/resource/{resourceUri}/crud/{resourceItem}     -- Edit
DELETE /admin/resource/{resourceUri}/crud/{resourceItem}     -- Delete
```

### Route Parameters

- `{resourceUri}` -- The URI slug of the resource (e.g., `users`, `posts`). This corresponds to the resource's `uriKey()` method.
- `{resourceItem}` -- The primary key of the specific record.

### Mass Delete

Send a DELETE request to the base CRUD endpoint with an array of IDs:

```
DELETE /admin/resource/users/crud
Content-Type: application/json
Accept: application/json

{
    "ids": [1, 2, 3]
}
```

### Listing with Filters

The GET listing endpoint supports query parameters for filtering, sorting, and pagination as configured on the resource.

```
GET /admin/resource/users/crud?page=2&sort=name&filter[status]=active
Accept: application/json
```

### Disabling Session Middleware

When using MoonShine purely as an API backend (no browser-based admin panel), remove session-related middleware in `config/moonshine.php`:

```php
// config/moonshine.php

return [
    'middleware' => [
        // Remove all session middleware for API-only mode
        // Do NOT include: StartSession, ShareErrorsFromSession, VerifyCsrfToken
    ],
    // ...
];
```

### Example: Creating a Record via API

```php
// Using Laravel HTTP client as an example consumer
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => 'Bearer ' . $token,
])->post('/admin/resource/posts/crud', [
    'title' => 'New Post',
    'content' => 'Post content here',
    'status' => 'published',
]);

$data = $response->json();
```

### Example: Editing a Record via API

```php
$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => 'Bearer ' . $token,
])->put('/admin/resource/posts/crud/42', [
    'title' => 'Updated Title',
]);
```

### Example: Deleting a Single Record

```php
$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => 'Bearer ' . $token,
])->delete('/admin/resource/posts/crud/42');
```

### JsonResponse in Controllers

Return structured JSON responses from custom controller methods:

```php
use MoonShine\Crud\JsonResponse;
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

// Basic success response with toast
return JsonResponse::make()
    ->toast('Record saved!', ToastType::SUCCESS);

// Response with events to trigger on the frontend
return JsonResponse::make()
    ->toast('Updated!', ToastType::SUCCESS)
    ->events([
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'index'),
    ]);
```

> **v4 namespace change:** Use `MoonShine\Crud\JsonResponse` instead of the v3 `MoonShine\Laravel\Http\Responses\MoonShineJsonResponse`.

---

## JWT Authentication

### Overview

The `moonshine/jwt` package provides JWT (JSON Web Token) authentication for MoonShine's API mode. This replaces session-based authentication with stateless token-based auth.

### Step 1: Install the Package

```shell
composer require moonshine/jwt
```

### Step 2: Publish the Configuration

```shell
php artisan vendor:publish --provider="MoonShine\JWT\Providers\JWTServiceProvider"
```

This creates the JWT configuration file.

### Step 3: Set the JWT Secret

Add a base64-encoded secret key to your `.env` file:

```ini
JWT_SECRET=YOUR_BASE64_SECRET_HERE
```

To generate a secure base64 secret:

```shell
php -r "echo base64_encode(random_bytes(32));"
```

### Step 4: Configure MoonShine Middleware

Update `config/moonshine.php` to use JWT middleware and auth pipeline:

```php
use MoonShine\JWT\JWTAuthPipe;
use MoonShine\JWT\Http\Middleware\AuthenticateApi;

return [
    'middleware' => [
        // Empty -- no session middleware needed for API mode
    ],
    'auth' => [
        // ...
        'middleware' => [
            AuthenticateApi::class,
        ],
        'pipelines' => [
            JWTAuthPipe::class,
        ],
    ],
    // ...
];
```

### Step 5: Authenticate and Use Tokens

Send credentials to the authentication endpoint. On success, a JWT token is returned:

```
POST /admin/authenticate
Content-Type: application/json
Accept: application/json

{
    "username": "admin@example.com",
    "password": "secret"
}
```

Response:

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

Use the token in all subsequent requests:

```
GET /admin/resource/users/crud
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Middleware Explained

- **AuthenticateApi** -- Replaces the default web authentication middleware. Validates the JWT token from the `Authorization` header and authenticates the user.
- **JWTAuthPipe** -- Auth pipeline that processes JWT token validation as part of MoonShine's authentication flow.

### Complete JWT Setup Example

Full `config/moonshine.php` for API-only mode with JWT:

```php
use MoonShine\JWT\JWTAuthPipe;
use MoonShine\JWT\Http\Middleware\AuthenticateApi;

return [
    // No session middleware for API-only mode
    'middleware' => [],

    'auth' => [
        'enabled' => true,
        'guard' => 'moonshine',

        'middleware' => [
            AuthenticateApi::class,
        ],

        'pipelines' => [
            JWTAuthPipe::class,
        ],
    ],

    // ... other configuration
];
```

---

## OpenAPI Generator (NEW in v4)

### Overview

The `moonshine/oag` package automatically generates OpenAPI (Swagger) specification files based on your declared MoonShine resources. This provides auto-generated API documentation that stays in sync with your resource definitions.

### Step 1: Install the Package

```shell
composer require moonshine/oag
```

### Step 2: Publish the Configuration

```shell
php artisan vendor:publish --provider="MoonShine\OAG\Providers\OAGServiceProvider"
```

### Step 3: Configuration Options

The published config file (`config/oag.php`) contains the following options:

```php
return [
    // Title displayed in the documentation UI
    'title' => 'Docs',

    // File system path where the YAML specification is stored
    'path' => realpath(resource_path('oag.yaml')),

    // Named route for retrieving the JSON specification
    'route' => 'oag.json',

    // Blade view used to render the documentation page
    'view' => 'oag::docs',
];
```

### Configuration Details

| Option | Default | Description |
|---|---|---|
| `title` | `'Docs'` | Title shown in the documentation viewer |
| `path` | `resource_path('oag.yaml')` | Filesystem path for the generated YAML spec |
| `route` | `'oag.json'` | Named route that serves the JSON spec to the docs viewer |
| `view` | `'oag::docs'` | Blade view for the Swagger/OpenAPI documentation UI |

### Step 4: Generate the Specification

Run the artisan command to generate the specification from your resources:

```shell
php artisan oag:generate
```

This analyzes all declared and configured resources in MoonShine and produces specification files.

### Generated Files

The specification files are created in the `resources` directory by default:

```
resources/
  oag.yaml    -- YAML format specification
  oag.json    -- JSON format specification
```

### Step 5: View the Documentation

Once generated, the documentation is available at:

```
https://your-domain.com/docs
```

The documentation UI renders the OpenAPI spec using the configured view (`oag::docs`).

### Regenerating After Changes

Run the generate command whenever you add or modify resources:

```shell
php artisan oag:generate
```

### Complete OAG Workflow

```shell
# 1. Install the package
composer require moonshine/oag

# 2. Publish configuration
php artisan vendor:publish --provider="MoonShine\OAG\Providers\OAGServiceProvider"

# 3. (Optional) Customize config/oag.php

# 4. Generate the spec
php artisan oag:generate

# 5. Visit /docs in your browser
```

### Customizing the Documentation View

To customize the documentation page appearance, publish the view:

```shell
php artisan vendor:publish --tag=oag-views
```

Then modify the published view as needed. The default view uses a standard Swagger UI rendering.

---

## Combined API + JWT + OAG Setup

For a complete API-first MoonShine installation with JWT auth and auto-generated docs:

```shell
# Install both packages
composer require moonshine/jwt moonshine/oag

# Publish configs
php artisan vendor:publish --provider="MoonShine\JWT\Providers\JWTServiceProvider"
php artisan vendor:publish --provider="MoonShine\OAG\Providers\OAGServiceProvider"
```

Update `.env`:

```ini
JWT_SECRET=YOUR_BASE64_SECRET_HERE
```

Update `config/moonshine.php`:

```php
use MoonShine\JWT\JWTAuthPipe;
use MoonShine\JWT\Http\Middleware\AuthenticateApi;

return [
    'middleware' => [],

    'auth' => [
        'enabled' => true,
        'guard' => 'moonshine',
        'middleware' => [
            AuthenticateApi::class,
        ],
        'pipelines' => [
            JWTAuthPipe::class,
        ],
    ],
];
```

Generate the API documentation:

```shell
php artisan oag:generate
```

Your API is now ready with JWT authentication and auto-generated OpenAPI documentation at `/docs`.
