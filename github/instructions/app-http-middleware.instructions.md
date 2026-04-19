---
applyTo: "app/Http/Middleware/**"
---

# Middleware



## Middleware Structure

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckCondition
{
    public function handle(Request $request, Closure $next)
    {
        if (!$this->is_valid($request)) {
            return response()->json(['message' => 'Condition not met'], 403);
        }

        return $next($request);
    }

    private function is_valid(Request $request): bool
    {
        // Validation logic
        return true;
    }
}
```

---

## Common Middleware Types

### Authentication Guard Check
```php
public function handle(Request $request, Closure $next)
{
    if (!auth('admin')->user()->is_active) {
        return response()->json(['message' => 'Account is not active'], 401);
    }
    return $next($request);
}
```

### Tenant Resolution
```php
public function handle(Request $request, Closure $next)
{
    $tenant_id = $this->resolve_tenant($request);
    $request->merge(['tenant_id' => $tenant_id]);
    return $next($request);
}
```

### Request Logging (post-response)
```php
public function handle(Request $request, Closure $next)
{
    $response = $next($request);

    // Defer logging to after response is sent
    app()->terminating(function () use ($request, $response) {
        LogRequestJob::dispatch($this->capture_data($request, $response));
    });

    return $response;
}
```

### Metrics Collection
```php
public function handle(Request $request, Closure $next)
{
    $start = microtime(true);
    $response = $next($request);
    $duration = microtime(true) - $start;

    // Record metric
    $this->record_metric($request, $response, $duration);

    return $response;
}
```

---

## Registration

```php
// In bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->appendToGroup('admin-api', [
        'auth:admin',
        CheckAdminIsActive::class,
        HttpLogMiddleware::class,
    ]);

    $middleware->appendToGroup('client-api', [
        'auth:api',
        ResolveTenant::class,
        HttpLogMiddleware::class,
    ]);
})
```

---

## Do's and Don'ts

### ✅ Do
- Register middleware on specific route groups, not globally
- Return JSON error responses — this is an API application
- Use jobs for heavy post-response work (logging, metrics)
- Use the correct auth guard (`auth('admin')`, `auth('api')`)

### ❌ Don't
- Don't register middleware globally unless absolutely necessary
- Don't perform heavy synchronous operations in middleware
- Don't use session-based middleware in API-only applications
- Don't use redirects — return JSON responses
