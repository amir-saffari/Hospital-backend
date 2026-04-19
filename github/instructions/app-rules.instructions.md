---
applyTo: "app/Rules/**"
---

# Custom Validation Rules



## Rule Structure

```php
<?php

namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class BelongsToTenant implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        $items = Entity::get_from_cache(get_current_tenant_id());

        if (!$items->contains('id', $value)) {
            $fail("The selected {$attribute} does not belong to this tenant.");
        }
    }
}
```

---

## Usage in Form Requests

```php
public function rules(): array
{
    return [
        'entity_id' => ['required', 'integer', new BelongsToTenant],
    ];
}
```

---

## Do's and Don'ts

### ✅ Do
- Use cached data for lookups — avoid raw DB queries
- Return clear, user-friendly error messages
- Scope rules to tenant/platform when applicable

### ❌ Don't
- Don't perform expensive DB queries in rules
- Don't duplicate logic that `exists` rule can handle
