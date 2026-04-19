# Copilot Instructions

## Project

A hospital management API platform for patient records, appointments, billing, and clinical workflows.
API-only Laravel backend for secure healthcare operations, role-based access, reporting, and third-party medical system integrations.

## Role
You are a senior laravel developer with 15 years of experience.
you prioritize readability, maintainability, and consistency in your code. You follow best practices and conventions for Laravel development, and you write clean, efficient, and well-documented code.

## Stack

- Framework: Laravel **13**
- Language: PHP **8.3**
- Backend: Laravel application with Eloquent ORM, queues, sessions, cache, and Artisan tooling
- Database: PostgreSQL (configured in `.env`)
- Authentication: JWT Bearer token via `tymon/jwt-auth`
- Cache / Session / Queue: database drivers by default, with Redis support configured via `phpredis`
- Developer Tooling: Laravel Tinker, Laravel Pail, and `concurrently` for local development workflow
- Testing: PHPUnit **12**
- Code Style: Laravel Pint

## Commands

- `composer install` — Install dependencies
- `php artisan serve` — Start dev server (or use Octane in Docker)
- `php artisan test` — Run PHPUnit test suite
- `php artisan l5-swagger:generate` — Regenerate Swagger docs
- `./vendor/bin/pint` — Fix code style
- `php artisan migrate` — Run migrations
- `php artisan db:seed` — Seed reference data

## Project Structure

```
app/
  Http/
    Controllers/       # V1 controllers (Admin/, Client/, Metrics/)
    Middleware/         # Platform resolution, auth, metrics
    Requests/          # Form Request validation
    Resources/         # API Resource transformations
  Models/              # Eloquent models ($guarded = [])
  Services/            # Business logic layer
  Events/              # Broadcast events (Centrifugo)
  Jobs/                # Queued jobs
  Observers/           # Model observers (cache invalidation)
routes/
  V1/                  # Versioned API routes (admin.php, client.php, metrics.php)
  channels.php         # Centrifugo channel auth
database/
  migrations/          # PostgreSQL + TimescaleDB migrations
  seeders/             # Reference data seeders
  factories/           # Model factories for tests
config/                # App configuration
tests/
  Feature/             # Feature tests
  Unit/                # Unit tests
```

Versioning: API versioning via route groups (`/api/v1/...`), with separate controllers, resources, requests, and routes for each version 

## Conventions

- **English only** — all classes, tables, routes, variables
- **StudlyCase** — classes, models, controllers, namespaces
- **camelCase** — variables, functions, Eloquent relationship methods
- **snake_case** — files, columns, config files, blade files, migration files, env keys (UPPER_SNAKE_CASE)
- **kebab-case** — route URIs (lower kebab-case)
- **No `declare(strict_types=1)`** — not used in this codebase
- **No docblocks** — Swagger `@OA\` annotations serve as docs
- **`$guarded = []`** on all models — never use `$fillable`
- All responses via base controller helpers: `$this->successResponse($data)`, `$this->errorResponse($msg, $code)`
- `DB::beginTransaction()` / `commit()` / `rollBack()` for multi-step writes
- Swagger `@OA\` annotations on every API endpoint

## Boundaries

- NEVER use `declare(strict_types=1)`
- NEVER use `$fillable` — models use `$guarded = []`
- NEVER use Pest — this project uses PHPUnit
- NEVER bypass Platform middleware for authenticated client routes
- NEVER use Laravel's built-in Notification table — use the custom `notifications` table
- NEVER commit `.env` files or secrets
- NEVER use `dd()` or `dump()` in committed code
- NEVER add dependencies without considering the impact

## Validation

After any code change:
1. `./vendor/bin/pint` — code style must pass
2. `php artisan test` — tests must pass
3. `php artisan l5-swagger:generate` — Swagger must regenerate without errors (if API endpoints changed)
