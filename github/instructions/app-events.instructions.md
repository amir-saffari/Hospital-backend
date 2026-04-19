---
applyTo: "app/Events/**"
---

# Events



## Event Categories

### Broadcast Events (Real-time)
Implement `ShouldBroadcast` — push data to WebSocket clients immediately:
```php
<?php

namespace App\Events\V1\Broadcast\Shared;

use Illuminate\Broadcasting\Channel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Queue\SerializesModels;

class EntityUpdated implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(
        public int $entity_id,
        public array $data,
        public string $type,
    ) {}

    public function broadcastOn(): Channel
    {
        return new Channel("entity.{$this->entity_id}");
    }
}
```

### Queue Events (Async)
Implement `ShouldQueue` — processed by queue workers for side effects:
```php
<?php

namespace App\Events\V1\Queue;

use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Broadcasting\InteractsWithSockets;

abstract class EventCore implements ShouldQueue
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function backoff(): array
    {
        return [3];
    }
}

class EntityCreatedEvent extends EventCore
{
    public function __construct(
        public Entity $entity,
        public User $user,
    ) {}
}
```

### Event Groups (Aggregators)
Static dispatchers that fire multiple related events atomically:
```php
<?php

namespace App\Events\V1\Groups;

class EntityCreatedEvents
{
    public static function dispatch(Entity $entity, User $user, array $extra)
    {
        // Broadcast to WebSocket
        EntityUpdated::dispatch($entity->id, $extra, 'entity_created');

        // Queue for async processing (metrics, notifications)
        EntityCreatedEvent::dispatch($entity, $user);
    }
}
```

---

## Channel Naming Patterns

Define your channel naming convention:

| Channel Pattern | Purpose |
|----------------|---------|
| `{entity}.{id}` | Per-entity real-time updates |
| `updates.{tenant_id}` | Tenant-wide updates |
| `personal.{user_type}.{user_id}` | User-specific notifications |
| `general` | System-wide broadcasts |

---

## Do's and Don'ts

### ✅ Do
- Use event groups to aggregate related broadcast + queue events
- Use constructor promotion for event properties
- Fire events from controllers, not from models or services
- Include event type identifiers in broadcast payloads
- Extend a base class for queue events (consistent backoff, error handling)

### ❌ Don't
- Don't fire broadcast events individually when they're part of an atomic operation — use groups
- Don't put heavy logic in event constructors
- Don't use broadcast events for async processing — use queue events
- Don't forget backoff strategies on queue events
