---
applyTo: "app/Scopes/**"
---

# Query Scopes



## Global Scope Structure

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class DefaultSortScope implements Scope
{
    public function apply(Builder $builder, Model $model): void
    {
        $query = $builder->getQuery();

        // Don't override existing order
        if (!empty($query->orders) || !empty($query->unionOrders)) {
            return;
        }

        $builder->orderBy($model->getTable() . '.created_at', 'desc');
    }
}
```

---

## Do's and Don'ts

### ✅ Do
- Check for existing query modifications before adding defaults
- Use table-qualified column names to avoid ambiguity in joins
- Document which models have global scopes applied

### ❌ Don't
- Don't enable global scopes without verifying all queries still work
- Don't apply scopes that could break pagination or aggregation
