---
applyTo: "tests/**"
---

# Testing



## Test Environment

```xml
<!-- phpunit.xml -->
<php>
    <env name="APP_ENV" value="testing"/>
    <env name="CACHE_STORE" value="array"/>
    <env name="DB_CONNECTION" value="sqlite"/>
    <env name="DB_DATABASE" value=":memory:"/>
    <env name="MAIL_MAILER" value="array"/>
    <env name="QUEUE_CONNECTION" value="sync"/>
    <env name="SESSION_DRIVER" value="array"/>
</php>
```

---

## Directory Structure

```
tests/
├── TestCase.php
├── Feature/
│   ├── Admin/
│   │   ├── AuthControllerTest.php
│   │   └── ExampleControllerTest.php
│   └── Client/
│       └── ExampleControllerTest.php
└── Unit/
    ├── Services/
    │   └── ExampleServiceTest.php
    └── Models/
        └── ExampleTest.php
```

---

## Test Class Structure (PHPUnit)

```php
<?php

namespace Tests\Feature\Admin;

use Tests\TestCase;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

class ExampleControllerTest extends TestCase
{
    use RefreshDatabase;

    // Helper for auth setup
    private function authenticate(): array
    {
        $user = User::factory()->create();
        $token = auth()->claims(['meta' => ['type' => 'admin']])->login($user);
        return [$user, $token];
    }

    public function test_authenticated_user_can_list_examples()
    {
        [$user, $token] = $this->authenticate();

        $response = $this->getJson('/api/v1/admin/examples', [
            'Authorization' => "Bearer {$token}"
        ]);

        $response->assertStatus(200)
                 ->assertJsonStructure(['status', 'data']);
    }

    public function test_unauthenticated_user_gets_401()
    {
        $response = $this->getJson('/api/v1/admin/examples');
        $response->assertStatus(401);
    }

    public function test_create_with_invalid_data_gets_422()
    {
        [$user, $token] = $this->authenticate();

        $response = $this->postJson('/api/v1/admin/examples/create', [], [
            'Authorization' => "Bearer {$token}"
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['name']);
    }
}
```

---

## Test Scenario Pattern

For each endpoint, test:

1. **Success** — Valid data + valid auth → expected response
2. **Unauthorized** — No token → 401
3. **Validation** — Invalid/missing fields → 422
4. **Forbidden** — Wrong permissions → 403
5. **Not Found** — Missing resource → 404

---

## HTTP Test Methods

```php
$this->getJson($uri, $headers);
$this->postJson($uri, $data, $headers);
$this->putJson($uri, $data, $headers);
$this->deleteJson($uri, $data, $headers);
```

---

## Assertions

```php
$response->assertStatus(200);
$response->assertJsonStructure(['status', 'data' => ['id', 'name']]);
$response->assertJson(['message' => 'Expected message']);
$response->assertJsonValidationErrors(['field']);
$this->assertDatabaseHas('examples', ['name' => 'Test']);
$this->assertSoftDeleted('examples', ['id' => $id]);
```

---

## Do's and Don'ts

### ✅ Do
- Use `RefreshDatabase` on every database-touching test
- Use factories for test data
- Test both success and failure paths for every endpoint
- Use `assertJsonStructure()` for response shape validation
- Follow consistent test naming conventions
- Mock external HTTP calls — never call real APIs

### ❌ Don't
- Don't share state between test methods
- Don't test with the production database connection
- Don't use `$this->actingAs()` if the app uses JWT — use proper token generation
- Don't write tests that depend on execution order
