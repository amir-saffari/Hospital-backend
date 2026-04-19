---
applyTo: "app/Providers/**"
---

# Service Providers



## Provider Structure

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\URL;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Singleton bindings for shared stateful services
    }

    public function boot(): void
    {
        if (app()->environment() !== 'local') {
            URL::forceScheme('https');
        }
    }
}
```

---

## Do's and Don'ts

### ✅ Do
- Use `register()` for bindings and `boot()` for configuration
- Register singletons for stateful shared services
- Force HTTPS in production environments

### ❌ Don't
- Don't create providers for every small concern — keep it minimal
- Don't bind services that are only used statically
- Don't perform heavy initialization in `register()`
