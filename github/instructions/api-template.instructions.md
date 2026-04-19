---
applyTo: "routes/**, app/Http/Controllers/**, app/Http/Resources/**, app/Http/Requests/**"
---

# API Design Conventions — Project-Wide

## API Structure
- **Base URL:** `/api/v1/`
- **Versioning:** URL path versioning (`v1`)
- **Two actor namespaces:** `/admin/` and `/client/`
- **Authentication:** JWT Bearer token via `tymon/jwt-auth`

## Authentication

### Admin Auth
- Login: `POST /api/v1/admin/auth/login` → returns JWT with `meta.type = "admin"` claim
- Guard: `admin`
- TTL: 30 days (43200 minutes)
- Header: `Authorization: Bearer {token}`

## Response Format

### Success
```json
{
    "status": 200,
    "data": { ... }
}
```

### Error
```json
{
    "status": 400,
    "message": "Error description"
}
```

### Not Found
```json
{
    "status": 404,
    "message": "Resource not found"
}
```

### Validation Error (422)
```json
{
    "message": "The given data was invalid.",
    "errors": {
        "field": ["Error message"]
    }
}
```

### Response Helpers (from base Controller)
```php
$this->successResponse($data);                    // 200
$this->errorResponse('Message', 400);         // Custom error code
$this->notFoundResponse('Not found');        // 404
$this->forbiddenResponse('Access denied');    // 403
```

## Pagination

### Standard Pagination
Uses Laravel's `->paginate(15)`:
```php
$items = Model::query()->paginate(15);
return $this->successResponse($items);
```

### Cursor-based Pagination
Custom implementation for messages (infinite scroll):
```php
$messages = Message::where('chat_id', $chat->id)
    ->when($cursor, fn($q) => $q->where('id', '<', $cursor))
    ->orderBy('id', 'desc')
    ->limit($limit)
    ->get();
```

## Swagger/OpenAPI Documentation

Every endpoint MUST have complete `@OA\` annotations:
```php
/**
 * @OA\Post(
 *     path="/api/v1/admin/examples",
 *     summary="Create a new example",
 *     description="Creates a new example record",
 *     operationId="createExample",
 *     tags={"Admin Example"},
 *     security={{"bearerAuth":{}}},
 *     @OA\RequestBody(
 *         required=true,
 *         @OA\JsonContent(
 *             required={"name"},
 *             @OA\Property(property="name", type="string", example="Example Name"),
 *         )
 *     ),
 *     @OA\Response(response=200, description="Success", @OA\JsonContent(ref="#/components/schemas/SuccessResponse")),
 *     @OA\Response(response=401, description="Unauthorized", @OA\JsonContent(ref="#/components/schemas/UnauthorizedResponse")),
 *     @OA\Response(response=422, description="Validation Error", @OA\JsonContent(ref="#/components/schemas/ValidationErrorResponse")),
 * )
 */
```

Reusable schema references:
- `#/components/schemas/SuccessResponse`
- `#/components/schemas/ErrorResponse`
- `#/components/schemas/UnauthorizedResponse`
- `#/components/schemas/ForbiddenResponse`
- `#/components/schemas/NotFoundResponse`
- `#/components/schemas/ValidationErrorResponse`

## Route Conventions

### URI Patterns
```
GET    /api/v1/admin/{entity}                 → list/paginate
GET    /api/v1/admin/{entity}/table            → paginated table
POST   /api/v1/admin/{entity}/create           → create
GET    /api/v1/admin/{entity}/{id}             → get single
PUT    /api/v1/admin/{entity}/{id}/update      → update
DELETE /api/v1/admin/{entity}/{id}/delete      → delete
POST   /api/v1/admin/{entity}/{id}/action      → custom action
```

### Route Registration
```php
Route::controller(ExampleController::class)->prefix('examples')->group(function () {
    Route::get('/table', 'table');
    Route::post('/create', 'create');
    Route::get('/{example}', 'get');
    Route::put('/{example}/update', 'update');
    Route::delete('/{example}/delete', 'delete');
});
```

## File Uploads
- Sent as `multipart/form-data`
- Processed by `FileService::store()` with image optimization
- Stored on `private` disk, served via authenticated download endpoint

## Error Handling
- Validation errors: automatic 422 from Form Requests
- Auth errors: automatic 401 from JWT middleware
- Business logic errors: `errorResponse()` with descriptive message
- Not found: `notFoundResponse()` or automatic 404 from route model binding
- Forbidden: `forbiddenResponse()` or automatic 403 from Form Request `authorize()`

## Do's and Don'ts
### ✅ Do
- Return data through API Resource classes
- Use Form Requests for validation + authorization
- Add Swagger annotations to every endpoint
- Use route model binding for entity access
- Include `security={{"bearerAuth":{}}}` in Swagger for auth endpoints
- Use kebab-case in URIs, snake_case in method names

### ❌ Don't
- Don't return raw model data — use Resources
- Don't use `Route::resource()` — define each route explicitly
- Don't skip Swagger annotations
- Don't return HTML/redirect responses — JSON only
- Don't use session-based auth — JWT only
