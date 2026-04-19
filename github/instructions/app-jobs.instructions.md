---
applyTo: "app/Jobs/**"
---

# Jobs



## Job Structure

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessExampleJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public array $data,
    ) {}

    public function handle(): void
    {
        // Job logic here
    }

    public function failed(\Throwable $exception): void
    {
        \Log::error("ProcessExampleJob failed: {$exception->getMessage()}");
    }
}
```

---

## Dispatching

```php
// From controllers
ProcessExampleJob::dispatch($data);

// From middleware (post-response)
app()->terminating(fn() => ProcessExampleJob::dispatch($logData));

// Delayed dispatch
ProcessExampleJob::dispatch($data)->delay(now()->addMinutes(5));

// On specific queue
ProcessExampleJob::dispatch($data)->onQueue('high');
```

---

## Do's and Don'ts

### ✅ Do
- Use constructor promotion for job properties
- Keep `handle()` focused on a single task
- Log errors within jobs for debugging
- Use `ShouldQueue` for all async work
- Pass only serializable data via constructor

### ❌ Don't
- Don't access `auth()` in jobs — pass needed data via constructor
- Don't run heavy operations synchronously — dispatch a job
- Don't silently swallow exceptions without logging
- Don't pass request objects or closures to jobs
