---
applyTo: "database/seeders/**"
---

# Seeders



## Seeder Structure

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        // Order matters — seed dependencies first
        $this->call([
            TenantSeeder::class,
            RoleSeeder::class,
            PermissionSeeder::class,
            UserSeeder::class,
            // ...
        ]);
    }
}
```

### Entity Seeder
```php
class TenantSeeder extends Seeder
{
    public function run(): void
    {
        Tenant::create([
            'name' => 'Default Tenant',
            'config' => [
                'feature_a' => true,
                'feature_b' => false,
            ],
        ]);
    }
}
```

---

## Do's and Don'ts

### ✅ Do
- Keep seeders idempotent where possible (`firstOrCreate`)
- Use CSV/JSON files for large reference datasets
- Seed in dependency order (tenants before users)
- Set up complete configuration for development

### ❌ Don't
- Don't hardcode production credentials
- Don't seed production-specific data in development seeders
- Don't create seeders with no `down()` strategy
