---
applyTo: "app/Models/**"
---

# Eloquent Models



## Base Classes

### If Custom Base Model
```php
<?php

namespace App\Models\Base;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class BaseModel extends Model
{
    use HasFactory, SoftDeletes;

    protected $guarded = []; // If chosen
}
```

### If Custom Base Auth Model
```php
<?php

namespace App\Models\Base;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Foundation\Auth\User as Authenticatable;
// Add JWT trait if using JWT:
// use Tymon\JWTAuth\Contracts\JWTSubject;

class BaseAuthModel extends Authenticatable // implements JWTSubject
{
    use HasFactory, SoftDeletes;

    protected $guarded = [];
}
```

---

## Model Structure Template

```php
<?php

namespace App\Models;

use App\Models\Base\BaseModel;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy(ExampleObserver::class)] // Only if observer needed
class Example extends BaseModel
{
    // 1. Properties
    protected $casts = [
        'status' => ExampleStatusEnum::class,
        'config' => 'array',
    ];

    // 2. Relationships
    public function parent()
    {
        return $this->belongsTo(Parent::class);
    }

    public function children()
    {
        return $this->hasMany(Child::class);
    }

    // 3. Scopes
    public function scopeActive($query)
    {
        return $query->where('status', StatusEnum::ACTIVE);
    }

    // 4. Cache Methods (if static cache pattern chosen)
    public static function get_from_cache($id)
    {
        return Cache::remember("examples", 100000, function () {
            return self::all();
        })->firstWhere('id', $id);
    }

    // 5. Business Logic Helpers
    public function is_active(): bool
    {
        return $this->status === StatusEnum::ACTIVE;
    }
}
```

---

## Relationships

### Naming Convention
```php

// If camelCase:
public function chatHistories() { return $this->hasMany(ChatHistory::class); }
public function lastMessage() { return $this->belongsTo(Message::class, 'last_message_id'); }
```

### Soft Deleted Records in Relationships
```php
// When soft-deleted related records should be accessible:
public function notes()
{
    return $this->hasMany(Note::class)->withTrashed();
}
```

### Polymorphic Relationships
```php
public function sender()
{
    return $this->morphTo();
}

public function notifications()
{
    return $this->morphMany(Notification::class, 'notifiable');
}
```

---

## Query Scopes

### Multi-Tenant / Platform Access Scope
```php
public function scopeAllowedForUser($query)
{
    $user = auth()->user();
    if ($user->is_super_admin()) return $query;
    $allowed_ids = $user->get_allowed_entity_ids('view_example');
    return $query->whereIn('tenant_id', $allowed_ids);
}
```

### Search Scope with Conditional Filters
```php
public function scopeSearch($query, array $filters)
{
    if (isset($filters['text']) && $filters['text'] !== '') {
        $query->where(function ($q) use ($filters) {
            $q->where('name', 'ilike', "%{$filters['text']}%");
        });
    }

    if (isset($filters['status'])) {
        $query->where('status', $filters['status']);
    }

    return $query;
}
```

### Cursor Pagination Scope
```php
public function scopeBeforeCursor($query, $cursor)
{
    return $query->where('id', '<', $cursor);
}
```

---

## Caching Pattern

### If Static Cache Methods
```php
public static function get_from_cache($id)
{
    return Cache::remember("examples", 100000, function () {
        return self::all();
    })->firstWhere('id', $id);
}

public static function get_all_from_cache()
{
    return Cache::remember("examples", 100000, function () {
        return self::all();
    });
}
```

Cache invalidation is handled by observers (see `observers.spec.md`).

---

## Route Model Binding

### Custom Binding (include soft-deleted)
```php
public function resolveRouteBinding($value, $field = null)
{
    return $this->where($field ?? 'id', $value)->withTrashed()->firstOrFail();
}
```

---

## Enums

```php
<?php

namespace App\Models\Enums; // or App\Enums based on choice

enum StatusEnum: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
    case ARCHIVED = 'archived';

    public function label(): string
    {
        return match ($this) {
            self::ACTIVE => 'Active',
            self::INACTIVE => 'Inactive',
            self::ARCHIVED => 'Archived',
        };
    }

    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }
}
```

---

## Do's and Don'ts

### ✅ Do
- Extend the chosen base class for all models
- Use `$casts` for typed columns (enums, JSON, dates)
- Add multi-tenant scopes for entities scoped to a tenant
- Use `Cache::remember()` for frequently accessed reference data
- Register observers via `#[ObservedBy]` for cache invalidation
- Use `withTrashed()` on relationships where soft-deleted records should be accessible

### ❌ Don't
- Don't put heavy business logic in models — use services or actions
- Don't skip the base class unless there's a specific reason (e.g., metrics model with different connection)
- Don't use raw queries in models — use scopes and Eloquent methods
- Don't create models without corresponding migration and factory
