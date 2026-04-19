---
applyTo: "app/Observers/**"
---

# Observers



## Observer Structure

```php
<?php

namespace App\Observers;

use App\Models\Example;
use Illuminate\Support\Facades\Cache;

class ExampleObserver
{
    public function created(Example $example): void
    {
        Cache::forget("examples");
        Cache::forget("example_related_{$example->parent_id}");
    }

    public function updated(Example $example): void
    {
        Cache::forget("examples");
        Cache::forget("example_related_{$example->parent_id}");
    }

    public function deleted(Example $example): void
    {
        Cache::forget("examples");
        Cache::forget("example_related_{$example->parent_id}");
    }
}
```

---

## Registration

### Via `#[ObservedBy]` Attribute (recommended)
```php
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy(ExampleObserver::class)]
class Example extends BaseModel { ... }
```

---

## Do's and Don'ts

### ✅ Do
- Keep observers focused on a single concern (e.g., only cache invalidation)
- Invalidate all related cache keys on any lifecycle event
- Use `Cache::forget()` for targeted invalidation
- Handle `created`, `updated`, and `deleted` at minimum

### ❌ Don't
- Don't put business logic in observers
- Don't dispatch events from observers — do that from controllers
- Don't create audit logs in observers — do that from controllers
- Don't perform heavy operations in observers — they run synchronously
