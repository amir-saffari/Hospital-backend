---
applyTo: "routes/**"
---

# Routes



## Route File Organization

### If Separate Files per Actor
```
routes/
├── V1/
│   ├── Admin/
│   │   ├── auth.php          # Admin authentication (minimal middleware)
│   │   └── api.php           # Authenticated admin endpoints
│   ├── Client/
│   │   ├── auth.php          # Client authentication
│   │   └── api.php           # Authenticated client endpoints
│   └── Public/
│       └── api.php           # Public/unauthenticated endpoints
├── channels.php
└── web.php
```

### Registration in `bootstrap/app.php`
```php
->withRouting(
    web: __DIR__ . '/../routes/web.php',
    then: function () {
        Route::prefix('api/v1/admin/auth')
            ->middleware(['api'])
            ->group(base_path('routes/V1/Admin/auth.php'));

        Route::prefix('api/v1/admin')
            ->middleware(['api', 'auth:admin'])
            ->group(base_path('routes/V1/Admin/api.php'));

        Route::prefix('api/v1/client')
            ->middleware(['api', 'auth:api'])
            ->group(base_path('routes/V1/Client/api.php'));
    },
)
```

---

## Route Definition Pattern

### Controller Groups (recommended)
```php
Route::controller(ExampleController::class)->prefix('examples')->group(function () {
    Route::get('/', 'index');
    Route::get('/table', 'table');
    Route::post('', 'create');
    Route::get('/{example}', 'get');
    Route::put('/{example}', 'update');
    Route::delete('/{example}', 'delete');
});
```

---

## URI Conventions

### HTTP Verb Mapping

| Action | Verb | URI Pattern | Method |
|--------|------|-------------|--------|
| List all | GET | `/examples` | `index` |
| Paginated table | GET/POST | `/examples/table` | `table` |
| Create | POST | `/examples` | `create` |
| Get single | GET | `/examples/{example}` | `get` |
| Update | PUT | `/examples/{example}` | `update` |
| Delete | DELETE | `/examples/{example}` | `delete` |
| Custom action | POST | `/examples/{example}/action-name` | `action_name` |

### URI Naming
```
# If kebab-case:
/api/v1/admin/chat-categories
/api/v1/admin/chats/{chat}/send-message

```

---

## Middleware Stacks

### Example: RBAC + Logging + Metrics
```
Auth routes:
  api, LogMiddleware

Authenticated routes:
  api, auth:{guard}, ActiveCheck, LogMiddleware, MetricsMiddleware

Public routes:
  api, RateLimiting
```

### Selective Middleware Removal
```php
Route::post('/public-endpoint', 'public_action')
    ->withoutMiddleware(['auth:api']);
```

---

## Route Model Binding
```php
// Singular parameter names match model class
Route::get('/{example}', 'get');        // Resolves Example model
Route::put('/{example}/update', 'update');
```

---

## Do's and Don'ts

### ✅ Do
- Group routes by controller using `Route::controller()`
- Use consistent URI naming convention throughout
- Apply middleware via route groups in `bootstrap/app.php`
- Use route model binding for entity access
- Separate route files by actor/auth level

### ❌ Don't
- Don't mix URI naming conventions (e.g., kebab-case and snake_case)
- Don't use `Route::resource()` if you need explicit control — define each route
- Don't add routes without corresponding controller methods
- Don't apply heavy middleware globally — scope to route groups
