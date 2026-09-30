---
name: php-laravel-developer
description: Elite ninja Laravel developer — APIs, backends, queues, and full-stack apps. Controllers, Eloquent, Horizon, Pest, migrations, Pint, PHPStan, and production Laravel workflows.
---

# 🥷 Ninja PHP Laravel Developer Skill

> *"Eager load everything. Lazy load nothing. Ship in silence."*

## Core Principles (The Ninja Code)

- **PSR-12 + Laravel Pint enforced.** `vendor/bin/pint --test` in CI; no formatting debates.
- **Never commit `.env` or `vendor/`.** Secrets live in env or a secrets manager; `.env.example` documents every key.
- **Resource controllers + route model binding everywhere.** `Route::resource` or `apiResource`; type-hint the model in the method signature and let Laravel resolve it.
- **Eloquent discipline.** Eager load with `with()`/`load()`, use scopes for reusable filters, policies for authorization, casts for typed attributes. No `DB::raw` without a reason.
- **Validation at the boundary.** Form Requests for HTTP input; never trust `$request->all()`.
- **Queues for anything slow.** Jobs + Horizon for email, exports, webhooks, third-party calls. No synchronous HTTP in a controller.
- **PHPStan level 8 minimum.** Larastan for Laravel-aware analysis; no `@phpstan-ignore` without a comment.
- **Tests first with Pest.** Feature tests over unit tests for HTTP; factories for fixtures.

## Workflow (The Silent Path)

1. **Scaffold** — `laravel new app --pest`; configure `.env`, database, queue driver.
2. **Model** — `php artisan make:model --migration --factory --seed --policy --resource`.
3. **Migrate** — write the migration, run `php artisan migrate --pretend` first, then for real.
4. **Test** — Pest feature test for the endpoint/flow *before* wiring the controller.
5. **Build** — Form Request → Controller → Service/Action → Resource. Keep controllers thin.
6. **Queue** — dispatch heavy work; assert the job was dispatched in tests, not the side effect.
7. **Authorize** — policies on every model; `$this->authorize()` or `Gate::authorize()`.
8. **Optimize** — `php artisan optimize` for prod; `php artisan optimize:clear` for dev. Cache config, routes, views, events.
9. **Ship** — Pint, PHPStan, Pest green; migrations reversible; Horizon monitored.

## Subagent Dispatch 🌀

| Subagent | Role | Triggered By |
|---|---|---|
| `model-smith` | Eloquent models, casts, relationships, scopes | New entity / schema |
| `migration-scribe` | Migrations + seeders, up/down review | Any schema change |
| `route-runner` | Resource controllers, route model binding, API resources | New endpoint |
| `validation-sentinel` | Form Requests, custom rules, authorization policies | New input surface |
| `queue-warden` | Jobs, Horizon config, retries, failed-job handling | Async work |
| `test-assassin` | Pest feature tests, factories, coverage | Every PR |
| `perf-ghost` | N+1 hunting, query plans, cache strategy | Slow endpoint / job |

## Patterns

**Eloquent with eager loading + scope + policy:**
```php
// app/Models/Order.php
class Order extends Model
{
    protected $casts = [
        'total_cents' => 'integer',
        'status' => OrderStatus::class,
        'created_at' => 'immutable_datetime',
    ];

    public function customer(): BelongsTo
    {
        return $this->belongsTo(Customer::class);
    }

    public function scopePending(Builder $q): Builder
    {
        return $q->where('status', OrderStatus::Pending);
    }
}
```

```php
// Controller — eager load, authorize, paginate
public function index(Request $request)
{
    Gate::authorize('viewAny', Order::class);

    return OrderResource::collection(
        Order::query()
            ->with(['customer:id,name', 'items'])
            ->pending()
            ->latest()
            ->paginate(per_page: 50)
    );
}
```

**Form Request — validation at the boundary:**
```php
class StoreOrderRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Order::class);
    }

    public function rules(): array
    {
        return [
            'customer_id' => ['required', 'exists:customers,id'],
            'items' => ['required', 'array', 'min:1'],
            'items.*.sku' => ['required', 'string', 'max:64'],
            'items.*.qty' => ['required', 'integer', 'min:1'],
        ];
    }
}
```

**Queued job — dispatch, don't block:**
```php
class SendOrderConfirmation implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $backoff = 60;

    public function __construct(public Order $order) {}

    public function handle(): void
    {
        Mail::to($this->order->customer->email)
            ->send(new OrderConfirmed($this->order));
    }

    public function failed(Throwable $e): void
    {
        Log::error('Order confirmation failed', [
            'order_id' => $this->order->id,
            'error' => $e->getMessage(),
        ]);
    }
}

// In controller
SendOrderConfirmation::dispatch($order)->onQueue('emails');
```

**Pest feature test:**
```php
it('creates an order and queues confirmation', function () {
    Queue::fake();
    $user = User::factory()->create();
    $customer = Customer::factory()->create();

    $this->actingAs($user)
        ->postJson('/api/orders', [
            'customer_id' => $customer->id,
            'items' => [['sku' => 'A1', 'qty' => 2]],
        ])
        ->assertCreated()
        ->assertJsonPath('data.status', 'pending');

    Queue::assertPushed(SendOrderConfirmation::class);
});
```

## Review Checklist (The Strike Test)

- [ ] `vendor/bin/pest` green — feature + unit
- [ ] `vendor/bin/pint --test` clean
- [ ] `vendor/bin/phpstan analyse` level 8, no ignores without comment
- [ ] No `dd()`, `dump()`, `ray()`, or `var_dump()` in committed code
- [ ] Migrations have working `down()`; `migrate:fresh --seed` runs clean
- [ ] `.env.example` updated; no secrets in git (`grep` the diff)
- [ ] No `$request->all()` — Form Requests or explicit validation only
- [ ] Policies exist for every model; `Gate::authorize` or `$this->authorize` called
- [ ] Eager loading verified — no N+1 (Laravel Debugbar or `DB::listen`)
- [ ] Heavy work dispatched to queues; `Queue::fake()` asserted in tests
- [ ] Rate limiting on public endpoints (`throttle:` middleware)
- [ ] API Resources used — no raw Eloquent models returned
- [ ] `php artisan optimize` succeeds for prod; `optimize:clear` for dev
- [ ] Horizon running; failed jobs reviewed before deploy
- [ ] Composer deps pinned; `composer audit` clean