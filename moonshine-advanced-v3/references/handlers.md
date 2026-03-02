# Handlers, Notifications & Toasts Reference

MoonShine v3 handler system, notification center, and toast notification API.

## Handler Full API

Source: `MoonShine\Laravel\Handlers\Handler`

Handler is an abstract class implementing: `HasIconContract`, `HasResourceContract`, `HasUriKeyContract`, `HasLabelContract`, `HasCoreContract`.

### Traits Used

- `Makeable` -- static `make()` factory
- `WithQueue` -- queue support
- `HasResource` -- resource access
- `WithIcon` -- icon support
- `WithUriKey` -- URI key generation
- `WithLabel` -- label management
- `WithCore` -- core access
- `Conditionable` -- conditional method chaining

### Constructor

```php
public function __construct(Closure|string $label)
```

Static factory: `Handler::make('My Handler')`

### Abstract Methods

```php
abstract public function handle(): Response;
abstract public function getButton(): ActionButtonContract;
```

### Concrete Methods

#### `getUrl(): string`

Returns the handler's execution URL using the resource's `handler` route with the handler's URI key.

```php
$url = $handler->getUrl();
// e.g., /admin/resource/post-resource/index-page/handler?handlerUri=my-custom-handler
```

#### `modifyButton(Closure $callback): static`

Customize the ActionButton before rendering.

```php
$handler->modifyButton(function (ActionButtonContract $button, Handler $ctx) {
    return $button->icon('heroicons.bolt')->class('btn-primary');
});
```

#### `notifyUsers(array|Closure $ids): static`

Set which admin user IDs receive notifications after handler execution.

```php
$handler->notifyUsers([1, 2, 3]);
// or dynamically
$handler->notifyUsers(fn(Handler $ctx) => User::admin()->pluck('id')->toArray());
```

#### `getNotifyUsers(): array`

Returns resolved array of user IDs.

### Handler Registration in Resource

```php
class PostResource extends ModelResource
{
    protected function handlers(): ListOf
    {
        return parent::handlers()->add(
            MyCustomHandler::make('Import Posts')
                ->notifyUsers([1])
                ->modifyButton(fn($btn, $ctx) => $btn->icon('heroicons.arrow-up-tray'))
        );
    }
}
```

After registration, the handler button appears automatically on the resource's index page.

### Queue Support

```php
public function handle(): Response
{
    if ($this->isQueue()) {
        ImportPostsJob::dispatch();
        MoonShineUI::toast(__('moonshine::ui.resource.queued'));
        return back();
    }

    // Synchronous processing
    self::process();
    return back();
}
```

### Generating a Handler

```shell
php artisan moonshine:handler {className?} {--base-dir=} {--base-namespace=}
```

Creates a class in `app/MoonShine/Handlers/`.

---

## Notifications

MoonShine uses Laravel Database Notifications by default but abstracts them behind contracts.

### Sending Notifications

```php
use MoonShine\Laravel\Notifications\MoonShineNotification;
use MoonShine\Laravel\Notifications\NotificationButton;
use MoonShine\Support\Enums\Color;

MoonShineNotification::send(
    message: 'Notification text',
    button: new NotificationButton('Click me', 'https://example.com', attributes: ['target' => '_blank']),
    ids: [1, 2, 3],       // null = send to all admins
    color: Color::GREEN,
    icon: 'information-circle'
);
```

Via DI:
```php
use MoonShine\Laravel\Contracts\Notifications\MoonShineNotificationContract;

public function myMethod(MoonShineNotificationContract $notification)
{
    $notification->notify('Hello');
}
```

### Configuration

```php
// config/moonshine.php
'use_notifications' => true,
'use_database_notifications' => true,

// Or via MoonShineServiceProvider
$config->useNotifications()->useDatabaseNotifications();
```

### Custom Notification System

Implement these interfaces:
- `MoonShineNotificationContract`
- `NotificationItemContract`
- `NotificationButtonContract` (optional)

Register in ServiceProvider:
```php
$this->app->singleton(MoonShineNotificationContract::class, MyNotificationSystem::class);
```

### WebSocket Notifications

Available via the [Rush](https://moonshine-laravel.com/plugins/rush) package.

---

## Toasts

Toast notifications use `session()->flash()` and can be triggered from controllers, JSON responses, or JS events.

### Flash (Session-Based)

```php
use MoonShine\Laravel\MoonShineUI;
use MoonShine\Support\Enums\ToastType;

MoonShineUI::toast('Success', ToastType::SUCCESS, duration: 3000);
MoonShineUI::toast('Sticky', duration: false);  // stays until clicked
```

ToastType values: `DEFAULT`, `SUCCESS`, `ERROR`, `WARNING`, `INFO`.

### Via MoonShineJsonResponse

```php
MoonShineJsonResponse::make()->toast('Test', type: ToastType::SUCCESS, duration: 1000);
```

### Via JS Events

```php
ActionButton::make('Toast')->dispatchEvent(
    AlpineJs::event(
        JsEvent::TOAST,
        params: ToastEventParams::make(ToastType::SUCCESS, 'Hello', duration: 2000)
    )
);
```

### Global Duration Override (JS)

```js
MoonShine.config().setToastDuration(5000);
```
