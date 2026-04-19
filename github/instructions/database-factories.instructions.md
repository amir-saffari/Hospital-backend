---
applyTo: "database/factories/**"
---

# Factories



## Factory Structure

```php
<?php

namespace Database\Factories;

use App\Models\Example;
use Illuminate\Database\Eloquent\Factories\Factory;

class ExampleFactory extends Factory
{
    protected $model = Example::class;

    public function definition(): array
    {
        return [
            'name' => fake()->words(3, true),
            'email' => fake()->unique()->safeEmail(),
            'status' => 'active',
            'tenant_id' => 1,
            'content' => fake()->paragraph(),
        ];
    }

    // State methods for common variations
    public function inactive(): static
    {
        return $this->state(fn(array $attributes) => [
            'status' => 'inactive',
        ]);
    }
}
```

---

## Caching Expensive Values

```php
public function definition(): array
{
    return [
        'password' => static::$password ??= Hash::make('password'),
    ];
}
```

---

## Do's and Don'ts

### ✅ Do
- Match factory fields to the actual database schema
- Use `fake()` helper for generated data
- Cache expensive computations in static properties
- Create state methods for common variations

### ❌ Don't
- Don't include fields that don't exist in the migration
- Don't hardcode foreign key IDs without creating dependencies
- Don't use real email addresses or sensitive data
