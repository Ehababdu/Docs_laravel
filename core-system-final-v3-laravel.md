# 🏗️ Core System — النسخة النهائية المُصحَّحة

> النسخة 3.0 — تطبيق كامل لجميع الملاحظات. لا غموض، لا ثغرات.

-----

## 📋 جدول المحتويات

1. [Actions vs Services — القاعدة النهائية](#1-actions-vs-services)
1. [Base Classes — تنفيذ آمن وكامل](#2-base-classes)
1. [HasAudit Trait — نسخة مُقوّاة](#3-hasaudit-trait)
1. [Telescope — حماية كاملة](#4-telescope)
1. [Module Boundaries — منع التشابك](#5-module-boundaries)

-----

## 1. Actions vs Services

### القاعدة النهائية

```
Action  → عملية واحدة، متزامنة، لمرة واحدة، تُطلق Events
Service → منطق مشترك، يُستدعى من أكثر من مكان
Job     → عملية واحدة، غير متزامنة، قابلة للـ retry في Queue
```

### شجرة القرار

```
هل العملية تعمل في الخلفية (async / queueable)؟
  ← نعم → Job (implements ShouldQueue)
  ← لا ↓

هل المنطق يُستدعى من أكثر من Controller أو Module؟
  ← نعم → Service
  ← لا ↓

هل العملية تحتاج خطوات متسلسلة معقدة (Pipeline)؟
  ← نعم → Action + Pipeline
  ← لا → Action بسيطة
```

### Action — النمط الصحيح

```php
<?php
// app/Modules/Customers/Actions/CreateCustomerAction.php

namespace App\Modules\Customers\Actions;

final class CreateCustomerAction
{
    public function __construct(
        private readonly CustomerRepository $repository,
    ) {}

    public function handle(array $data): Customer
    {
        $data['credit_limit'] = $this->calculateInitialCreditLimit($data);

        $customer = $this->repository->create($data);

        // ✅ Events مقبولة في Action — هي إعلان عن حدث، ليست منطقًا
        event(new CustomerCreated($customer));

        return $customer;
    }

    private function calculateInitialCreditLimit(array $data): float
    {
        return match($data['customer_type'] ?? 'standard') {
            'premium' => 5000.0,
            default   => 1000.0,
        };
    }
}
```

### Action + Pipeline — للخطوات المتسلسلة

```php
<?php
// app/Modules/Orders/Actions/CreateOrderAction.php

namespace App\Modules\Orders\Actions;

use Illuminate\Pipeline\Pipeline;

final class CreateOrderAction
{
    public function __construct(
        private readonly OrderRepository $repository,
        private readonly Pipeline $pipeline,
    ) {}

    public function handle(array $data): Order
    {
        return $this->pipeline
            ->send($data)
            ->through([
                ValidateStockPipe::class,
                CalculateShippingPipe::class,
                ApplyDiscountPipe::class,
            ])
            ->then(fn (array $processed) => $this->repository->create($processed));
    }
}
```

### Job — للعمليات غير المتزامنة

```php
<?php
// app/Modules/Invoices/Jobs/SendInvoiceReminderJob.php

namespace App\Modules\Invoices\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;

final class SendInvoiceReminderJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable;

    public int $tries   = 3;
    public int $backoff = 60;

    public function __construct(
        private readonly string $invoiceId,
    ) {}

    public function handle(InvoiceRepository $repo): void
    {
        // منطق يعمل في الخلفية
    }
}
```

### Service — المنطق المشترك

```php
<?php
// app/Modules/Financing/Services/FinancingCalculatorService.php
// يُستدعى من: FinancingController, ReportController, CustomerController

namespace App\Modules\Financing\Services;

use App\Core\Abstracts\BaseService;

final class FinancingCalculatorService extends BaseService
{
    public function calculateMonthlyInstallment(
        float $principal,
        float $annualRate,
        int   $months,
    ): float {
        if ($annualRate === 0.0) {
            return round($principal / $months, 3);
        }

        $r = $annualRate / 12 / 100;

        return round(
            $principal * ($r * pow(1 + $r, $months)) / (pow(1 + $r, $months) - 1),
            3
        );
    }

    public function calculateTotalProfit(float $principal, float $rate, int $months): float
    {
        $monthly = $this->calculateMonthlyInstallment($principal, $rate, $months);

        return round(($monthly * $months) - $principal, 3);
    }
}
```

-----

## 2. Base Classes

### BaseModel — آمن ومُقيَّد

```php
<?php
// app/Core/Abstracts/BaseModel.php

namespace App\Core\Abstracts;

use Illuminate\Database\Eloquent\Model;

abstract class BaseModel extends Model
{
    // دعم UUID
    public    $incrementing = false;
    protected $keyType      = 'string';

    // ✅ حماية كاملة افتراضيًا — كل Model تحدد fillable صراحةً
    // $guarded = [] خطر لأن المطور قد ينسى $fillable
    protected $guarded = ['*'];

    protected $casts = [
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
    ];

    // ✅ إجبار كل Model على تحديد الحقول القابلة للتعبئة
    abstract protected function getFillableAttributes(): array;

    public function __construct(array $attributes = [])
    {
        $this->fillable = $this->getFillableAttributes();
        parent::__construct($attributes);
    }
}
```

```php
// مثال استخدام:
class Customer extends BaseModel
{
    use HasUuid, HasAudit;

    protected function getFillableAttributes(): array
    {
        return ['name', 'phone', 'national_id', 'credit_limit', 'status'];
        // created_by / updated_by غير موجودة هنا — HasAudit يملأها تلقائيًا
    }
}
```

### BaseRepository — Type-Safe + Transactions

```php
<?php
// app/Core/Abstracts/BaseRepository.php

namespace App\Core\Abstracts;

use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Pagination\LengthAwarePaginator;
use Illuminate\Support\Facades\DB;

/**
 * @template T of BaseModel
 */
abstract class BaseRepository
{
    /**
     * @param class-string<T> $modelClass
     */
    public function __construct(
        protected readonly string $modelClass,
    ) {}

    /**
     * @return T|null
     */
    public function find(string $id): ?Model
    {
        return $this->modelClass::find($id);
    }

    /**
     * @return T
     */
    public function findOrFail(string $id): Model
    {
        return $this->modelClass::findOrFail($id);
    }

    /**
     * @return Collection<int, T>
     */
    public function all(): Collection
    {
        return $this->modelClass::all();
    }

    /**
     * @return LengthAwarePaginator<T>
     */
    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return $this->modelClass::latest()->paginate($perPage);
    }

    /**
     * @return T
     */
    public function create(array $data): Model
    {
        return $this->modelClass::create($data);
    }

    public function update(Model $model, array $data): bool
    {
        return $model->update($data);
    }

    public function delete(Model $model): bool
    {
        return (bool) $model->delete();
    }

    /**
     * تغليف العمليات في Transaction
     *
     * @template TReturn
     * @param callable(): TReturn $callback
     * @return TReturn
     */
    public function transaction(callable $callback): mixed
    {
        return DB::transaction($callback);
    }
}
```

```php
// مثال Repository محدد:
/** @extends BaseRepository<Customer> */
class CustomerRepository extends BaseRepository
{
    public function __construct()
    {
        parent::__construct(Customer::class);
    }

    // IDE يعرف أن find() ترجع Customer|null تلقائيًا

    public function findByNationalId(string $nationalId): ?Customer
    {
        return Customer::where('national_id', $nationalId)->first();
    }

    public function findActiveWithFinancings(): Collection
    {
        return Customer::active()->with('financings')->get();
    }
}
```

```php
// مثال Transaction في Repository:
class OrderRepository extends BaseRepository
{
    public function __construct()
    {
        parent::__construct(Order::class);
    }

    public function createWithItems(array $orderData, array $items): Order
    {
        return $this->transaction(function () use ($orderData, $items) {
            $order = $this->create($orderData);
            $order->items()->createMany($items);
            return $order;
        });
    }
}
```

### BaseService — فارغة عن قصد

```php
<?php
// app/Core/Abstracts/BaseService.php

namespace App\Core\Abstracts;

// فارغة عن قصد — قيمتها الوحيدة:
// 1. تعريف الـ Services كصنف واضح في الكود
// 2. نقطة توسعة مستقبلية إذا احتجت منطقًا مشتركًا
abstract class BaseService
{
    // intentionally empty
}
```

-----

## 3. HasAudit Trait

### التنفيذ الكامل المُقوّى

```php
<?php
// app/Core/Traits/HasAudit.php

namespace App\Core\Traits;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Log;

trait HasAudit
{
    // ✅ تحكم دقيق — يُعطّل HasAudit فقط بدون تعطيل بقية Events
    private static bool $auditEnabled = true;

    // ─────────────────────────────────────────
    // Boot
    // ─────────────────────────────────────────

    public static function bootHasAudit(): void
    {
        static::creating(function (self $model): void {
            if (! self::$auditEnabled) {
                return;
            }

            $userId = self::resolveAuditUser();

            if ($userId) {
                $model->created_by ??= $userId;
                $model->updated_by ??= $userId;
            }
        });

        static::updating(function (self $model): void {
            if (! self::$auditEnabled) {
                return;
            }

            $userId = self::resolveAuditUser();

            if ($userId) {
                $model->updated_by = $userId;
            }
        });

        // ✅ تتبع من حذف السجل (Soft Deletes فقط)
        static::deleting(function (self $model): void {
            if (! self::$auditEnabled) {
                return;
            }

            // نتحقق أن الموديل يستخدم SoftDeletes وليس Hard Delete
            if (! in_array('Illuminate\Database\Eloquent\SoftDeletes', class_uses_recursive($model))) {
                return;
            }

            $userId = self::resolveAuditUser();

            if ($userId) {
                $model->deleted_by = $userId;
                $model->saveQuietly(); // saveQuietly لتجنب triggering updating event مرة أخرى
            }
        });
    }

    // ─────────────────────────────────────────
    // User Resolution
    // ─────────────────────────────────────────

    /**
     * يمكن تجاوزه في الموديل إذا احتجت منطقًا خاصًا
     */
    protected static function resolveAuditUser(): ?string
    {
        // حالة 1: مستخدم مسجّل دخول (HTTP request عادي)
        if (Auth::check()) {
            return (string) Auth::id();
        }

        // حالة 2: Queue أو Artisan Command — استخدم system user
        if (app()->runningInConsole()) {
            $systemUserId = config('system.system_user_id');

            if (! $systemUserId) {
                Log::warning('HasAudit: system.system_user_id غير مُعرَّف في config/system.php', [
                    'model' => static::class,
                ]);
            }

            return $systemUserId ? (string) $systemUserId : null;
        }

        // حالة 3: HTTP request بدون auth — غير متوقع، سجّل تحذير
        Log::warning('HasAudit: لا يوجد مستخدم في HTTP context', [
            'model' => static::class,
            'url'   => request()?->fullUrl(),
        ]);

        return null;
    }

    // ─────────────────────────────────────────
    // Helpers
    // ─────────────────────────────────────────

    /**
     * تعطيل HasAudit فقط — بقية Events تعمل بشكل طبيعي
     *
     * @template T
     * @param callable(): T $callback
     * @return T
     */
    public static function withoutAudit(callable $callback): mixed
    {
        self::$auditEnabled = false;

        try {
            return $callback();
        } finally {
            // يُعاد التفعيل دائمًا حتى لو حصل exception
            self::$auditEnabled = true;
        }
    }

    // ─────────────────────────────────────────
    // Relations
    // ─────────────────────────────────────────

    public function creator(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(config('auth.providers.users.model'), 'created_by');
    }

    public function updater(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(config('auth.providers.users.model'), 'updated_by');
    }

    public function deleter(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(config('auth.providers.users.model'), 'deleted_by');
    }
}
```

### HasUuid

```php
<?php
// app/Core/Traits/HasUuid.php

namespace App\Core\Traits;

use Illuminate\Support\Str;

trait HasUuid
{
    public static function bootHasUuid(): void
    {
        static::creating(function (self $model): void {
            $model->{$model->getKeyName()} ??= (string) Str::uuid();
        });
    }
}
```

### Migration — الأعمدة الكاملة

```php
// في كل migration لجدول يستخدم HasAudit:
Schema::create('customers', function (Blueprint $table) {
    $table->uuid('id')->primary();

    // ... بقية الأعمدة ...

    // HasAudit columns
    $table->foreignUuid('created_by')->nullable()->constrained('users')->nullOnDelete();
    $table->foreignUuid('updated_by')->nullable()->constrained('users')->nullOnDelete();
    $table->foreignUuid('deleted_by')->nullable()->constrained('users')->nullOnDelete(); // إذا كان SoftDeletes

    $table->timestamps();
    $table->softDeletes(); // إذا كان مطلوبًا
});
```

### config/system.php

```php
<?php
// config/system.php

return [
    /*
    |  معرّف المستخدم النظامي — يُستخدم في Queue و Artisan Commands
    |  أضف في .env: SYSTEM_USER_ID=uuid-of-system-user
    */
    'system_user_id' => env('SYSTEM_USER_ID'),
];
```

### استخدامات HasAudit

```php
// ✅ HTTP request عادي — تلقائي بالكامل
$customer = Customer::create($data);
// created_by = auth()->id() ✅

// ✅ Queue Job — تلقائي عبر system_user_id
// في .env: SYSTEM_USER_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

// ✅ تعطيل HasAudit فقط — Seeder مثلاً
Customer::withoutAudit(function () {
    Customer::create($data); // لا يُسجّل created_by
    // لكن Events الأخرى (Search Index, Cache Clear...) تعمل ✅
});

// ❌ لا تستخدم withoutEvents — يُعطّل كل شيء
Customer::withoutEvents(fn() => Customer::create($data)); // ❌
```

-----

## 4. Telescope

### TelescopeServiceProvider — الحماية الكاملة

```php
<?php
// app/Providers/TelescopeServiceProvider.php

namespace App\Providers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;
use Laravel\Telescope\TelescopeApplicationServiceProvider;

class TelescopeServiceProvider extends TelescopeApplicationServiceProvider
{
    public function register(): void
    {
        if (! $this->shouldEnable()) {
            return;
        }

        Telescope::night(); // واجهة داكنة — اختياري

        $this->hideSensitiveData();

        parent::register();
    }

    public function boot(): void
    {
        if (! $this->shouldEnable()) {
            // ✅ حظر Route حتى لو سُجّلت في ServiceProvider آخر
            $this->app['router']->get('telescope{any}', function () {
                abort(404);
            })->where('any', '.*');

            return;
        }

        parent::boot();
    }

    // ─────────────────────────────────────────

    private function shouldEnable(): bool
    {
        return $this->app->isLocal()
            || $this->app->runningUnitTests()
            || (
                $this->app->environment('staging')
                && config('telescope.enabled_staging', false)
            );
    }

    private function hideSensitiveData(): void
    {
        // ✅ إخفاء الحقول الحساسة تلقائيًا
        Telescope::hideSensitiveRequestDetails();

        Telescope::filter(function (IncomingEntry $entry): bool {
            // ✅ حذف البيانات الحساسة من Request entries
            if ($entry->type === 'request') {
                $sensitiveKeys = [
                    'password',
                    'password_confirmation',
                    'current_password',
                    'credit_card',
                    'card_number',
                    'cvv',
                    'national_id',
                    'token',
                    'secret',
                ];

                $content = $entry->content;

                foreach ($sensitiveKeys as $key) {
                    if (isset($content['payload'][$key])) {
                        $content['payload'][$key] = '***HIDDEN***';
                    }
                }

                $entry->content = $content;
            }

            return true;
        });
    }
}
```

### الإعدادات الكاملة

```php
// config/telescope.php — القيم المهمة فقط
return [
    'enabled' => env('TELESCOPE_ENABLED', false), // false افتراضي — Secure by Default

    'enabled_staging' => env('TELESCOPE_ENABLED_STAGING', false),

    'path' => env('TELESCOPE_PATH', 'telescope'),

    'driver' => env('TELESCOPE_DRIVER', 'database'),

    'storage' => [
        'database' => [
            'connection' => env('DB_CONNECTION', 'mysql'),
            'chunk'      => 1000,
        ],
    ],

    // احتفظ بالسجلات 48 ساعة فقط في Dev
    'prune' => [
        'hours' => env('TELESCOPE_PRUNE_HOURS', 48),
    ],
];
```

```bash
# .env.local
TELESCOPE_ENABLED=true

# .env.production
TELESCOPE_ENABLED=false
# لا تضع TELESCOPE_ENABLED في .env.production أصلاً — القيمة الافتراضية false تكفي

# .env.staging (اختياري)
TELESCOPE_ENABLED=false
TELESCOPE_ENABLED_STAGING=false  # true فقط للتشخيص المؤقت
```

### جدول نهائي — Dev vs Staging vs Production

|الأداة       |Dev    |Staging    |Production|
|-------------|-------|-----------|----------|
|**Telescope**|✅      |⚠️ بإذن صريح|❌         |
|**Debugbar** |✅      |❌          |❌         |
|**Sentry**   |اختياري|✅          |✅         |
|**Horizon**  |✅      |✅          |✅         |
|**Pulse**    |✅      |✅          |✅         |

-----

## 5. Module Boundaries

### القاعدة الصارمة

```
Module يستورد من → Core فقط
Module لا يستورد من → Module آخر مباشرةً أبدًا
```

### ❌ المشكلة — Circular / Direct Import

```php
// ❌ خطأ في Financing Module
use App\Modules\Customers\Models\Customer;     // import مباشر من module آخر
use App\Modules\Customers\Services\CustomerService;
```

### ✅ الحل — Contracts في Core

```php
// app/Core/Contracts/CustomerRepositoryInterface.php
namespace App\Core\Contracts;

interface CustomerRepositoryInterface
{
    public function find(string $id): ?object;
    public function findByNationalId(string $id): ?object;
    public function getCreditLimit(string $customerId): float;
}
```

```php
// app/Modules/Customers/Repositories/CustomerRepository.php
// يُنفّذ الـ Interface
class CustomerRepository extends BaseRepository implements CustomerRepositoryInterface
{
    public function getCreditLimit(string $customerId): float
    {
        return (float) Customer::findOrFail($customerId)->credit_limit;
    }
}
```

```php
// app/Providers/AppServiceProvider.php
// ربط الـ Interface بالتنفيذ
public function register(): void
{
    $this->app->bind(
        \App\Core\Contracts\CustomerRepositoryInterface::class,
        \App\Modules\Customers\Repositories\CustomerRepository::class,
    );
}
```

```php
// ✅ صح في Financing Module — يستخدم Interface من Core
namespace App\Modules\Financing\Services;

use App\Core\Contracts\CustomerRepositoryInterface; // ✅ من Core

class FinancingService extends BaseService
{
    public function __construct(
        private readonly CustomerRepositoryInterface $customerRepo, // ✅
    ) {}
}
```

### ✅ الحل الثاني — Events للتواصل بين Modules

```php
// Customers Module يُطلق Event
// app/Modules/Customers/Events/CustomerCreditLimitUpdated.php
class CustomerCreditLimitUpdated
{
    public function __construct(
        public readonly string $customerId,
        public readonly float  $newLimit,
    ) {}
}
```

```php
// Financing Module يستمع — بدون أي import من Customers
// app/Modules/Financing/Listeners/RecalculateFinancingOnCreditChange.php
class RecalculateFinancingOnCreditChange
{
    public function handle(CustomerCreditLimitUpdated $event): void
    {
        // $event->customerId, $event->newLimit
        // لا import من Customers Module — فقط من Core\Contracts
    }
}
```

```php
// app/Providers/EventServiceProvider.php
protected $listen = [
    CustomerCreditLimitUpdated::class => [
        RecalculateFinancingOnCreditChange::class,
        UpdateCustomerRiskProfile::class,
    ],
];
```

### خريطة الـ Dependencies المسموحة

```
┌─────────────────────────────────────────┐
│              Core                       │
│  Abstracts, Traits, Contracts, Events   │
└────────────────┬────────────────────────┘
                 │ يستورد منها الكل
    ┌────────────┼────────────┐
    ↓            ↓            ↓
┌────────┐  ┌──────────┐  ┌──────────┐
│Customers│  │Financing │  │ Invoices │
│        │  │          │  │          │
└────────┘  └──────────┘  └──────────┘
    ↑ Events فقط ↑              ↑
    └────────────┘──────────────┘
    لا imports مباشرة بين Modules
```

-----

## 📐 القواعد النهائية الشاملة

```
1.  Action  = عملية واحدة متزامنة، يجوز لها إطلاق Events
2.  Service = منطق مشترك من أكثر من مكان
3.  Job     = عملية واحدة غير متزامنة في Queue
4.  BaseModel → $guarded = ['*'] — كل Model تحدد fillable صراحةً
5.  BaseRepository → transaction() موجودة، Generic DocBlocks موجودة
6.  HasAudit → resolveAuditUser() تتعامل مع 3 حالات
7.  HasAudit → withoutAudit() بدلاً من withoutEvents()
8.  Telescope → ثلاث طبقات + حظر Route + إخفاء البيانات الحساسة
9.  TELESCOPE_ENABLED=false افتراضي في config
10. Module → لا يستورد من Module آخر — فقط Core Contracts أو Events
```

-----

*النسخة النهائية — يونيو 2026*