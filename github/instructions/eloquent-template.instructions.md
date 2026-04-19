---
applyTo: "app/Models/**"
---

# Eloquent Model Conventions — Project-Wide

## Base Classes
All models **MUST** extend one of:
- `App\Models\Base\BaseModel` — standard models (SoftDeletes + `$guarded = []` + HasFactory)
- `App\Models\Base\BaseAuthModel` — authenticatable models (JWT + SoftDeletes + `$guarded = []`)

Exception: `ChatEvent` extends base `Model` directly (uses `metrics` TimescaleDB connection).

## Mass Assignment
**All models use `$guarded = []`** (inherited). Never define `$fillable`.
```php
// ✅ Correct — inherited from BaseModel
class Example extends BaseModel { }

// ❌ Wrong
class Example extends BaseModel {
    protected $fillable = ['name', 'status'];
}
```

## Casts
Use `$casts` for enum, array, and collection fields:
```php
protected $casts = [
    'status' => ChatStatusEnum::class,         // Enum cast
    'logs' => 'array',                          // JSON → array
    'payload' => 'array',                       // JSON → array
    'action' => AdminLogActionEnum::class,      // Enum cast
    'details' => AsCollection::class,           // JSON → Collection
];
```

Do NOT use accessor/mutator methods — use `$casts` instead.

## Relationships

### Naming
All relationship methods use **camelCase**:
```php
public function chatHistories() { return $this->hasMany(ChatHistory::class); }
public function lastMessage() { return $this->belongsTo(Message::class, 'last_message_id'); }
public function phoneCountry() { return $this->belongsTo(Country::class, 'phone_country_id'); }
```

### SoftDeletes in Relationships
Use `withTrashed()` when soft-deleted records should be accessible:
```php
public function notes()
{
    return $this->hasMany(Note::class)->withTrashed();
}
```

### Polymorphic
Messages use `morphTo` for sender/receiver; Notifications use `morphMany`:
```php
public function sender() { return $this->morphTo(); }
public function notifications() { return $this->morphMany(Notification::class, 'model'); }
```

## Query Scopes

### Platform Access Scope
Standard pattern for multi-tenant filtering:
```php
public function scopeAdminAllowedPlatforms($query)
{
    $admin = auth('admin')->user();
    if ($admin->is_super_admin()) return $query;
    $platform_ids = $admin->get_platform_ids_with_certain_permission('view_example');
    return $query->whereIn('platform_id', $platform_ids);
}
```

### Search Scopes
Complex search with conditional filters:
```php
public function scopeSearchForAdmin($query, $data)
{
    if (isset($data['text']) && $data['text'] !== '') {
        $query->where(function ($q) use ($data) {
            $q->where('name', 'ilike', "%{$data['text']}%");
        });
    }
    if (isset($data['platform_id'])) {
        $query->where('platform_id', $data['platform_id']);
    }
}
```

### Cursor Pagination Scopes
```php
public function scopeBeforeCursor($query, $cursor)
{
    return $query->where('id', '<', $cursor);
}
```

## Caching Pattern
Static cache methods with `Cache::remember()`:
```php
public static function get_from_cache($id)
{
    return Cache::remember("examples", 100000, function () {
        return self::all();
    })->firstWhere('id', $id);
}
```

TTL: ~100000 seconds (~27.7 hours). Cache busted by observers.

## Observers
Register via `#[ObservedBy]` attribute. Observers handle cache invalidation only:
```php
#[ObservedBy(ExampleObserver::class)]
class Example extends BaseModel { ... }
```

## Custom Route Binding
For models where soft-deleted records should be accessible via route binding:
```php
public function resolveRouteBinding($value, $field = null)
{
    return $this->where($field ?? 'id', $value)->withTrashed()->firstOrFail();
}
```

## Enums
Located in `app/Models/Enums/`. PHP 8.1+ backed enums:
```php
enum ExampleTypeEnum: string
{
    case TYPE_A = 'type_a';
    case TYPE_B = 'type_b';

    public function to_string(): string
    {
        return match ($this) {
            self::TYPE_A => 'Type A',
            self::TYPE_B => 'Type B',
        };
    }
}
```

## Do's and Don'ts
### ✅ Do
- Extend `BaseModel` or `BaseAuthModel`
- Use `$casts` for typed columns
- Use camelCase for relationships and scopes
- Add `scopeAdminAllowedPlatforms()` for platform-scoped entities
- Use `Cache::remember()` for hot reference data
- Register observers for cache invalidation via `#[ObservedBy]`

### ❌ Don't
- Don't use `$fillable` — all models use `$guarded = []`
- Don't use accessor/mutator methods — use `$casts`
- Don't put business logic in models
- Don't skip SoftDeletes (unless metrics model)
- Don't create models without base class extension
