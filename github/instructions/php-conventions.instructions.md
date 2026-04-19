---
applyTo: "**/*.php"
---

# PHP Conventions



## File Structure

### Namespace & PSR-4
- Standard PSR-4 autoloading under `App\` namespace
- Single class per file
- Namespace matches directory structure exactly

### Import Ordering
```php
// 1. PHP built-in classes
use Throwable;

// 2. Laravel/framework classes
use Illuminate\Support\Facades\DB;
use Illuminate\Http\Request;

// 3. Application classes
use App\Models\User;
use App\Services\ExampleService;
```

### Strict Types
```php
// If enabled:
<?php

declare(strict_types=1);

namespace App\...;

// If disabled:
<?php

namespace App\...;
```

---

## Naming Rules

### Methods
```php
public function getUserById($id) { ... }
public function sendMessage(Request $request) { ... }

### Variables
```php
// If camelCase:
$userId = auth()->id();
$chatHistory = ChatHistory::find($id);
```

### Variables
```php
$userId = auth()->id();
$chatHistory = ChatHistory::find($id);
```

### Constants & Enums
```php
// PHP 8.1+ backed enums
enum StatusEnum: int
{
    case ACTIVE = 1;
    case INACTIVE = 0;

    public function label(): string
    {
        return match($this) {
            self::ACTIVE => 'Active',
            self::INACTIVE => 'Inactive',
        };
    }
}

// Class constants — always UPPER_CASE
private const MAX_RETRIES = 3;
```

---

## Type Hints

### If "Always"
```php
public function getUser(int $id): User
{
    return User::findOrFail($id);
}
```

### If "Selective"
```php
// Type hints on Form Requests and service methods with complex signatures
public function authorize(): bool { ... }
public function rules(): array { ... }
public static function process(User|Admin $actor): Message { ... }

// Optional on simple controller methods
public function index() { ... }
```

---

## Error Handling

### Database Transactions
```php
DB::beginTransaction();
try {
    // operations...
    DB::commit();
} catch (\Throwable $th) {
    DB::rollBack();
    // handle error
}
```

### Service-Level Try/Catch
```php
try {
    return Http::get($url)->throw()->json();
} catch (\Exception $exception) {
    \Log::error("Operation failed: {$exception->getMessage()}");
    return [];
}
```

---

## Dependency Resolution

### If Facades
```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Redis;

DB::beginTransaction();
Cache::remember('key', 3600, fn() => Model::all());
```

### If Constructor Injection
```php
public function __construct(
    private readonly UserRepository $users,
    private readonly CacheManager $cache,
) {}
```

---

## Common Patterns

### Null Coalescing
```php
$value = $data['key'] ?? 'default';
```

### Array Manipulation
```php
$ids = array_values(array_unique($ids));
$names = array_map(fn($item) => $item['name'], $items);
```

### Collection Operations
```php
$items->pluck('id')->toArray();
$items->firstWhere('id', $value);
$items->contains('id', $value);
```

### Conditional Query Building
```php
$query->when(isset($data['status']), function ($q) use ($data) {
    $q->where('status', $data['status']);
});
```

---

## Do's and Don'ts

### ✅ Do
- Follow the chosen naming convention consistently across all files
- Use PHP 8.1+ backed enums for type-safe constants
- Log errors with contextual information
- Use `DB::beginTransaction()` for multi-step writes

### ❌ Don't
- Don't mix naming conventions (e.g., camelCase in some files, snake_case in others)
- Don't catch exceptions silently without logging
- Don't use `@` error suppression operator
- Don't use `eval()` or dynamic class instantiation from user input
