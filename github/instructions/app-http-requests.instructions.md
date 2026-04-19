---
applyTo: "app/Http/Requests/**"
---

# Form Requests



## Class Structure

```php
<?php

namespace App\Http\Requests\V1\Admin\Example;

use Illuminate\Foundation\Http\FormRequest;

class CreateExampleRequest extends FormRequest
{
    public function authorize(): bool
    {
        // Authorization logic here
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'description' => ['nullable', 'string'],
            'status' => ['required', 'string', 'in:active,inactive'],
            'tenant_id' => ['required', 'integer', 'exists:tenants,id'],
            'config' => ['nullable', 'array'],
        ];
    }

    // Optional: after-validation hook for state checks
    public function after(): array
    {
        return [
            function ($validator) {
                // State-dependent validation that should only run
                // if basic validation passes
            }
        ];
    }
}
```

---

## Authorization Patterns

### Permission Check
```php
public function authorize(): bool
{
    $user = auth()->user();
    return $user->has_permission('create_example', $this->tenant_id);
}
```

### Ownership Check
```php
public function authorize(): bool
{
    return $this->route('example')->user_id === auth()->id();
}
```

### Super Admin Bypass
```php
public function authorize(): bool
{
    $user = auth()->user();
    if ($user->is_super_admin()) return true;
    return $user->has_permission('update_example', $this->route('example')->tenant_id);
}
```

### Tenant/Platform Access Check
```php
public function authorize(): bool
{
    return auth()->user()->has_access_to_tenant($this->tenant_id);
}
```

---

## Validation Rules

### Array Syntax (recommended)
```php
public function rules(): array
{
    return [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'email', 'unique:users,email'],
        'files' => ['nullable', 'array'],
        'files.*' => ['file', 'max:102400'],
        'parent_id' => ['nullable', 'integer', 'exists:parents,id'],
    ];
}
```

### Custom Rules
```php
use App\Rules\BelongsToTenant;

public function rules(): array
{
    return [
        'category_id' => ['required', 'integer', new BelongsToTenant],
    ];
}
```

---

## After-Validation Hooks

Used for checks that depend on validated data or application state:

```php
public function after(): array
{
    return [
        function ($validator) {
            $entity = $this->route('entity');

            if ($entity->status !== StatusEnum::ACTIVE) {
                $validator->errors()->add('entity', 'Entity is not in active state');
            }
        }
    ];
}
```

---

## Directory Structure Pattern

```
V1/Admin/
├── Auth/              (LoginRequest, etc.)
├── EntityA/           (Create, Update, Delete, GetAll)
├── EntityB/           (Create, Update, Delete)
V1/Client/
├── Auth/              (RegisterRequest, etc.)
├── EntityA/           (ClientCreateRequest)
```

---

## Do's and Don'ts

### ✅ Do
- Put all authorization logic in `authorize()`
- Use array syntax for validation rules
- Use `after()` for state-dependent validation
- Access route model bindings via `$this->route('model')`
- Use custom Rule classes for tenant/platform-scoped validations
- Group requests by entity in subdirectories

### ❌ Don't
- Don't validate inline in controllers
- Don't throw exceptions in Form Requests — use `$validator->errors()->add()`
- Don't put business logic in Form Requests — only validation and authorization
- Don't skip `authorize()` — always implement it (return `true` if no auth needed)
