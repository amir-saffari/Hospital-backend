---
applyTo: "app/Listeners/**"
---

# Listeners



## Listener Structure

```php
<?php

namespace App\Listeners\V1\Admin;

use App\Events\V1\Queue\Admin\EntityCreatedEvent;

class EntityCreatedListener
{
    public function handle(EntityCreatedEvent $event): void
    {
        // Side-effect logic: metrics, logging, notifications, etc.
    }
}
```

---

## Registration

### Auto-Discovery (Laravel 11+)
Listeners are auto-discovered when they follow naming conventions.

### Manual Registration (EventServiceProvider)
```php
protected $listen = [
    EntityCreatedEvent::class => [
        EntityCreatedListener::class,
    ],
];
```

---

## Organization Pattern

```
app/Listeners/
├── V1/
│   ├── Admin/
│   │   ├── Entity/
│   │   │   └── EntityCreatedListener.php
│   │   └── OtherEntity/
│   │       └── OtherListener.php
│   └── Client/
│       └── Entity/
│           └── EntityListener.php
```

---

## Do's and Don'ts

### ✅ Do
- Keep listeners focused on a single side-effect
- Use the event's public properties to access data
- Log errors within listeners

### ❌ Don't
- Don't put broadcast logic in listeners — use Broadcast events
- Don't perform heavy synchronous operations without considering queue impact
- Don't create circular event chains (listener dispatching events that trigger the same listener)
