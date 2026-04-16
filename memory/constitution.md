# Project Constitution — hospital-backend

## Identity
API-only hospital backend. No server-rendered views. All responses are JSON.

## Roles
| Role    | Notes                                      |
|---------|--------------------------------------------|
| admin   | Full management access; may create patients |
| doctor  | Clinical access; no patient-creation rights in current scope |
| patient | Self-service access to own records         |

## Authentication
- JWT-based (stateless, no session cookies)
- Token issued on login; passed as `Authorization: Bearer <token>`

## Database
- PostgreSQL (production and development)
- SQLite in-memory for PHPUnit test runs (see `phpunit.xml`)

## Domain Rules (patient registration)
1. Required registration fields: `full_name`, `national_id`, `phone`, `date_of_birth`, `gender`, `address`, `password`
2. `email` is optional and stored when provided
3. `national_id` is the uniqueness key — duplicate registrations must be rejected with a 422
4. A newly registered patient account is **active immediately** — no approval gate
5. Self-registration: patient submits own password
6. Admin-created patient: admin supplies the initial password on behalf of the patient
7. Admins may create patient accounts via a separate privileged endpoint

## Security Baseline
- Passwords hashed with bcrypt (BCRYPT_ROUNDS=4 in test env, default in production)
- No sensitive fields (password, national_id) in API responses
- Input validated at the HTTP layer before reaching the service/model layer

## Team Practices
- PSR-12 code style enforced by Laravel Pint (`./vendor/bin/pint`)
- Feature tests live in `tests/Feature/`, unit tests in `tests/Unit/`
- One migration per logical change; never edit shipped migrations
- Route files: `routes/api.php` for all API routes (does not exist yet — must be created)
- Controllers are thin; business logic lives in service classes or action classes
- All public API endpoints must have a corresponding Feature test
