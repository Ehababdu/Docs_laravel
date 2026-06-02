# 🔍 مراجعة تصميم معماري — Core System Missing Details

> **تاريخ المراجعة:** 2026-06-01  
> **المراجع:** مهندس معماري متخصص في Laravel & AI Agent Documentation  
> **التقييم العام:** جيد جدًا مع ملاحظات تحسينية مهمة

---

## 📊 ملخص التقييم

| القسم | التقييم | المشاكل | التحسينات |
|-------|---------|---------|-----------|
| 1. Actions vs Services | ⭐⭐⭐⭐⭐ ممتاز | 0 | 3 اقتراحات |
| 2. Base Classes | ⭐⭐⭐⭐☆ جيد جدًا | 2 | 3 إصلاحات |
| 3. HasAudit Trait | ⭐⭐⭐⭐☆ جيد جدًا | 2 | 3 إصلاحات |
| 4. Telescope Security | ⭐⭐⭐☆☆ مقبول | 3 | 3 إصلاحات أمنية |

---

## 1️⃣ قاعدة Actions vs Services

### ✅ ما هو ممتاز

| النقطة | التقييم |
|--------|---------|
| الفصل الواضح بين "عملية لمرة واحدة" و"خبير domain" | ✅ ممتاز |
| قاعدة القرار التدفقية (flowchart) | ✅ عملية جدًا |
| المثال العملي بالأرقام (FinancingCalculator) | ✅ واقعي |

### ⚠️ ملاحظات التحسين

#### أ. Action يمكنها استدعاء Events — وهذا مقبول

```php
// ✅ هذا مقبول — Action تطلق Event (ليس Service)
final class CreateCustomerAction
{
    public function handle(array $data): Customer
    {
        $customer = $this->repository->create($data);

        // ✅ Event هو notification وليس Service
        event(new CustomerCreated($customer));

        return $customer;
    }
}
```

> **الفرق الدقيق:** Event = "حدث حصل" (إعلان)، Service = "منطق يُنفّذ" (عمل).  
> Action تُطلق Events ✅ | Action تستدعي Services ❌

#### ب. نقص: ماذا عن Command Pattern؟

```php
// في بعض الحالات، Action تستحق أن تكون Command منفصلة
// إذا كانت العملية: async + queueable + retryable

final class SendInvoiceReminderCommand implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable;

    public function handle(InvoiceRepository $repo): void
    {
        // منطق يعمل في الخلفية — لا يناسب Action متزامنة
    }
}
```

#### ج. اقتراح إضافة: Action Pipeline

```php
// للعمليات المعقدة التي تحتاج خطوات متسلسلة
final class CreateOrderAction
{
    public function handle(array $data): Order
    {
        return app(Pipeline::class)
            ->send($data)
            ->through([
                ValidateStock::class,      // ✅ Action تستدعي Pipe وليس Service
                CalculateShipping::class,
                ApplyDiscount::class,
                ProcessPayment::class,
            ])
            ->then(fn ($data) => $this->repository->create($data));
    }
}
```

---

## 2️⃣ تنفيذ Base Classes

### ✅ ما هو ممتاز
- `BaseModel` مع UUID و `$guarded = []` — ✅ أمنيًا صحيح
- `BaseRepository` — ✅ تبسيط CRUD
- `BaseService` فارغة عن قصد — ✅ قرار معماري سليم

### 🚨 ثغرات أمنية ومعمارية

#### أ. BaseModel — `$guarded = []` خطر بدون Global Scope

```php
// ❌ المشكلة: أي موديل ينسى يحدد $fillable = ثغرة Mass Assignment
abstract class BaseModel extends Model
{
    protected $guarded = []; // خطر إذا نسى المطور $fillable
}

// ✅ الحل: إجبار المطور على تحديد $fillable
abstract class BaseModel extends Model
{
    protected $guarded = ['*']; // افتراضي: حماية كاملة

    abstract protected function getFillableAttributes(): array;

    public function __construct(array $attributes = [])
    {
        $this->fillable = $this->getFillableAttributes();
        parent::__construct($attributes);
    }
}

// الاستخدام:
class Customer extends BaseModel
{
    protected function getFillableAttributes(): array
    {
        return ['name', 'phone', 'national_id', 'credit_limit', 'status'];
    }
}
```

#### ب. BaseRepository — نقص في Type Safety

```php
// ❌ المشكلة: return type عام (Model) يفقد IDE autocomplete
abstract class BaseRepository
{
    public function find(string $id): ?Model; // ❌ لا يعرف أنه Customer
}

// ✅ الحل: Generics (PHP 8.3+) أو DocBlocks
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
}

// الاستخدام:
/** @extends BaseRepository<Customer> */
class CustomerRepository extends BaseRepository
{
    public function __construct()
    {
        parent::__construct(Customer::class);
    }

    // IDE الآن يعرف أن find() ترجع Customer|null
}
```

#### ج. BaseRepository — نقص في Transaction Handling

```php
// ✅ أضف هذا في BaseRepository
use Illuminate\Support\Facades\DB;

abstract class BaseRepository
{
    /**
     * @template T
     * @param callable(): T $callback
     * @return T
     */
    public function transaction(callable $callback): mixed
    {
        return DB::transaction($callback);
    }
}

// الاستخدام:
class OrderRepository extends BaseRepository
{
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

---

## 3️⃣ HasAudit Trait — تنفيذ صحيح

### ✅ ما هو ممتاز
- استخدام `bootHasAudit()` — ✅ طريقة Laravel السليمة
- `??=` لتجنب الكتابة فوق قيم موجودة — ✅
- Relations `creator()` و `updater()` — ✅

### 🚨 ثغرات أمنية

#### أ. Auth::check() في Queue يفشل silently

```php
// ❌ المشكلة: في Queue، Auth::check() = false → created_by = null
// هذا يُفقد معلومات مهمة بدون سجل خطأ

// ✅ الحل: Explicit System User + Logging
trait HasAudit
{
    public static function bootHasAudit(): void
    {
        static::creating(function (self $model): void {
            $userId = self::resolveAuditUser();
            if ($userId) {
                $model->created_by ??= $userId;
                $model->updated_by ??= $userId;
            }
        });

        static::updating(function (self $model): void {
            $userId = self::resolveAuditUser();
            if ($userId) {
                $model->updated_by = $userId;
            }
        });
    }

    /**
     * قابل للتجاوز في الموديلات الخاصة
     */
    protected static function resolveAuditUser(): ?string
    {
        if (Auth::check()) {
            return Auth::id();
        }

        // في Queue/Commands — استخدم system user
        if (app()->runningInConsole()) {
            return config('system.system_user_id');
        }

        // في API بدون auth (نادر) — سجل تحذير
        \Log::warning('HasAudit: No user resolved for ' . static::class, [
            'trace' => debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5),
        ]);

        return null;
    }
}
```

#### ب. نقص: Soft Deletes تحتاج `deleted_by`

```php
// ✅ أضف هذا في Migration
$table->foreignUuid('deleted_by')->nullable()->constrained('users')->nullOnDelete();

// ✅ أضف هذا في Trait
public static function bootHasAudit(): void
{
    // ... existing code ...

    static::deleting(function (self $model): void {
        if (Auth::check() && method_exists($model, 'forceDelete')) {
            // soft delete only
            $model->deleted_by = Auth::id();
            $model->saveQuietly(); // تجنب loop
        }
    });
}
```

#### ج. نقص: التعامل مع `withoutEvents`

```php
// ❌ المشكلة: withoutEvents يُعطل ALL events
// إذا كان هناك Event مهم آخر (مثل SearchIndex) يُعطل أيضًا

// ✅ الحل: تجاوز HasAudit فقط
trait HasAudit
{
    private static bool $auditEnabled = true;

    public static function withoutAudit(callable $callback): mixed
    {
        self::$auditEnabled = false;
        try {
            return $callback();
        } finally {
            self::$auditEnabled = true;
        }
    }

    public static function bootHasAudit(): void
    {
        static::creating(function (self $model): void {
            if (!self::$auditEnabled) return;
            // ...
        });
    }
}

// الاستخدام:
Customer::withoutAudit(function () use ($data) {
    return Customer::create($data); // HasAudit مُعطل، بقية Events تعمل
});
```

---

## 4️⃣ Telescope — منع التشغيل في Production

### ✅ ما هو ممتاز
- 3 طبقات دفاع — ✅ Defense in Depth
- `TELESCOPE_ENABLED=false` افتراضي — ✅ Secure by Default

### 🚨 ثغرات أمنية خطيرة

#### أ. الطبقة الثانية غير كافية — يجب حظر Route أيضًا

```php
// ❌ المشكلة: المستند يتجاهل حماية Route
// حتى لو register() عاد مبكرًا، Route قد تُسجّل في ServiceProvider آخر

// ✅ الحل الكامل:
class TelescopeServiceProvider extends TelescopeApplicationServiceProvider
{
    public function register(): void
    {
        if (! $this->shouldEnable()) {
            return;
        }

        parent::register();
    }

    public function boot(): void
    {
        if (! $this->shouldEnable()) {
            // ❌ حظر Route حتى لو سُجّلت
            $this->app['router']->get('telescope*', function () {
                abort(404);
            });
            return;
        }

        parent::boot();
    }

    private function shouldEnable(): bool
    {
        return $this->app->isLocal() 
            || $this->app->runningUnitTests()
            || ($this->app->environment('staging') && config('telescope.enabled_staging', false));
    }
}
```

#### ب. نقص: حماية البيانات الحساسة

```php
// ✅ أضف هذا في TelescopeServiceProvider::register()
Telescope::hideSensitiveRequestDetails();

Telescope::filter(function (IncomingEntry $entry) {
    // لا تسجّل كلمات المرور أبدًا
    if ($entry->type === 'request') {
        $content = $entry->content;
        unset($content['password'], $content['password_confirmation'], $content['credit_card']);
        $entry->content = $content;
    }

    return true;
});
```

#### ج. نقص: Rate Limiting للـ Telescope

```php
// ✅ في Production (حتى لو مُعطل، احمِ من الخطأ البشري)
// routes/telescope.php أو ServiceProvider

RateLimiter::for('telescope', function (Request $request) {
    return Limit::perMinute(10)->by($request->ip());
});
```

---

## 5️⃣ اقتراح إضافي: Module Boundaries — منع Circular Dependencies

> **ملاحظة:** هذا قسم إضافي غير موجود في المستند الأصلي لكنه ضروري لبنية modular سليمة.

### القاعدة: Module يستورد من Core فقط، لا من Module آخر

```
app/Modules/
├── Customers/          ← يستورد من: Core
├── Financing/          ← يستورد من: Core, Customers (❌ ممنوع!)
└── Invoicing/          ← يستورد من: Core, Customers, Financing (❌ ممنوع!)
```

### ✅ الحل: Shared Contracts في Core

```
app/Core/Contracts/
├── CustomerRepositoryInterface.php
├── FinancingCalculatorInterface.php
└── InvoiceGeneratorInterface.php
```

### ✅ الحل: Events للتواصل بين Modules

```php
// في Customers Module
event(new CustomerCreditLimitUpdated($customerId, $newLimit));

// في Financing Module (Listener)
class UpdateFinancingOnCreditChange
{
    public function handle(CustomerCreditLimitUpdated $event): void
    {
        // لا import مباشر من Customers Module
    }
}
```

---

## 📋 ملخص التحسينات المقترحة

| # | الملف | المشكلة | الخطورة | الحل |
|---|-------|---------|---------|------|
| 1 | `BaseModel` | `$guarded = []` خطر | 🔴 عالية | إجبار `$fillable` |
| 2 | `BaseRepository` | لا Transaction wrapper | 🟡 متوسطة | `transaction()` method |
| 3 | `HasAudit` | Queue يُنتج null silently | 🔴 عالية | `resolveAuditUser()` + Logging |
| 4 | `HasAudit` | لا `deleted_by` | 🟡 متوسطة | إضافة Soft Delete tracking |
| 5 | `TelescopeServiceProvider` | Route غير محمية | 🔴 عالية | حظر Route في `boot()` |
| 6 | `Telescope` | بيانات حساسة تُسجّل | 🔴 عالية | `hideSensitiveRequestDetails()` |
| 7 | `Architecture` | لا Module Boundaries | 🟡 متوسطة | Contracts + Events |

---

## 🏗️ التوصيات النهائية

### أولوية فورية (قبل Production)
1. إصلاح `BaseModel::$guarded` — خطر Mass Assignment
2. حماية Telescope Routes — ثغرة أمنية مباشرة
3. إضافة `resolveAuditUser()` — فقدان بيانات audit

### أولوية قصيرة (قبل الإصدار الأول)
4. إضافة Transaction wrapper للـ Repository
5. إضافة `deleted_by` للـ Soft Deletes
6. حماية البيانات الحساسة في Telescope

### أولوية متوسطة (تحسينات مستقبلية)
7. Module Boundaries مع Contracts
8. Action Pipeline للعمليات المعقدة
9. Generics للـ BaseRepository (PHP 8.3+)

---

*تم إعداد هذه المراجعة بواسطة مهندس معماري متخصص في Laravel & AI Agent Documentation.*
*المستند الأصلي: `core-system-missing-details.md`*
