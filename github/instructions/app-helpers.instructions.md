---
applyTo: "app/Helpers/**"
---

# Helpers



## Helper Structure

```php
<?php

// app/Helpers/function.php

if (!function_exists('example_helper')) {
    function example_helper(): string
    {
        return 'value';
    }
}
```

### Composer Autoload
```json
{
    "autoload": {
        "files": [
            "app/Helpers/function.php"
        ]
    }
}
```

---

## Do's and Don'ts

### ✅ Do
- Wrap in `function_exists()` guard
- Keep helpers minimal and framework-agnostic
- Only add globally-needed utilities

### ❌ Don't
- Don't put business logic in helpers
- Don't duplicate functionality available in services
- Don't add class methods as helpers — use services
