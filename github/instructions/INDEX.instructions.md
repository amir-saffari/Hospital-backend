---
applyTo: "**"
---

# Laravel Project — Project Instructions Index

## Project Overview
A Laravel application using Spec-Driven Development

## Instruction Files

| File | Applies To | Description |
|------|-----------|-------------|
| [app-http-resources.instructions.md](app-http-resources.instructions.md) | `app/Http/Resources/**` | API Resource transformation — actor separation, JSON structure |
| [database-factories.instructions.md](database-factories.instructions.md) | `database/factories/**` | Model factories — test data generation, states |
| [app-providers.instructions.md](app-providers.instructions.md) | `app/Providers/**` | Service providers — binding, booting, registration |
| [app-console.instructions.md](app-console.instructions.md) | `app/Console/**` | Artisan commands — signatures, I/O, scheduling |
| [app-broadcasting.instructions.md](app-broadcasting.instructions.md) | `app/Broadcasting/**` | Broadcasting driver — channel auth, presence |
| [app-http-controllers.instructions.md](app-http-controllers.instructions.md) | `app/Http/Controllers/**` | Controller architecture — response helpers, transactions, swagger |
| [app-http-requests.instructions.md](app-http-requests.instructions.md) | `app/Http/Requests/**` | Form Request validation — authorization, rules, after-hooks |
| [routes.instructions.md](routes.instructions.md) | `routes/**` | Routing patterns — versioning, naming, middleware groups |
| [database-seeders.instructions.md](database-seeders.instructions.md) | `database/seeders/**` | Database seeders — reference data, idempotency |
| [app-http-middleware.instructions.md](app-http-middleware.instructions.md) | `app/Http/Middleware/**` | HTTP middleware — platform resolution, auth, metrics |
| [app-rules.instructions.md](app-rules.instructions.md) | `app/Rules/**` | Custom validation rules — platform-scoped, reusable |
| [app-observers.instructions.md](app-observers.instructions.md) | `app/Observers/**` | Model observers — cache invalidation patterns |
| [api-conventions.instructions.md](api-conventions.instructions.md) | `routes/**, app/Http/Controllers/**, app/Http/Resources/**, app/Http/Requests/**` | REST API conventions — versioning, response format, pagination |
| [php-conventions.instructions.md](php-conventions.instructions.md) | `**/*.php` | PHP language conventions — naming, types, error handling |
| [app-helpers.instructions.md](app-helpers.instructions.md) | `app/Helpers/**` | Global helper functions — utility, formatting |
| [app-jobs.instructions.md](app-jobs.instructions.md) | `app/Jobs/**` | Queued jobs — retry, backoff, rate limiting |
| [app-models.instructions.md](app-models.instructions.md) | `app/Models/**` | Eloquent model patterns — base classes, relationships, caching, enums |
| [app-listeners.instructions.md](app-listeners.instructions.md) | `app/Listeners/**` | Event listeners — metrics, async processing |
| [app-events.instructions.md](app-events.instructions.md) | `app/Events/**` | Event architecture — broadcast, queued, group events |
| [database-migrations.instructions.md](database-migrations.instructions.md) | `database/migrations/**` | Database migrations — schema patterns, indexing, constraints |
| [app-notifications.instructions.md](app-notifications.instructions.md) | `app/Notifications/**` | Notification system — channels, database, telegram |
| [app-services.instructions.md](app-services.instructions.md) | `app/Services/**` | Service layer — static vs instance, error handling, integration |
| [app-scopes.instructions.md](app-scopes.instructions.md) | `app/Scopes/**` | Query scopes — global scopes, local scopes |
| [config.instructions.md](config.instructions.md) | `config/**` | Configuration — env vars, service config, caching |
| [tests.instructions.md](tests.instructions.md) | `tests/**` | Testing strategy — structure, assertions, authentication |

## How These Instructions Were Generated
These instruction files are generated from the SDD (Spec-Driven Development) framework.

- Source specs: `laravel-sdd/specs/`
- Master config: `laravel-sdd/PROJECT_SPEC.md`
- Generator: `laravel-sdd/scripts/generate-instructions.sh`

To regenerate after spec changes:
```bash
./laravel-sdd/scripts/generate-instructions.sh
```

