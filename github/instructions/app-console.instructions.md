---
applyTo: "app/Console/**"
---

# Console Commands



## Command Structure

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ExampleCommand extends Command
{
    protected $signature = 'example:run {--force : Force the operation}';
    protected $description = 'Run the example task';

    public function handle(): int
    {
        $this->info('Starting...');

        // Command logic

        $this->info('Done.');
        return self::SUCCESS;
    }
}
```

---

## Do's and Don'ts

### ✅ Do
- Use `$this->info()`, `$this->error()` for output
- Return status codes (`self::SUCCESS`, `self::FAILURE`)
- Add descriptive `$signature` with options and arguments

### ❌ Don't
- Don't put scheduled job logic in commands — use Job classes
- Don't access `auth()` in commands — there is no HTTP context
- Don't create commands for one-time operations — use tinker
