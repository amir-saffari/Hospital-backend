---
applyTo: "database/migrations/**"
---

# Migrations



## Table Creation Pattern

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('examples', function (Blueprint $table) {
            $table->id();
            $table->foreignId('tenant_id')->constrained('tenants');
            $table->string('name');
            $table->integer('status');                    // PHP enum in model
            $table->longText('content')->nullable();
            $table->longText('config')->nullable();      // JSON cast in model
            $table->boolean('is_active')->default(true);
            $table->softDeletes();
            $table->timestamps();
        });

        Schema::table('examples', function (Blueprint $table) {
            $table->index('tenant_id', 'idx_examples_tenant_id');
            $table->index('status', 'idx_examples_status');
            $table->index('deleted_at', 'idx_examples_deleted_at');
        });
    }

    public function down(): void
    {
        Schema::table('examples', function (Blueprint $table) {
            $table->dropIndex('idx_examples_tenant_id');
            $table->dropIndex('idx_examples_status');
            $table->dropIndex('idx_examples_deleted_at');
        });
        Schema::dropIfExists('examples');
    }
};
```

---

## Alter Table Pattern

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::table('examples', function (Blueprint $table) {
            $table->string('new_column')->nullable()->after('existing_column');
        });
    }

    public function down(): void
    {
        Schema::table('examples', function (Blueprint $table) {
            $table->dropColumn('new_column');
        });
    }
};
```

---

## Column Type Guide

| Purpose | Column Type |
|---------|------------|
| Primary key | `$table->id()` |
| Foreign keys | `$table->foreignId('parent_id')->constrained('parents')` |
| Short text | `$table->string('name')` |
| Long text | `$table->longText('content')` |
| JSON data | `$table->longText('data')->nullable()` (or `$table->json('data')`) |
| Flags | `$table->boolean('is_active')->default(true)` |
| Enum-like | `$table->integer('status')` (cast to PHP enum in model) |
| Polymorphic | `$table->morphs('morphable')` |
| Timestamps | `$table->timestamps()` + `$table->softDeletes()` |

---

## Index Naming Convention

```php
// Pattern: idx_tablename_columnnames
$table->index(['status', 'tenant_id'], 'idx_examples_status_tenant');
$table->index('deleted_at', 'idx_examples_deleted_at');
$table->unique('email', 'idx_users_email_unique');
```

---

## Do's and Don'ts

### ✅ Do
- Always add named indexes on frequently queried columns
- Always include `softDeletes()` on entity tables (if spec requires)
- Use `foreignId()->constrained()` for all foreign keys
- Implement `down()` migrations that properly drop indexes before tables
- Use `longText` or `json` for payload/config columns
- Add `nullable()` to optional columns

### ❌ Don't
- Don't use native `enum` column type — use `string` with PHP enum cast
- Don't skip indexes on frequently queried columns
- Don't modify existing migrations — create new ones for alterations
- Don't use `dropIfExists()` without first dropping foreign keys/indexes
