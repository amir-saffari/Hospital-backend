---
applyTo: "app/Notifications/**"
---

# Notifications



## Notification Structure

### With Custom Base Class
```php
<?php

namespace App\Notifications;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

abstract class NotificationCore extends Notification implements ShouldQueue
{
    protected array $channels = [];

    public function via($notifiable): array
    {
        return array_merge([DatabaseChannel::class], $this->channels);
    }
}
```

### Concrete Notification
```php
class EntityCreatedNotification extends NotificationCore
{
    protected array $channels = [/* additional channels */];

    public function __construct(
        public Entity $entity,
        public User $actor,
    ) {}

    public function toDatabase($notifiable): array
    {
        return [
            'title' => 'Entity Created',
            'body' => "New entity: {$this->entity->name}",
            'type' => 'entity_created',
            'data' => json_encode(['entity_id' => $this->entity->id]),
        ];
    }
}
```

---

## Custom Channels

### Database Channel (custom table)
```php
class DatabaseChannel
{
    public function send($notifiable, Notification $notification): void
    {
        $data = $notification->toDatabase($notifiable);

        try {
            CustomNotification::create([
                'notifiable_type' => get_class($notifiable),
                'notifiable_id' => $notifiable->id,
                ...$data,
            ]);
        } catch (\Throwable $e) {
            // Handle duplicate constraint violations gracefully
            if (!str_contains($e->getMessage(), '23505')) throw $e;
        }
    }
}
```

### External Service Channel (e.g., Telegram, Slack)
```php
class ExternalChannel
{
    public function send($notifiable, Notification $notification): void
    {
        $data = $notification->toExternal($notifiable);
        ExternalService::send($data);
    }
}
```

---

## Do's and Don'ts

### ✅ Do
- Extend a base notification class for consistency
- Queue all notifications for async delivery
- Handle deduplication via database constraints where possible
- Store external message IDs for later reference (edit/delete)

### ❌ Don't
- Don't send notifications synchronously for non-critical alerts
- Don't skip the notification system for direct channel calls
- Don't put heavy logic in notification constructors
