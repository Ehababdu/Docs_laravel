# 🔧 Core System — التفاصيل المفقودة

> ملحق للوثيقة الرئيسية. يحل 4 غموضات محددة.

-----

## 1. قاعدة Actions vs Services — متى تستخدم أيهما

### الفرق الجوهري

|             |**Action**                        |**Service**                            |
|-------------|----------------------------------|---------------------------------------|
|**تعريف**    |عملية واحدة، غرض واحد             |مجموعة عمليات مترابطة لنفس الـ domain  |
|**الحجم**    |ملف صغير — method واحدة `handle()`|ملف أكبر — عدة methods                 |
|**الاستدعاء**|مرة واحدة لعملية محددة            |من أماكن متعددة                        |
|**التبعيات** |لا يستدعي Service آخر             |يستدعي Repositories وأحيانًا Actions    |
|**متى تنشئه**|عملية لها منطق خاص بها            |منطق مشترك يُستدعى من أكثر من Controller|

### القاعدة العملية

```
السؤال: هل هذا المنطق يُستدعى من أكثر من مكان؟
  ← نعم → Service
  ← لا، عملية واحدة محددة → Action

السؤال: هل العملية تحتاج إلى خطوات متعددة (validate + save + notify + log)؟
  ← نعم، ومتكررة → Service
  ← نعم، ولكن خاصة بهذا الطلب فقط → Action
```

### مثال واضح

```php
// ✅ Action — عملية واحدة، غرض واحد، لا تتكرر
// app/Modules/Customers/Actions/CreateCustomerAction.php
final class CreateCustomerAction
{
    public function __construct(
        private CustomerRepository $repository,
    ) {}

    public function handle(array $data): Customer
    {
        // منطق خاص بعملية الإنشاء فقط
        $data['credit_limit'] = $this->calculateInitialCreditLimit($data);

        $customer = $this->repository->create($data);

        event(new CustomerCreated($customer));

        return $customer;
    }

    private function calculateInitialCreditLimit(array $data): float
    {
        // منطق حساب الحد الائتماني الابتدائي
        return match($data['customer_type']) {
            'premium' => 5000.0,
            default   => 1000.0,
        };
    }
}
```

```php
// ✅ Service — منطق مشترك، يُستدعى من أكثر من مكان
// app/Modules/Financing/Services/FinancingCalculatorService.php
final class FinancingCalculatorService
{
    // يُستدعى من: FinancingController, ReportController, CustomerController
    public function calculateMonthlyInstallment(
        float $principal,
        float $annualRate,
        int   $months,
    ): float {
        if ($annualRate === 0.0) {
            return round($principal / $months, 3);
        }

        $monthlyRate = $annualRate / 12 / 100;
        return round(
            $principal * ($monthlyRate * pow(1 + $monthlyRate, $months))
                       / (pow(1 + $monthlyRate, $months) - 1),
            3
        );
    }

    // يُستدعى من: FinancingController, DashboardController
    public function calculateTotalProfit(float $principal, float $rate, int $months): float
    {
        $monthly = $this->calculateMonthlyInstallment($principal, $rate, $months);
        return round(($monthly * $months) - $principal, 3);
    }
}
```

```php
// ❌ خطأ — Action تستدعي Service آخر وتنمو
// هذا يعني أنك تحتاج Service وليس Action
class CreateCustomerAction
{
    public function handle(array $data): Customer
    {
        $this->notificationService->send(...);   // ❌
        $this->reportService->generate(...);     // ❌
        $this->auditService->log(...);           // ❌
        // إذا وصلت هنا → حوّل لـ Service
    }
}
```

### ملخص القرار

```
Action  → "افعل شيئًا واحدًا محددًا الآن"
Service → "أنا خبير في هذا الـ domain، استدعني من أي مكان"
```

-----

## 2. تنفيذ Base Classes الكاملة

### BaseModel

```php
<?php
// app/Core/Abstracts/BaseModel.php

namespace App\Core\Abstracts;

use Illuminate\Database\Eloquent\Model;

abstract class BaseModel extends Model
{
    // UUID كـ primary key — لا auto-increment
    public $incrementing = false;
    protected $keyType   = 'string';

    // منع Mass Assignment على مستوى التطبيق كله
    // كل model تحدد $fillable بنفسها صراحةً
    protected $guarded = [];

    // Cast افتراضي للتواريخ
    protected $casts = [
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
    ];
}
```

### BaseRepository

```php
<?php
// app/Core/Abstracts/BaseRepository.php

namespace App\Core\Abstracts;

use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Pagination\LengthAwarePaginator;

abstract class BaseRepository
{
    public function __construct(
        protected readonly Model $model,
    ) {}

    public function find(string $id): ?Model
    {
        return $this->model->find($id);
    }

    public function findOrFail(string $id): Model
    {
        return $this->model->findOrFail($id);
    }

    public function all(): Collection
    {
        return $this->model->all();
    }

    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return $this->model->latest()->paginate($perPage);
    }

    public function create(array $data): Model
    {
        return $this->model->create($data);
    }

    public function update(Model $model, array $data): bool
    {
        return $model->update($data);
    }

    public function delete(Model $model): bool
    {
        return $model->delete();
    }

    // للـ Repositories الأبناء — تضيف queries خاصة بـ domain
    // مثال في CustomerRepository:
    // public function findByNationalId(string $id): ?Customer { ... }
}
```

### BaseService

```php
<?php
// app/Core/Abstracts/BaseService.php

namespace App\Core\Abstracts;

// BaseService لا تحتوي منطقًا — هي عقد للـ DI فقط
// كل service تمتد منها وتحقن Repository خاصها

abstract class BaseService
{
    // intentionally empty
    // الأبناء يحقنون dependencies في constructor
}
```

> **ملاحظة:** BaseService فارغة عن قصد. لا تضع فيها منطقًا مشتركًا لأن كل Service لها طبيعة مختلفة. قيمتها الوحيدة هي التعرف السريع على الـ Services في الكود.

-----

## 3. HasAudit Trait — تنفيذ صحيح

### المشكلة التي يحلها

```php
// بدون HasAudit — تكتر هذا في كل Service
$data['created_by'] = auth()->id();   // ❌ منسي في نص الكود
$data['updated_by'] = auth()->id();   // ❌ قد يكون null بدون سبب واضح
```

### التنفيذ الصحيح

```php
<?php
// app/Core/Traits/HasAudit.php

namespace App\Core\Traits;

use Illuminate\Support\Facades\Auth;

trait HasAudit
{
    public static function bootHasAudit(): void
    {
        // عند الإنشاء — يسجل من أنشأ السجل
        static::creating(function (self $model): void {
            if (Auth::check()) {
                $model->created_by ??= Auth::id();
                $model->updated_by ??= Auth::id();
            }
        });

        // عند التعديل — يسجل من عدّل السجل
        static::updating(function (self $model): void {
            if (Auth::check()) {
                $model->updated_by = Auth::id();
            }
        });
    }

    // Helper methods للاستخدام في الكود
    public function creator(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(config('auth.providers.users.model'), 'created_by');
    }

    public function updater(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(config('auth.providers.users.model'), 'updated_by');
    }
}
```

### Migration — الأعمدة المطلوبة

```php
// أضف هذا في كل migration لجدول يستخدم HasAudit
$table->foreignUuid('created_by')->nullable()->constrained('users')->nullOnDelete();
$table->foreignUuid('updated_by')->nullable()->constrained('users')->nullOnDelete();
```

### استخدامه في Model

```php
class Customer extends BaseModel
{
    use HasUuid, HasAudit;

    // لا تضع created_by في $fillable
    // HasAudit يملأه تلقائيًا
    protected $fillable = [
        'name', 'phone', 'national_id', 'credit_limit', 'status',
        // ❌ لا: 'created_by', 'updated_by'
    ];
}
```

### حالة Queue / Commands

```php
// في Jobs أو Commands — لا يوجد auth() user
// الحل: تجاوز الـ trait يدويًا عند الحاجة

$customer = Customer::withoutEvents(function () use ($data) {
    // أو: أضف system_user_id في .env واستخدمه
    $data['created_by'] = config('system.system_user_id');
    return Customer::create($data);
});
```

### HasUuid — للاكتمال

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

-----

## 4. Telescope — منع التشغيل في Production

### المشكلة

`require-dev` يمنع تثبيت Telescope في Production عبر Composer.
لكن إذا نُسخ الكود بدون `--no-dev`، يجب خط دفاع ثانٍ.

### الحل الكامل — 3 طبقات

#### الطبقة الأولى: composer install صحيح

```bash
# في Production دائمًا
composer install --no-dev --optimize-autoloader

# هذا وحده يمنع تثبيت Telescope في Production
```

#### الطبقة الثانية: TelescopeServiceProvider

```php
<?php
// app/Providers/TelescopeServiceProvider.php
// هذا الملف موجود افتراضيًا — تحقق من محتواه

namespace App\Providers;

use Laravel\Telescope\TelescopeApplicationServiceProvider;

class TelescopeServiceProvider extends TelescopeApplicationServiceProvider
{
    public function register(): void
    {
        // ✅ يمنع تشغيل Telescope إذا لم يكن في require-dev
        if (! $this->app->isLocal() && ! $this->app->runningUnitTests()) {
            return;
        }

        parent::register();
    }
}
```

#### الطبقة الثالثة: .env + config

```bash
# .env.production
TELESCOPE_ENABLED=false

# .env.local
TELESCOPE_ENABLED=true
```

```php
// config/telescope.php
'enabled' => env('TELESCOPE_ENABLED', false),  // false افتراضي — الأكثر أمانًا
```

#### التحقق السريع

```bash
# تأكد أن Telescope لا يعمل في Production
php artisan telescope:clear  # يجب أن يفشل أو يعطي خطأ في Production
```

### جدول سريع — Dev vs Production

|الأداة       |Dev    |Production         |
|-------------|-------|-------------------|
|**Telescope**|✅ شغّال |❌ مُعطّل بـ 3 طبقات  |
|**Debugbar** |✅ شغّال |❌ `require-dev` فقط|
|**Sentry**   |اختياري|✅ مطلوب            |
|**Horizon**  |✅      |✅                  |
|**Pulse**    |✅      |✅                  |

-----

*هذا الملف ملحق — يُقرأ مع الوثيقة الرئيسية `core-system-laravel-architecture.md`*