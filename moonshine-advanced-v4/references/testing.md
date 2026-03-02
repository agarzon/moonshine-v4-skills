# Testing Reference (MoonShine v4)

## Creating a Resource with a Test File

By adding the `--test` flag to the `moonshine:resource` command, you can generate a test file along with a basic set of tests:

```shell
php artisan moonshine:resource PostResource --test
```

This creates `tests/Feature/PostResourceTest.php`. For Pest, use `--pest`:

```shell
php artisan moonshine:resource PostResource --pest
```

---

## Setting Up an Authenticated User

MoonShine resources use the `moonshine` guard. Set up the authenticated user in your test's `setUp()` method:

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use MoonShine\Laravel\Models\MoonshineUser;
use Tests\TestCase;

class PostResourceTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();

        $user = MoonshineUser::factory()->create();

        $this->be($user, 'moonshine');
    }
}
```

---

## Testing Index Page

```php
public function test_index_page_successful(): void
{
    $response = $this->get(
        $this->getResource()->getIndexPageUrl()
    )->assertSuccessful();
}
```

---

## Testing Form Page

```php
public function test_form_page_successful(): void
{
    $response = $this->get(
        $this->getResource()->getFormPageUrl()
    )->assertSuccessful();
}
```

---

## Testing Detail Page

```php
public function test_detail_page_successful(): void
{
    $item = Post::factory()->create();

    $response = $this->get(
        $this->getResource()->getDetailPageUrl($item->getKey())
    )->assertSuccessful();
}
```

---

## Testing Resource CRUD Operations

### Test Creating a Record

```php
public function test_store_successful(): void
{
    $data = [
        'title' => 'Test Post',
        'content' => 'Test content',
        'active' => true,
    ];

    $response = $this->post(
        $this->getResource()->getFormPageUrl(),
        $data
    );

    $response->assertRedirect();

    $this->assertDatabaseHas('posts', [
        'title' => 'Test Post',
    ]);
}
```

### Test Updating a Record

```php
public function test_update_successful(): void
{
    $post = Post::factory()->create();

    $data = [
        'title' => 'Updated Title',
        'content' => $post->content,
    ];

    $response = $this->put(
        $this->getResource()->getFormPageUrl($post->getKey()),
        $data
    );

    $response->assertRedirect();

    $this->assertDatabaseHas('posts', [
        'id' => $post->id,
        'title' => 'Updated Title',
    ]);
}
```

### Test Deleting a Record

```php
public function test_delete_successful(): void
{
    $post = Post::factory()->create();

    $response = $this->delete(
        $this->getResource()->getFormPageUrl($post->getKey())
    );

    $response->assertRedirect();

    $this->assertDatabaseMissing('posts', [
        'id' => $post->id,
    ]);
}
```

---

## Testing with Custom Guard

If you use a custom authentication guard for MoonShine:

```php
protected function setUp(): void
{
    parent::setUp();

    $user = MoonshineUser::factory()->create([
        'moonshine_user_role_id' => 1,  // Admin role
    ]);

    $this->actingAs($user, 'moonshine');
}
```

---

## Testing Handlers

```php
public function test_custom_handler(): void
{
    $handler = new MyCustomHandler();

    // Test handler logic directly
    MyCustomHandler::process();

    // Or test via HTTP
    $response = $this->get($handler->getUrl());
    $response->assertRedirect();
}
```

---

## Testing AsyncMethod

```php
public function test_async_method(): void
{
    $post = Post::factory()->create();

    $resource = app(PostResource::class);
    $url = $resource->getAsyncMethodUrl('myAction');

    $response = $this->post($url, [
        'resourceItem' => $post->getKey(),
    ]);

    $response->assertSuccessful();
}
```

---

## Testing Controllers

```php
public function test_custom_controller(): void
{
    $response = $this->get(route('moonshine.custom-action', [
        'resourceUri' => 'post-resource',
        'pageUri' => 'index-page',
    ]));

    $response->assertSuccessful();
}
```

---

## Testing with Pest

```php
use MoonShine\Laravel\Models\MoonshineUser;

beforeEach(function () {
    $user = MoonshineUser::factory()->create();
    $this->be($user, 'moonshine');
});

it('can view index page', function () {
    $this->get($this->getResource()->getIndexPageUrl())
        ->assertSuccessful();
});

it('can create a post', function () {
    $this->post($this->getResource()->getFormPageUrl(), [
        'title' => 'Test Post',
        'content' => 'Test content',
    ])->assertRedirect();

    $this->assertDatabaseHas('posts', [
        'title' => 'Test Post',
    ]);
});

it('can delete a post', function () {
    $post = Post::factory()->create();

    $this->delete($this->getResource()->getFormPageUrl($post->getKey()))
        ->assertRedirect();

    $this->assertDatabaseMissing('posts', [
        'id' => $post->id,
    ]);
});
```

---

## Testing Notifications

```php
use MoonShine\Laravel\Notifications\MoonShineNotification;
use MoonShine\Crud\Notifications\NotificationButton;

public function test_notification_sent(): void
{
    $user = MoonshineUser::factory()->create();

    MoonShineNotification::send(
        message: 'Test notification',
        button: new NotificationButton('Click', '/test'),
        ids: [$user->id]
    );

    $this->assertDatabaseHas('notifications', [
        'notifiable_id' => $user->id,
    ]);
}
```

---

## Testing Toast Responses

```php
public function test_toast_in_response(): void
{
    // When testing controllers that return toast notifications,
    // check for the session flash data
    $response = $this->post(route('moonshine.some-action'));

    $response->assertSessionHas('toast');
}
```

---

## Testing JsonResponse

```php
public function test_json_response(): void
{
    $response = $this->postJson(route('moonshine.async-action'));

    $response->assertSuccessful();
    $response->assertJsonStructure([
        'message',
        'messageType',
    ]);
}
```

---

## Common Test Patterns

### Helper Method for Resource

```php
protected function getResource(): PostResource
{
    return app(PostResource::class);
}
```

### Testing with Different Roles

```php
public function test_admin_can_access(): void
{
    $admin = MoonshineUser::factory()->create([
        'moonshine_user_role_id' => 1,
    ]);
    $this->be($admin, 'moonshine');

    $this->get($this->getResource()->getIndexPageUrl())
        ->assertSuccessful();
}

public function test_editor_cannot_delete(): void
{
    $editor = MoonshineUser::factory()->create([
        'moonshine_user_role_id' => 2,
    ]);
    $this->be($editor, 'moonshine');

    $post = Post::factory()->create();

    $this->delete($this->getResource()->getFormPageUrl($post->getKey()))
        ->assertForbidden();
}
```

### Testing Filters

```php
public function test_filter_by_title(): void
{
    Post::factory()->create(['title' => 'Laravel Guide']);
    Post::factory()->create(['title' => 'PHP Basics']);

    $response = $this->get(
        $this->getResource()->getIndexPageUrl() . '?filters[title]=Laravel'
    );

    $response->assertSuccessful();
    $response->assertSee('Laravel Guide');
    $response->assertDontSee('PHP Basics');
}
```

---

## Test Organization Tips

1. **One test file per resource**: Keep tests focused on a single resource.
2. **Use factories**: Always use model factories for test data.
3. **RefreshDatabase trait**: Use `RefreshDatabase` to ensure clean state.
4. **Test CRUD operations**: At minimum, test index, create, update, and delete.
5. **Test authorization**: Verify that policies are enforced correctly.
6. **Test validation**: Ensure form validation rules work as expected.
