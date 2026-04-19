---
applyTo: "**/*.php"
---

# PHP Coding Conventions — Project-Wide

## File Structure
- **No `declare(strict_types=1)`** — not used in this project
- **Standard PSR-4 namespace** matching directory structure under `App\`
- **Single class per file**
- **Import ordering:** PHP built-ins → Laravel facades/classes → App classes (no strict enforcement)

## Naming Conventions

### Methods
All method names use **camelCase**:
```php
// ✅ Correct
public function getAllHistories() { ... }
public function sendMessage() { ... }
public function createSystemMessage() { ... }
public static function getPlatformFromCache() { ... }

// ❌ Wrong
public function getAllHistories() { ... }
public function sendMessage() { ... }
```

### Variables
Local variables use **camelCase**:
```php
$adminId = auth()->id();
$chatHistory = ChatHistory::find($id);
$platformIds = $admin->getPlatformIdsWithCertainPermission('view_chat');
$seenMessageIds = [];
```

### Constants & Enums
PHP 8.1+ backed enums with UPPER_CASE cases:
```php
enum ChatStatusEnum: int
{
    case PENDING = 1;
    case OPEN = 2;
    case CLOSED = 3;

    public function label(): string
    {
        return match($this) {
            self::PENDING => 'pending',
            self::OPEN => 'open',
            self::CLOSED => 'closed',
        };
    }
}
```

Class constants (rare) in UPPER_CASE:
```php
private const IMAGE_EXTENSIONS = ['jpg', 'jpeg', 'png', 'gif', 'svg', 'webp'];
```

## Type Hints
- **Return types** are used on Form Request methods (`authorize(): bool`, `rules(): array`) and some service methods
- **Parameter types** are used selectively (especially for model type hints and union types like `User|Admin`)
- **No strict typing culture** — type hints are optional in this codebase

## Error Handling
```php
// Database transactions with rollback
DB::beginTransaction();
try {
    // operations...
    DB::commit();
} catch (\Throwable $th) {
    DB::rollBack();
    return $this->errorResponse($th->getMessage());
}

// Service-level try/catch with logging
try {
    return Http::get($url)->throw()->json();
} catch (\Exception $exception) {
    \Log::error("Operation failed: {$exception->getMessage()}");
    return [];
}
```

## Documentation
- **Swagger annotations** (`@OA\`) on controller methods serve as API documentation
- **No PHPDoc blocks** on methods — code is self-documenting
- **No inline comments** except for complex logic sections

## Facades vs Injection
- **Facades used throughout:** `DB::`, `Cache::`, `Redis::`, `Http::`, `Storage::`, `\Log::`, `Route::`
- **No constructor injection** — services are called statically
- **`auth()` helper** for authentication: `auth()->user()`, `auth('admin')->user()`, `auth()->id()`

## Common Patterns

### Null Coalescing
```php
$data['message'] ?? ''
$user->email != '' ? $user->email : 'N/A'
```

### Array Manipulation
```php
$ids = array_values(array_unique($ids));
$res = array_map(fn($user) => $user['id'], $data);
```

### Collection Operations
```php
$categories->contains('id', $value);
$platforms->firstWhere('id', $platform_id);
$admins->pluck('id')->toArray();
```

### Conditional Query Building
```php
$query->when(isset($data['status']), function ($q) use ($data) {
    $q->where('status', $data['status']);
});
```

## Do's and Don'ts
### ✅ Do
- Use camelCase for methods, variables, and function names
- Use PHP 8.1+ backed enums for type-safe constants
- Use facades for framework services
- Use `DB::beginTransaction()` for multi-step writes
- Log errors with `\Log::error()` or `\Log::info()`

### ❌ Don't
- Don't use `declare(strict_types=1)`
- Don't use snake_case for methods or variables
- Don't create interfaces/contracts for services
- Don't use dependency injection — use static calls and facades
- Don't add PHPDoc blocks unless documenting complex parameters
