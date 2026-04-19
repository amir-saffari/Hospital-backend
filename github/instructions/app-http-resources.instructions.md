---
applyTo: "app/Http/Resources/**"
---

# API Resources



## Resource Structure

```php
<?php

namespace App\Http\Resources\V1\Admin\Example;

use Illuminate\Http\Resources\Json\JsonResource;

class ExampleResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'status' => $this->status,
            'parent' => new ParentResource($this->whenLoaded('parent')),
            'children' => ChildResource::collection($this->whenLoaded('children')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

---

## Actor Separation (if chosen)

### Admin Resource (full data)
```php
class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'phone' => $this->phone,
            'role' => $this->role,
            'is_active' => $this->is_active,
            'last_login_at' => $this->last_login_at,
            'created_at' => $this->created_at,
        ];
    }
}
```

### Client Resource (limited data)
```php
class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'avatar' => $this->avatar_url,
        ];
    }
}
```

---

## Collection Resources

Custom collections when you need a wrapped response:
```php
class ExampleCollection extends ResourceCollection
{
    public $collects = ExampleResource::class;
}
```

---

## Conditional Relationships

Always use `whenLoaded()` to prevent N+1 queries:
```php
'parent' => new ParentResource($this->whenLoaded('parent')),
'items' => ItemResource::collection($this->whenLoaded('items')),
```

---

## Directory Structure Pattern

```
V1/
├── Base/
│   ├── BaseCollection.php
│   └── BaseResource.php
├── Admin/
│   ├── EntityA/          (EntityAResource.php)
│   ├── EntityB/          (EntityBResource.php)
├── Client/
│   ├── EntityA/          (EntityAResource.php — limited fields)
└── Shared/
    └── CommonEntity/     (CommonResource.php — used by both)
```

---

## Do's and Don'ts

### ✅ Do
- Create separate resources for different actors when they expose different fields
- Use `$this->whenLoaded()` for all optional relationships
- Place shared resources in `Shared/` directory
- Return explicit field arrays — never use `parent::toArray()`
- Keep resources thin — only transformation logic

### ❌ Don't
- Don't expose sensitive fields (passwords, tokens, secrets)
- Don't share admin resources with client endpoints
- Don't use `$this->resource->` — use `$this->` directly
- Don't include relationships without `whenLoaded()` — it causes N+1 queries
- Don't put business logic in resources
