---
applyTo: "app/Services/**"
---

# Services



## Service Structure

### If Static Methods
```php
<?php

namespace App\Services\Example;

use Illuminate\Support\Facades\Http;

class ExampleService
{
    public static function do_something(int $id): array
    {
        try {
            $response = Http::withHeaders([
                'Authorization' => 'Bearer ' . config('services.example.key'),
            ])->post(config('services.example.url'), [
                'id' => $id,
            ]);

            if ($response->failed()) {
                \Log::error("Example API failed: {$response->status()}");
                return [];
            }

            return $response->json();
        } catch (\Exception $e) {
            \Log::error("Example service error: {$e->getMessage()}");
            return [];
        }
    }
}
```

### If Constructor Injection
```php
<?php

namespace App\Services\Example;

class ExampleService
{
    public function __construct(
        private readonly ExampleClient $client,
    ) {}

    public function process(int $id): ExampleResult
    {
        return $this->client->fetch($id);
    }
}
```

### If Action Classes
```php
<?php

namespace App\Actions\Example;

class CreateExampleAction
{
    public function execute(array $data): Example
    {
        return Example::create($data);
    }
}
```

---

## Organization

### Domain Subdirectories
```
app/Services/
├── Auth/               # Authentication services
├── File/               # File storage & processing
├── Notification/       # Notification dispatch
├── Payment/            # Payment integration
├── Search/             # External search API
└── ThirdParty/         # External API clients
```

---

## Error Handling Patterns

### Return Fallback Values
```php
public static function fetch_data(int $id): array
{
    try {
        return Http::get($url)->throw()->json();
    } catch (\Exception $e) {
        \Log::error("Fetch failed: {$e->getMessage()}");
        return [];
    }
}
```

### Throw Exceptions
```php
public function process(array $data): Result
{
    $response = Http::post($url, $data);

    if ($response->failed()) {
        throw new ServiceException("Processing failed: {$response->status()}");
    }

    return new Result($response->json());
}
```

---

## Do's and Don'ts

### ✅ Do
- Organize services into domain subdirectories
- Catch exceptions and handle errors gracefully
- Log errors with contextual information
- Use the Http facade for external API calls
- Keep services focused on a single domain concern

### ❌ Don't
- Don't put controller/request logic in services
- Don't access `auth()` in services called from queued jobs
- Don't mix patterns (e.g., some services static, others injected) without documenting why
- Don't hardcode API keys — use config or environment variables
