# 🏗️ Core System — Laravel 13 | هيكلة موديولار احترافية

> وثيقة هندسية عملية. لا نظريات. كل قرار له سبب.

-----

## 📋 جدول المحتويات

1. [المبدأ الأساسي](#-المبدأ-الأساسي)
1. [هيكلة المجلدات](#-هيكلة-المجلدات)
1. [الـ Module الواحد — بنيته الداخلية](#-بنية-الموديول-الواحد)
1. [الأدوات المختارة — فقط ما يُحتاج](#-الأدوات-المختارة)
1. [طبقات التطبيق](#-طبقات-التطبيق)
1. [قاعدة البيانات والعلاقات](#-قاعدة-البيانات)
1. [الأمان](#-الأمان)
1. [الاختبارات](#-الاختبارات)
1. [النشر](#-النشر)
1. [قواعد لا تُكسر](#-قواعد-لا-تُكسر)

-----

## 🎯 المبدأ الأساسي

النظام مبني على **Modular Monolith** — ليس Microservices ولا Laravel تقليدي.

```
كل موديول = وحدة مستقلة لها:
  - Routes خاصة
  - Models خاصة
  - Services خاصة
  - Tests خاصة
```

**لماذا Modular Monolith وليس Microservices؟**

- فريق صغير → Microservices overhead غير مبرر
- قاعدة بيانات واحدة → لا حاجة لـ distributed transactions
- قابل للتحول لـ Microservices لاحقًا إذا احتجت

-----

## 📁 هيكلة المجلدات

```
app/
├── Core/                          # الكود المشترك بين كل الموديولات
│   ├── Abstracts/
│   │   ├── BaseRepository.php
│   │   ├── BaseService.php
│   │   └── BaseModel.php
│   ├── Traits/
│   │   ├── HasUuid.php
│   │   ├── HasAudit.php
│   │   └── Filterable.php
│   ├── Exceptions/
│   │   ├── Handler.php
│   │   └── BusinessException.php
│   ├── Http/
│   │   ├── Middleware/
│   │   └── Resources/
│   │       └── BaseResource.php
│   └── Helpers/
│       └── helpers.php
│
├── Modules/
│   ├── Auth/
│   ├── Users/
│   ├── Customers/
│   ├── Financing/
│   ├── Products/
│   ├── Inventory/
│   ├── Orders/
│   ├── Payments/
│   ├── Reports/
│   ├── Notifications/
│   ├── Documents/
│   ├── Settings/
│   └── AuditLog/
│
bootstrap/
config/
database/
│   ├── migrations/               # كل migrations في مكان واحد — أسهل للإدارة
│   ├── seeders/
│   └── factories/
resources/
routes/
│   ├── api.php                   # يستورد فقط من كل موديول
│   └── web.php
tests/
│   ├── Unit/
│   ├── Feature/
│   └── Modules/                  # tests مرتبة بنفس ترتيب الموديولات
```

-----

## 🧩 بنية الموديول الواحد

**مثال: موديول Customers**

```
app/Modules/Customers/
├── Controllers/
│   ├── CustomerController.php         # CRUD عادي
│   └── CustomerFinancingController.php
├── Models/
│   └── Customer.php
├── Services/
│   └── CustomerService.php            # كل Business Logic هنا فقط
├── Repositories/
│   └── CustomerRepository.php         # كل DB queries هنا فقط
├── Requests/
│   ├── StoreCustomerRequest.php
│   └── UpdateCustomerRequest.php
├── Resources/
│   ├── CustomerResource.php
│   └── CustomerCollection.php
├── Actions/                           # عمليات محددة وحيدة الغرض
│   ├── CreateCustomerAction.php
│   ├── UpdateCustomerStatusAction.php
│   └── CalculateCreditLimitAction.php
├── Events/
│   └── CustomerCreated.php
├── Listeners/
│   └── SendWelcomeNotification.php
├── Policies/
│   └── CustomerPolicy.php
├── routes/
│   └── customers.php
└── Tests/
    ├── CustomerServiceTest.php
    └── CustomerApiTest.php
```

-----

## 🔧 الأدوات المختارة

### ✅ ما يُستخدم فعلًا — مع السبب

#### الأساس

|الأداة                   |السبب                                |
|-------------------------|-------------------------------------|
|**Laravel 13**           |أحدث إصدار مستقر — PHP 8.3+          |
|**Inertia.js v2**        |SPA بدون API مستقل — مثالي مع Laravel|
|**React 19 + TypeScript**|واجهة أمامية قوية ومنظمة             |
|**Vite**                 |Build tool سريع — مدمج مع Laravel    |

#### قاعدة البيانات

|الأداة     |السبب                                  |
|-----------|---------------------------------------|
|**MySQL 8**|الأكثر استخدامًا مع Laravel — دعم ممتاز |
|**Redis**  |Sessions + Cache + Queues في أداة واحدة|

#### المصادقة والصلاحيات

|الأداة                       |السبب                                     |
|-----------------------------|------------------------------------------|
|**Laravel Sanctum**          |مصادقة SPA بالكوكيز — الأبسط والأكثر أمانًا|
|**Spatie Laravel Permission**|الأكثر نضجًا لإدارة Roles/Permissions      |

#### جودة الكود

|الأداة                 |السبب                                        |
|-----------------------|---------------------------------------------|
|**Laravel Pint**       |تنسيق الكود — مدمج مع Laravel، صفر إعداد     |
|**Larastan (Level 6+)**|تحليل ثابت — يمنع أخطاء كثيرة قبل الـ Runtime|
|**Pest**               |أفضل من PHPUnit في القراءة والكتابة          |

#### المراقبة

|الأداة               |الاستخدام                                       |
|---------------------|------------------------------------------------|
|**Laravel Telescope**|في بيئة Development فقط — لا تثبّته في Production|
|**Sentry**           |في بيئة Production فقط — تتبع الأخطاء الحقيقية  |

#### السجلات والتدقيق

|الأداة                        |السبب                                     |
|------------------------------|------------------------------------------|
|**Spatie Laravel Activitylog**|تسجيل من فعل ماذا — ضروري في أنظمة التمويل|

#### الأداء

|الأداة             |السبب                                        |
|-------------------|---------------------------------------------|
|**Laravel Horizon**|إدارة Queues — ضروري لمعالجة العمليات الثقيلة|

-----

### ❌ ما لا يُستخدم — مع السبب

|الأداة                   |السبب                                                |
|-------------------------|-----------------------------------------------------|
|**Laravel Octane**       |يحتاج Swoole أو RoadRunner — تعقيد غير مبرر الآن     |
|**Meilisearch**          |MySQL Full-Text Search يكفي في هذه المرحلة           |
|**Laravel Scout**        |لا حاجة له بدون محرك بحث خارجي                       |
|**Memcached**            |Redis يغني عنه تمامًا                                 |
|**Rubix ML**             |للـ AI استخدم Python Microservice — لا تخلط PHP بـ ML|
|**Laravel Forge/Envoyer**|GitHub Actions + VPS مباشرة أبسط وأرخص               |
|**L5 Swagger**           |Postman Collections أسرع وأعملي في الفريق الصغير     |
|**PHP Insights**         |Larastan + Pint يغطيان 90% من احتياجات جودة الكود    |

-----

## 🏛️ طبقات التطبيق

```
HTTP Request
    ↓
[ Route ] → [ Middleware ]
    ↓
[ FormRequest ]     ← Validation هنا فقط
    ↓
[ Controller ]      ← يستقبل الطلب ويردّ فقط — لا Business Logic
    ↓
[ Action / Service ]← كل Business Logic هنا
    ↓
[ Repository ]      ← كل DB Queries هنا فقط
    ↓
[ Model ]           ← Eloquent + Relationships + Scopes
    ↓
[ Database ]
```

**قاعدة صارمة:**

- Controller لا يستدعي Model مباشرة أبدًا
- Service لا تكتب SQL مباشرة أبدًا
- Repository لا يحتوي Business Logic أبدًا

-----

## 🗄️ قاعدة البيانات

### Migrations — قواعد

```php
// ✅ صح — اسم واضح ومحدد
2025_01_15_create_customers_table.php
2025_01_16_add_credit_limit_to_customers_table.php

// ❌ غلط
2025_01_15_update_table.php
```

### Model — البنية القياسية

```php
<?php

namespace App\Modules\Customers\Models;

use App\Core\Abstracts\BaseModel;
use App\Core\Traits\HasUuid;
use App\Core\Traits\HasAudit;

class Customer extends BaseModel
{
    use HasUuid, HasAudit;

    protected $fillable = [
        'name', 'phone', 'national_id', 'credit_limit', 'status',
    ];

    protected $casts = [
        'credit_limit' => 'decimal:3',
        'status'       => CustomerStatus::class,  // PHP Enum
        'created_at'   => 'datetime',
    ];

    // Relationships
    public function financings(): HasMany
    {
        return $this->hasMany(Financing::class);
    }

    // Scopes
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('status', CustomerStatus::Active);
    }
}
```

### Indexing — لا تنساه

```php
// في كل migration — ضع indexes على:
$table->index('status');
$table->index('created_at');
$table->index(['customer_id', 'status']);   // Composite index
$table->unique('national_id');
```

-----

## 🔒 الأمان

### طبقات الأمان بالترتيب

```
1. HTTPS — إجباري في الإنتاج
2. Sanctum HttpOnly Cookies — منع XSS token theft
3. Rate Limiting — على كل endpoints الحساسة
4. FormRequest Validation — قبل أي معالجة
5. Policies — للتحقق من صلاحية الوصول للموارد
6. Laravel Crypt — للبيانات الحساسة في DB
```

### Rate Limiting — إعداد واقعي

```php
// في RouteServiceProvider
RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('auth', function (Request $request) {
    return Limit::perMinute(5)->by($request->ip());  // صارم على تسجيل الدخول
});
```

-----

## 🧪 الاختبارات

### الأولوية في الاختبارات

```
1. Feature Tests   → أهم شيء — اختبر الـ API endpoints كاملة
2. Unit Tests      → للـ Services والـ Actions المعقدة فقط
3. لا تختبر Eloquent نفسه — هذا مسؤولية Laravel
```

### مثال Pest Test واقعي

```php
it('can create a customer with valid data', function () {
    $data = Customer::factory()->make()->toArray();

    $response = $this->actingAs($this->adminUser)
        ->postJson('/api/customers', $data);

    $response->assertCreated()
        ->assertJsonStructure(['data' => ['id', 'name', 'phone']]);

    $this->assertDatabaseHas('customers', ['phone' => $data['phone']]);
});

it('rejects duplicate national_id', function () {
    Customer::factory()->create(['national_id' => '123456789']);

    $response = $this->actingAs($this->adminUser)
        ->postJson('/api/customers', ['national_id' => '123456789', ...]);

    $response->assertUnprocessable();
});
```

-----

## 🚀 النشر

### Stack النشر الموصى

```
Ubuntu 24.04 LTS
├── Nginx
├── PHP 8.3 FPM
├── MySQL 8
├── Redis 7
└── Supervisor (لـ Queue Workers)
```

### GitHub Actions — Pipeline بسيط وفعّال

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Run Tests
        run: php artisan test --parallel

      - name: Deploy to Server
        run: |
          ssh user@server "
            cd /var/www/core-system &&
            git pull origin main &&
            composer install --no-dev --optimize-autoloader &&
            php artisan migrate --force &&
            php artisan config:cache &&
            php artisan route:cache &&
            php artisan view:cache &&
            php artisan queue:restart &&
            sudo systemctl reload php8.3-fpm
          "
```

-----

## 📐 قواعد لا تُكسر

```
1. لا Business Logic في Controllers
2. لا DB Queries في Controllers أو Services مباشرة
3. لا Raw SQL — استخدم Query Builder أو Eloquent دائمًا
4. كل endpoint لها FormRequest خاص بها
5. كل موديول له Policy خاصة به
6. Telescope في Dev فقط — Sentry في Production فقط
7. لا تضيف أداة إلا إذا حللت مشكلة حقيقية الآن
8. Redis للكل: Sessions + Cache + Queues
9. كل migration لها rollback (down method) قابل للتنفيذ
10. لا Soft Deletes إلا إذا كان هناك متطلب صريح لها
```

-----

## 📦 composer.json — الـ Dependencies النهائية

```json
{
  "require": {
    "php": "^8.3",
    "laravel/framework": "^13.0",
    "laravel/sanctum": "^4.0",
    "spatie/laravel-permission": "^6.0",
    "spatie/laravel-activitylog": "^4.0",
    "laravel/horizon": "^5.0"
  },
  "require-dev": {
    "pestphp/pest": "^3.0",
    "pestphp/pest-plugin-laravel": "^3.0",
    "larastan/larastan": "^3.0",
    "laravel/pint": "^1.0",
    "laravel/telescope": "^5.0",
    "fakerphp/faker": "^1.23"
  }
}
```

-----

## 📦 package.json — Frontend Dependencies

```json
{
  "dependencies": {
    "react": "^19.0",
    "react-dom": "^19.0",
    "@inertiajs/react": "^2.0",
    "react-hook-form": "^7.0",
    "zod": "^3.0",
    "@hookform/resolvers": "^3.0",
    "recharts": "^2.0",
    "react-toastify": "^10.0",
    "sweetalert2": "^11.0"
  },
  "devDependencies": {
    "typescript": "^5.0",
    "vite": "^6.0",
    "@vitejs/plugin-react": "^4.0",
    "tailwindcss": "^4.0"
  }
}
```

-----

*الهيكلة صُممت لفريق 1–3 مطورين، نظام إنتاجي حقيقي، قابل للتوسع.*
*آخر تحديث: يونيو 2026*