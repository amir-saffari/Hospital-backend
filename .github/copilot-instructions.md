# Copilot Instructions — hospital-backend

## Project Overview
API-only hospital backend built with Laravel 13 and PHP 8.3.
No Blade views. Every response is JSON. All routes belong in `routes/api.php`.

## Tech Stack
| Layer        | Technology                        |
|--------------|-----------------------------------|
| Framework    | Laravel 13                        |
| Language     | PHP 8.3                           |
| Auth         | JWT (stateless, Bearer token)     |
| Database     | PostgreSQL (prod/dev), SQLite :memory: (tests) |
| Testing      | PHPUnit 12                        |
| Frontend build | Vite (asset pipeline only)     |
| Code style   | PSR-12 via Laravel Pint           |

## Build & Test Commands
```bash
# Install dependencies
composer install

# Run all tests
php artisan test
# or
./vendor/bin/phpunit

# Code style fix
./vendor/bin/pint

# Run development server
php artisan serve

# Run Vite (if needed for asset compilation)
npm run dev
```

## Project Structure
```
app/
  Http/
    Controllers/   # Thin controllers — delegate to services
    Middleware/
  Models/          # Eloquent models
  Services/        # Business logic lives here
routes/
  api.php          # All API routes (create if absent)
  web.php          # Unused — do not add API routes here
tests/
  Feature/         # HTTP-level feature tests (required for every public endpoint)
  Unit/            # Unit tests for services and helpers
database/
  migrations/      # One migration per logical change; never edit shipped migrations
  factories/
  seeders/
memory/
  constitution.md  # Project domain rules and team practices (source of truth)
```

## Code Conventions
- Controllers must be thin. Move validation to FormRequest classes and logic to Service/Action classes.
- Never return sensitive fields (`password`, `national_id`) in API responses.
- Use `422 Unprocessable Entity` for validation failures (Laravel default).
- Use `201 Created` for successful resource creation.
- All new endpoints require a Feature test in `tests/Feature/`.
- PSR-12 enforced — run Pint before committing.
- Migrations are append-only after the first commit that ships them.

## Domain Summary (Patient Registration Feature)
- Roles: admin, doctor, patient
- Required patient fields: full_name, national_id, phone, date_of_birth, gender, address, password
- Optional field: email
- national_id is the uniqueness key (duplicate → 422)
- New accounts are active immediately (no approval gate)
- Patients self-register with their own password
- Admins create patients and set the initial password
- See `memory/constitution.md` for full domain rules
