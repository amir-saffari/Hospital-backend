---
applyTo: "routes/**, app/Http/Controllers/**, app/Http/Resources/**, app/Http/Requests/**"
---

# API Design

> Cross-cutting spec that defines the complete API contract design conventions.


## Response Envelope

### Success Response
```json
{
    "status": 200,
    "data": { }
}
```

### Error Response
```json
{
    "status": 400,
    "message": "Error description"
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

### Not Found (404)
```json
{
    "status": 404,
    "message": "Resource not found"
}
```

---

## Authentication Flow

### Token-Based (JWT/Sanctum)
```
POST   /api/v1/auth/login     → Returns token
POST   /api/v1/auth/register  → Returns token
POST   /api/v1/auth/refresh   → Returns new token
POST   /api/v1/auth/logout    → Invalidates token
GET    /api/v1/auth/profile   → Returns authenticated user
```

### Headers
```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

---

## Endpoint Conventions

| Action | Verb | Pattern |
|--------|------|---------|
| List | GET | `/api/v1/{entities}` |
| Get single | GET | `/api/v1/{entities}/{id}` |
| Create | POST | `/api/v1/{entities}` 
| Update | PUT | `/api/v1/{entities}/{id}` 
| Delete | DELETE | `/api/v1/{entities}/{id}` 
| Custom action | POST | `/api/v1/{entities}/{id}/{action}` |

---

## Do's and Don'ts

### ✅ Do
- Return data through API Resource classes
- Use Form Requests for all validation
- Add API documentation annotations to every endpoint
- Use consistent response format across all endpoints
- Include authentication on all non-public endpoints

### ❌ Don't
- Don't return raw model data — use Resources
- Don't return HTML or redirects — JSON only
- Don't mix response formats across endpoints
- Don't expose internal error details in production
