---
applyTo: "app/Http/Controllers/**"
---

# Controllers



## Response Helpers

### If Custom Base Controller
```php
<?php

namespace App\Http\Controllers;

class Controller
{
    protected function successResponse($data, $status = 200)
    {
        return response()->json([
            'status' => $status,
            'data' => $data,
        ], $status);
    }

    protected function errorResponse($message, $status = 400)
    {
        return response()->json([
            'status' => $status,
            'message' => $message,
        ], $status);
    }

    protected function notFoundResponse($message = 'Not found')
    {
        return response()->json([
            'status' => 404,
            'message' => $message,
        ], 404);
    }

    protected function forbiddenResponse($message = 'Forbidden')
    {
        return response()->json([
            'status' => 403,
            'message' => $message,
        ], 403);
    }
}
```

---

## Controller Structure

### Resource Controller Pattern
```php
<?php

namespace App\Http\Controllers\V1\Admin;

use App\Http\Controllers\Controller;
use App\Http\Requests\V1\Admin\Example\CreateExampleRequest;
use App\Http\Requests\V1\Admin\Example\UpdateExampleRequest;
use App\Http\Resources\V1\Admin\Example\ExampleResource;
use App\Models\Example;
use Illuminate\Support\Facades\DB;

class ExampleController extends Controller
{
    public function table(Request $request)
    {
        $items = Example::query()
            ->allowedForUser()
            ->search($request->all())
            ->paginate(15);

        return $this->successResponse($items);
    }

    public function get(Example $example)
    {
        return $this->successResponse(new ExampleResource($example->load('relation')));
    }

    public function create(CreateExampleRequest $request)
    {
        DB::beginTransaction();
        try {
            $example = Example::create($request->validated());

            // Audit log (if enabled)
            // Event dispatch (if enabled)

            DB::commit();
            return $this->successResponse(new ExampleResource($example));
        } catch (\Throwable $th) {
            DB::rollBack();
            return $this->errorResponse($th->getMessage());
        }
    }

    public function update(UpdateExampleRequest $request, Example $example)
    {
        DB::beginTransaction();
        try {
            $example->update($request->validated());

            DB::commit();
            return $this->successResponse(new ExampleResource($example));
        } catch (\Throwable $th) {
            DB::rollBack();
            return $this->errorResponse($th->getMessage());
        }
    }

    public function delete(Example $example)
    {
        $example->delete();
        return $this->successResponse(['message' => 'Deleted']);
    }
}
```

---

## Swagger Annotations (if chosen)

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
 *     @OA\Response(response=200, description="Success"),
 *     @OA\Response(response=401, description="Unauthorized"),
 *     @OA\Response(response=422, description="Validation Error"),
 * )
 */
public function create(CreateExampleRequest $request) { ... }
```

---

## Key Patterns

### Route Model Binding
```php
public function get(Example $example) { ... }
public function update(UpdateExampleRequest $request, Example $example) { ... }
```

### Eager Loading
```php
// Prevent N+1 queries
$example->load(['relation', 'nested.relation']);
return $this->successResponse(new ExampleResource($example));
```

### Audit Logging (if enabled)
```php
AuditLog::create([
    'user_id' => auth()->id(),
    'action' => ActionEnum::CREATE,
    'description' => "Created example: {$example->name}",
    'payload' => json_encode($example->toArray()),
]);
```

### Deferred Operations
```php
// Heavy operations after HTTP response is sent
app()->terminating(function () use ($data) {
    ExternalService::notify($data);
});
```

---

## Do's and Don'ts

### ✅ Do
- Use Form Request classes for all validation and authorization
- Return data through API Resource classes
- Use `DB::beginTransaction()` for multi-step writes
- Eager load relationships to prevent N+1 queries
- Add API documentation annotations to every endpoint
- Fire events for real-time updates and side effects

### ❌ Don't
- Don't validate inline in controllers
- Don't return raw model data — use Resources
- Don't call external services synchronously — defer or queue
- Don't access `request()` directly when a Form Request is available
- Don't put complex business logic in controllers — extract to services
