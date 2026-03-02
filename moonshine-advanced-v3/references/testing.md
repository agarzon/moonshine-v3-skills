# Testing MoonShine v3 Resources

## Test Generation

When creating a resource, add the `--test` or `--pest` flag to automatically generate a test file.

### PHPUnit

```shell
php artisan moonshine:resource PostResource --test
```

Generates `tests/Feature/PostResourceTest.php` with a basic test set.

### Pest

```shell
php artisan moonshine:resource PostResource --pest
```

Generates a Pest-style test file.

### Resource Command Full Signature

```
moonshine:resource {className?} {--type=} {--m|model=} {--t|title=} {--test} {--pest} {--p|policy} {--base-dir=} {--base-namespace=}
```

---

## Test Setup and Fixtures

### Authenticated User Setup

MoonShine resource testing follows standard Laravel testing conventions. Authenticate as a MoonShine user using the `moonshine` guard:

```php
use MoonShine\Laravel\Models\MoonshineUser;

protected function setUp(): void
{
    parent::setUp();

    $user = MoonshineUser::factory()->create();

    $this->be($user, 'moonshine');
}
```

Key points:
- Use `MoonshineUser::factory()` to create test admin users.
- The guard name is `moonshine` (not `web`).
- Call `$this->be($user, 'moonshine')` to authenticate all subsequent requests.

---

## Testing Resource Pages

### Index Page Test

```php
public function test_index_page_successful(): void
{
    $response = $this->get(
        $this->getResource()->getIndexPageUrl()
    )->assertSuccessful();
}
```

### Form Page Test

```php
public function test_form_page_successful(): void
{
    $response = $this->get(
        $this->getResource()->getFormPageUrl()
    )->assertSuccessful();
}
```

### Detail Page Test

```php
public function test_detail_page_successful(): void
{
    $item = Post::factory()->create();

    $response = $this->get(
        $this->getResource()->getDetailPageUrl($item->getKey())
    )->assertSuccessful();
}
```

### Testing CRUD Operations

```php
public function test_store(): void
{
    $response = $this->post(
        $this->getResource()->getRoute('crud.store'),
        [
            'title' => 'Test Post',
            'content' => 'Test content',
        ]
    );

    $response->assertRedirect();
    $this->assertDatabaseHas('posts', ['title' => 'Test Post']);
}

public function test_update(): void
{
    $item = Post::factory()->create();

    $response = $this->put(
        $this->getResource()->getRoute('crud.update', $item->getKey()),
        [
            'title' => 'Updated Title',
        ]
    );

    $response->assertRedirect();
    $this->assertDatabaseHas('posts', ['title' => 'Updated Title']);
}

public function test_delete(): void
{
    $item = Post::factory()->create();

    $response = $this->delete(
        $this->getResource()->getRoute('crud.destroy', $item->getKey())
    );

    $response->assertRedirect();
    $this->assertDatabaseMissing('posts', ['id' => $item->getKey()]);
}
```

---

## Resource URL Helpers

MoonShine resources provide built-in URL helpers useful in tests:

| Method | Description |
|--------|-------------|
| `getIndexPageUrl()` | URL of the index (list) page |
| `getFormPageUrl()` | URL of the create form page |
| `getFormPageUrl($key)` | URL of the edit form page for a specific item |
| `getDetailPageUrl($key)` | URL of the detail page for a specific item |
| `getRoute($name, $key?)` | Named route for the resource |

---

## Tips for Testing

1. **Database transactions**: Use `RefreshDatabase` or `DatabaseTransactions` trait to keep tests isolated.
2. **MoonShine guard**: Always specify `'moonshine'` as the guard when authenticating test users.
3. **Factory setup**: Ensure `MoonshineUser` factory is available (comes with MoonShine installation).
4. **Async endpoints**: Test async methods (e.g., `asyncMethod`) by posting to the resource's async method URL with appropriate parameters.
5. **JSON responses**: When testing handlers or async actions that return `MoonShineJsonResponse`, assert on JSON structure:

```php
$response->assertJson([
    'message' => 'Success',
    'message_type' => 'success',
]);
```

---

## Installation Test Mode

When installing MoonShine in CI/CD or test environments, use the `--tests-mode` flag:

```shell
php artisan moonshine:install --tests-mode
```

Or combine with quick mode to skip all dialogs:

```shell
php artisan moonshine:install --tests-mode --quick-mode
```
