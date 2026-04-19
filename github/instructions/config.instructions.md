---
applyTo: "config/**"
---

# Configuration



## Configuration Conventions

### Environment Variables
```php
// Always provide defaults
'api_key' => env('SERVICE_API_KEY', ''),
'timeout' => env('SERVICE_TIMEOUT', 30),
```

### Auth Guards
```php
// config/auth.php
'defaults' => [
    'guard' => '[CHOSEN_DEFAULT]',
],
'guards' => [
    'admin' => [
        'driver' => '[jwt/sanctum/session]',
        'provider' => 'admins',
    ],
    'api' => [
        'driver' => '[jwt/sanctum/session]',
        'provider' => 'users',
    ],
],
```

### Multiple Database Connections (if applicable)
```php
// config/database.php
'connections' => [
    'pgsql' => [ /* Primary */ ],
    'metrics' => [ /* Metrics/Analytics DB */ ],
],
```

---

## Do's and Don'ts

### ✅ Do
- Provide sensible defaults for all `env()` calls
- Keep sensitive values in `.env` — never hardcode credentials
- Document new env vars in `.env.example`
- Use `config()` helper in application code when possible

### ❌ Don't
- Don't change the default auth guard without updating all `auth()` calls
- Don't add database connections without corresponding infrastructure
- Don't hardcode environment-specific values in config files
