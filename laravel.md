# 📚 توثيق Laravel 13.x — دليل شامل بالعربية

> **المصدر الرسمي:** [laravel.com/docs/13.x](https://laravel.com/docs/13.x)  
> **الإصدار:** Laravel 13.x | PHP 8.5+  
> **آخر تحديث:** يونيو 2026

---

## 📋 فهرس المحتويات

1. [مقدمة عن Laravel](#1-مقدمة-عن-laravel)
2. [التثبيت والإعداد](#2-التثبيت-والإعداد)
3. [هيكل المشروع](#3-هيكل-المشروع)
4. [دورة حياة الطلب](#4-دورة-حياة-الطلب)
5. [مفاهيم المعمارية](#5-مفاهيم-المعمارية)
6. [الأساسيات — Routing, Middleware, Controllers](#6-الأساسيات)
7. [العروض — Views & Blade](#7-العروض--views--blade)
8. [قاعدة البيانات والـ Eloquent ORM](#8-قاعدة-البيانات-والـ-eloquent-orm)
9. [الأمان والمصادقة](#9-الأمان-والمصادقة)
10. [الذكاء الاصطناعي — Laravel AI SDK](#10-الذكاء-الاصطناعي--laravel-ai-sdk)
11. [الطوابير والجدولة والأحداث](#11-الطوابير-والجدولة-والأحداث)
12. [الاختبار — Testing](#12-الاختبار--testing)
13. [الحزم الرسمية](#13-الحزم-الرسمية)
14. [النشر والإنتاج](#14-النشر-والإنتاج)
15. [أوامر Artisan المرجعية](#15-أوامر-artisan-المرجعية)

---

## 1. مقدمة عن Laravel

Laravel هو إطار عمل PHP تعبيري وأنيق لبناء تطبيقات الويب الحديثة. يُقدّم للمطور بنية جاهزة تُتيح له التركيز على المنطق التجاري دون الانشغال بالتفاصيل التقنية الدقيقة.

### لماذا Laravel؟

| الميزة | الوصف |
|--------|-------|
| **إطار تطوري** | يناسب المبتدئين والمحترفين على حدٍّ سواء |
| **إطار قابل للتوسع** | يدعم التوسع الأفقي عبر Redis والخوادم الموزعة |
| **إطار AI-Ready** | بنيته المنظمة تجعله مثالياً للتطوير بمساعدة الذكاء الاصطناعي |
| **مجتمع نشط** | آلاف المساهمين وتوثيق شامل |

### طرق استخدام Laravel

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│   Full Stack Framework          API Backend          │
│   ──────────────────           ────────────          │
│   Blade Templates         →    REST / GraphQL        │
│   Inertia.js (React/Vue)  →    Next.js / Flutter     │
│   Livewire                →    Mobile Apps           │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 2. التثبيت والإعداد

### المتطلبات

- PHP >= 8.2
- Composer 2.x
- Node.js / Bun (للـ frontend assets)

### التثبيت السريع

**macOS:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.5)"
```

**Windows (PowerShell كمسؤول):**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.5'))
```

**Linux:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.5)"
```

**تثبيت Laravel Installer عبر Composer:**
```bash
composer global require laravel/installer
```

### إنشاء مشروع جديد

```bash
laravel new example-app
```

سيطلب منك المُثبِّت اختيار:
- إطار عمل الاختبار (Pest / PHPUnit)
- قاعدة البيانات (SQLite / MySQL / PostgreSQL)
- Starter Kit (None / Breeze / Jetstream)

### تشغيل بيئة التطوير

```bash
cd example-app
npm install && npm run build
composer run dev
```

يُشغّل هذا الأمر دفعةً واحدة:
- خادم Laravel المحلي على `http://localhost:8000`
- Queue Worker
- Vite Development Server

### إعداد قاعدة البيانات

في ملف `.env`:

```ini
# MySQL
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

# PostgreSQL
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=postgres
DB_PASSWORD=
```

```bash
php artisan migrate
```

### Laravel Herd (بيئة تطوير مرئية)

[Laravel Herd](https://herd.laravel.com) هو بيئة تطوير محلية مدمجة تحتوي على PHP و Nginx جاهزة للاستخدام في macOS و Windows.

```bash
cd ~/Herd
laravel new my-app
cd my-app
herd open
# التطبيق متاح على: http://my-app.test
```

---

## 3. هيكل المشروع

```
my-app/
├── app/
│   ├── Ai/              # AI Agents والـ Tools
│   ├── Console/         # Artisan Commands
│   ├── Exceptions/      # معالجة الأخطاء
│   ├── Http/
│   │   ├── Controllers/ # المتحكمات
│   │   ├── Middleware/  # الـ Middleware
│   │   └── Requests/    # Form Requests
│   ├── Models/          # Eloquent Models
│   ├── Providers/       # Service Providers
│   └── Services/        # Business Logic
├── bootstrap/           # إقلاع التطبيق
├── config/              # ملفات الإعداد
├── database/
│   ├── factories/       # مصانع البيانات
│   ├── migrations/      # الترحيلات
│   └── seeders/         # بذور البيانات
├── public/              # نقطة دخول الطلبات
├── resources/
│   ├── css/
│   ├── js/
│   └── views/           # Blade Templates
├── routes/
│   ├── web.php          # مسارات الويب
│   ├── api.php          # مسارات API
│   └── console.php      # مسارات Artisan
├── storage/             # الملفات المُولَّدة
├── tests/               # الاختبارات
├── .env                 # متغيرات البيئة
└── composer.json
```

---

## 4. دورة حياة الطلب

```
HTTP Request
     │
     ▼
public/index.php          ← نقطة الدخول الوحيدة
     │
     ▼
bootstrap/app.php         ← إنشاء Application Instance (Service Container)
     │
     ▼
HTTP Kernel               ← التهيئة + Middleware Stack
     │
     ▼
Service Providers         ← تسجيل الخدمات وربطها
     │
     ▼
Router                    ← تطابق الـ Route
     │
     ▼
Middleware (Route)        ← فحص المصادقة، CSRF...
     │
     ▼
Controller / Closure      ← تنفيذ المنطق
     │
     ▼
Response                  ← إرجاع الاستجابة للمتصفح
```

### Service Providers

أهم مرحلة في دورة الحياة. يمر كل Service Provider بمرحلتين:

```php
// register() — تسجيل الخدمات في الـ Container
public function register(): void
{
    $this->app->bind(PaymentGateway::class, StripeGateway::class);
}

// boot() — يُنفَّذ بعد تسجيل جميع الخدمات
public function boot(): void
{
    View::composer('*', function ($view) {
        $view->with('user', Auth::user());
    });
}
```

---

## 5. مفاهيم المعمارية

### Service Container

قلب Laravel — حاوية تعتمد على Dependency Injection:

```php
// ربط بسيط
$this->app->bind(MailerInterface::class, SmtpMailer::class);

// Singleton — مثيل واحد طوال فترة التطبيق
$this->app->singleton(Cache::class, RedisCache::class);

// حقن تلقائي في المتحكمات
class OrderController extends Controller
{
    public function __construct(
        private readonly PaymentGateway $payment,
        private readonly OrderRepository $orders,
    ) {}
}
```

### Facades

واجهات ثابتة توفر وصولاً سهلاً لخدمات الـ Container:

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Queue;
use Illuminate\Support\Facades\Storage;

Cache::put('key', 'value', 3600);
$users = DB::table('users')->get();
Log::info('User logged in', ['id' => $user->id]);
Storage::disk('s3')->put('file.txt', $content);
```

### Contracts

واجهات (Interfaces) تُحدِّد عقوداً بين مكونات التطبيق:

```php
use Illuminate\Contracts\Cache\Repository as CacheContract;

class ReportGenerator
{
    public function __construct(private CacheContract $cache) {}
}
```

---

## 6. الأساسيات

### Routing

```php
// routes/web.php

// مسارات أساسية
Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
Route::put('/users/{user}', [UserController::class, 'update']);
Route::delete('/users/{user}', [UserController::class, 'destroy']);

// مسار Resource كامل (7 مسارات دفعة واحدة)
Route::resource('products', ProductController::class);

// مسارات API Resource (بدون create/edit)
Route::apiResource('orders', OrderController::class);

// تجميع المسارات
Route::middleware(['auth', 'verified'])->prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard']);
    Route::resource('users', AdminUserController::class);
});

// Route Model Binding — تحميل النموذج تلقائياً
Route::get('/users/{user}', function (User $user) {
    return $user;
});

// مسارات باسم
Route::get('/profile', [ProfileController::class, 'show'])->name('profile.show');
$url = route('profile.show');
```

### Middleware

```php
// إنشاء Middleware
php artisan make:middleware CheckAge

// app/Http/Middleware/CheckAge.php
public function handle(Request $request, Closure $next): Response
{
    if ($request->user()->age < 18) {
        return redirect('/home');
    }
    return $next($request);
}

// تسجيل في bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'check.age' => CheckAge::class,
    ]);
})

// استخدام في المسارات
Route::get('/adults-only', handler())->middleware('check.age');
```

### Controllers

```php
// إنشاء متحكم
php artisan make:controller ProductController --resource

// app/Http/Controllers/ProductController.php
class ProductController extends Controller
{
    public function index(): View
    {
        $products = Product::paginate(15);
        return view('products.index', compact('products'));
    }

    public function store(StoreProductRequest $request): RedirectResponse
    {
        $product = Product::create($request->validated());
        return redirect()->route('products.show', $product)
                         ->with('success', 'تم إنشاء المنتج بنجاح');
    }

    public function show(Product $product): View
    {
        return view('products.show', compact('product'));
    }

    public function update(UpdateProductRequest $request, Product $product): RedirectResponse
    {
        $product->update($request->validated());
        return back()->with('success', 'تم التحديث بنجاح');
    }

    public function destroy(Product $product): RedirectResponse
    {
        $product->delete();
        return redirect()->route('products.index');
    }
}
```

### Form Requests (التحقق من البيانات)

```php
// إنشاء Form Request
php artisan make:request StoreProductRequest

// app/Http/Requests/StoreProductRequest.php
class StoreProductRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Product::class);
    }

    public function rules(): array
    {
        return [
            'name'        => ['required', 'string', 'max:255'],
            'price'       => ['required', 'numeric', 'min:0'],
            'description' => ['nullable', 'string'],
            'category_id' => ['required', 'exists:categories,id'],
            'image'       => ['nullable', 'image', 'max:2048'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required'  => 'اسم المنتج مطلوب',
            'price.required' => 'السعر مطلوب',
        ];
    }
}
```

---

## 7. العروض — Views & Blade

### Blade Templates الأساسية

```blade
{{-- resources/views/products/index.blade.php --}}

@extends('layouts.app')

@section('title', 'المنتجات')

@section('content')
    <div class="container">
        {{-- طباعة متغير (مع escape تلقائي) --}}
        <h1>{{ $title }}</h1>

        {{-- طباعة HTML خام (بدون escape) --}}
        {!! $htmlContent !!}

        {{-- شروط --}}
        @if ($products->isEmpty())
            <p>لا توجد منتجات</p>
        @elseif ($products->count() < 5)
            <p>عدد قليل من المنتجات</p>
        @else
            <p>{{ $products->count() }} منتج</p>
        @endif

        {{-- حلقات --}}
        @foreach ($products as $product)
            <div class="product-card">
                <h3>{{ $product->name }}</h3>
                <p>{{ number_format($product->price, 2) }} د.ل</p>

                @forelse ($product->images as $image)
                    <img src="{{ $image->url }}" alt="{{ $product->name }}">
                @empty
                    <img src="/default.jpg" alt="لا توجد صورة">
                @endforelse
            </div>
        @endforeach

        {{-- Pagination --}}
        {{ $products->links() }}
    </div>
@endsection
```

### Blade Components

```bash
php artisan make:component Alert
```

```blade
{{-- resources/views/components/alert.blade.php --}}
<div class="alert alert-{{ $type }} {{ $attributes->get('class') }}">
    {{ $slot }}
</div>
```

```php
// app/View/Components/Alert.php
class Alert extends Component
{
    public function __construct(
        public string $type = 'info'
    ) {}

    public function render(): View
    {
        return view('components.alert');
    }
}
```

```blade
{{-- الاستخدام --}}
<x-alert type="success" class="mt-4">
    تم حفظ البيانات بنجاح!
</x-alert>
```

### Asset Bundling (Vite)

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

```blade
{{-- في الـ layout --}}
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

---

## 8. قاعدة البيانات والـ Eloquent ORM

### Migrations

```php
// إنشاء Migration
php artisan make:migration create_products_table

// database/migrations/xxxx_create_products_table.php
public function up(): void
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description')->nullable();
        $table->decimal('price', 10, 2);
        $table->integer('stock')->default(0);
        $table->boolean('is_active')->default(true);
        $table->foreignId('category_id')->constrained()->cascadeOnDelete();
        $table->string('image')->nullable();
        $table->softDeletes();  // حذف ناعم
        $table->timestamps();   // created_at & updated_at
    });
}

public function down(): void
{
    Schema::dropIfExists('products');
}
```

```bash
php artisan migrate             # تطبيق الترحيلات
php artisan migrate:rollback    # التراجع عن آخر batch
php artisan migrate:fresh       # حذف كل شيء وإعادة التطبيق
php artisan migrate:status      # عرض حالة الترحيلات
```

### Eloquent Models

```php
// app/Models/Product.php
#[Table('products')]
class Product extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'name', 'description', 'price', 'stock', 'is_active', 'category_id'
    ];

    protected $casts = [
        'price'     => 'decimal:2',
        'is_active' => 'boolean',
        'deleted_at'=> 'datetime',
    ];

    // علاقات
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function images(): HasMany
    {
        return $this->hasMany(ProductImage::class);
    }

    public function orders(): BelongsToMany
    {
        return $this->belongsToMany(Order::class, 'order_items')
                    ->withPivot('quantity', 'price')
                    ->withTimestamps();
    }

    // Scope
    public function scopeActive(Builder $query): void
    {
        $query->where('is_active', true);
    }

    // Accessor
    public function getFormattedPriceAttribute(): string
    {
        return number_format($this->price, 2) . ' د.ل';
    }
}
```

### استعلامات Eloquent

```php
// ─── قراءة ───────────────────────────────────────────
Product::all();                                  // كل السجلات
Product::find(1);                                // بالـ ID
Product::findOrFail(1);                          // أو رمي استثناء
Product::where('is_active', true)->get();
Product::active()->orderBy('price')->paginate(15);
Product::with('category', 'images')->get();      // Eager Loading

// ─── إنشاء ───────────────────────────────────────────
Product::create([
    'name'        => 'قميص قطني',
    'price'       => 89.99,
    'category_id' => 2,
]);

// ─── تحديث ───────────────────────────────────────────
$product = Product::findOrFail(1);
$product->update(['price' => 99.99]);

// أو مباشرة
Product::where('category_id', 3)->update(['is_active' => false]);

// ─── حذف ─────────────────────────────────────────────
$product->delete();                  // حذف ناعم (مع SoftDeletes)
$product->forceDelete();             // حذف دائم
Product::withTrashed()->get();       // استعادة المحذوفات
$product->restore();                 // استعادة سجل

// ─── Upsert ──────────────────────────────────────────
Product::upsert(
    [
        ['sku' => 'PRD-001', 'price' => 99.99, 'name' => 'منتج 1'],
        ['sku' => 'PRD-002', 'price' => 149.99, 'name' => 'منتج 2'],
    ],
    uniqueBy: ['sku'],
    update: ['price']
);
```

### Query Builder

```php
DB::table('products')
    ->select('name', 'price', DB::raw('COUNT(*) as count'))
    ->join('categories', 'products.category_id', '=', 'categories.id')
    ->where('products.is_active', true)
    ->whereBetween('price', [50, 200])
    ->groupBy('category_id')
    ->having('count', '>', 5)
    ->orderByDesc('price')
    ->limit(10)
    ->get();
```

### Seeders & Factories

```php
// database/factories/ProductFactory.php
class ProductFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name'        => $this->faker->words(3, true),
            'description' => $this->faker->paragraph(),
            'price'       => $this->faker->randomFloat(2, 10, 500),
            'stock'       => $this->faker->numberBetween(0, 100),
            'is_active'   => true,
            'category_id' => Category::factory(),
        ];
    }
}

// database/seeders/DatabaseSeeder.php
public function run(): void
{
    Category::factory(10)->create();
    Product::factory(100)->create();
}
```

```bash
php artisan db:seed
php artisan db:seed --class=ProductSeeder
```

---

## 9. الأمان والمصادقة

### Authentication (Sanctum للـ API)

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

```php
// routes/api.php
Route::post('/login', [AuthController::class, 'login']);
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', fn(Request $r) => $r->user());
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::apiResource('products', ProductController::class);
});

// AuthController.php
public function login(Request $request): JsonResponse
{
    $credentials = $request->validate([
        'email'    => ['required', 'email'],
        'password' => ['required'],
    ]);

    if (! Auth::attempt($credentials)) {
        return response()->json(['message' => 'بيانات غير صحيحة'], 401);
    }

    $token = $request->user()->createToken('api-token')->plainTextToken;
    return response()->json(['token' => $token]);
}

public function logout(Request $request): JsonResponse
{
    $request->user()->currentAccessToken()->delete();
    return response()->json(['message' => 'تم تسجيل الخروج']);
}
```

### Authorization (Policies)

```bash
php artisan make:policy ProductPolicy --model=Product
```

```php
// app/Policies/ProductPolicy.php
class ProductPolicy
{
    public function viewAny(User $user): bool
    {
        return true;
    }

    public function update(User $user, Product $product): bool
    {
        return $user->id === $product->user_id || $user->hasRole('admin');
    }

    public function delete(User $user, Product $product): bool
    {
        return $user->hasRole('admin');
    }
}

// الاستخدام في المتحكم
public function update(UpdateProductRequest $request, Product $product)
{
    $this->authorize('update', $product);  // يرمي 403 إذا غير مصرح
    $product->update($request->validated());
}
```

### Encryption & Hashing

```php
// تشفير القيم
$encrypted = Crypt::encrypt('بيانات سرية');
$decrypted = Crypt::decrypt($encrypted);

// تجزئة كلمات المرور
$hashed = Hash::make('password123');
Hash::check('password123', $hashed);  // true
```

---

## 10. الذكاء الاصطناعي — Laravel AI SDK

Laravel 13.x يُقدّم **Laravel AI SDK** — واجهة موحدة للتفاعل مع موفري الذكاء الاصطناعي.

### التثبيت

```bash
composer require laravel/ai
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
php artisan migrate
```

### إعداد متغيرات البيئة

```ini
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=AIza...
GROQ_API_KEY=gsk_...
```

### الموفرون المدعومون

| الميزة | الموفرون |
|--------|----------|
| **نص (Chat)** | OpenAI, Anthropic, Gemini, Azure, Groq, DeepSeek, Mistral, Ollama |
| **صور** | OpenAI, Gemini, xAI, Azure |
| **تحويل نص لصوت (TTS)** | OpenAI, ElevenLabs, Gemini |
| **تحويل صوت لنص (STT)** | OpenAI, ElevenLabs, Mistral, Gemini |
| **Embeddings** | OpenAI, Gemini, Cohere, Jina, VoyageAI, Ollama |
| **Reranking** | Cohere, Jina, VoyageAI |

### إنشاء AI Agent

```bash
php artisan make:agent CustomerSupportAgent
php artisan make:agent AnalysisAgent --structured  # مع Structured Output
```

```php
// app/Ai/Agents/CustomerSupportAgent.php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\SearchKnowledgeBase;
use Laravel\Ai\Agent;

class CustomerSupportAgent extends Agent
{
    /**
     * التعليمات الأساسية للوكيل
     */
    protected function instructions(): string
    {
        return <<<PROMPT
        أنت مساعد دعم عملاء متخصص في متجرنا.
        أجب بشكل ودود ومهني باللغة العربية.
        استخدم قاعدة المعرفة المتاحة للإجابة على الأسئلة.
        إذا لم تجد إجابة، اعتذر وأحل المستخدم لفريق الدعم البشري.
        PROMPT;
    }

    /**
     * الأدوات المتاحة للوكيل
     */
    protected function tools(): array
    {
        return [
            new SearchKnowledgeBase(),
        ];
    }
}
```

### استخدام الوكيل

```php
use App\Ai\Agents\CustomerSupportAgent;

// استخدام مباشر
$response = CustomerSupportAgent::prompt('ما هي سياسة الاسترجاع؟');
echo $response->text();

// مع سياق محادثة (Conversation)
$agent = app(CustomerSupportAgent::class);
$conversation = $agent->continue($conversationId);
$response = $conversation->prompt('كيف أتتبع طلبي؟');

// Streaming
CustomerSupportAgent::prompt('اشرح خطوات الطلب')
    ->stream(function (string $chunk) {
        echo $chunk;
    });
```

### Structured Output

```php
// make:agent --structured
class ProductAnalysisAgent extends Agent
{
    protected function instructions(): string
    {
        return 'حلل المنتج وأرجع بيانات منظمة.';
    }

    protected function schema(): JsonSchema
    {
        return JsonSchema::object([
            'sentiment'   => JsonSchema::string()->enum(['positive', 'negative', 'neutral']),
            'score'       => JsonSchema::number()->minimum(0)->maximum(10),
            'keywords'    => JsonSchema::array(JsonSchema::string()),
            'summary'     => JsonSchema::string(),
        ]);
    }
}

// الاستخدام
$result = ProductAnalysisAgent::prompt('تقييم: منتج ممتاز جداً')->structured();
echo $result->sentiment;  // positive
echo $result->score;      // 9.5
```

### إنشاء Tool مخصص

```php
// app/Ai/Tools/SearchProducts.php
use Laravel\Ai\Tool;

class SearchProducts extends Tool
{
    protected string $name = 'search_products';
    protected string $description = 'البحث عن المنتجات في قاعدة البيانات';

    protected function parameters(): array
    {
        return [
            'query'    => ['type' => 'string', 'description' => 'مصطلح البحث'],
            'category' => ['type' => 'string', 'description' => 'الفئة (اختياري)'],
        ];
    }

    public function handle(string $query, ?string $category = null): array
    {
        return Product::query()
            ->when($category, fn($q) => $q->whereHas('category', fn($q) => $q->where('name', $category)))
            ->where('name', 'like', "%{$query}%")
            ->active()
            ->limit(5)
            ->get(['id', 'name', 'price'])
            ->toArray();
    }
}
```

### توليد الصور

```php
use Illuminate\Support\Facades\Ai;

$image = Ai::image('صمم شعاراً لمتجر ملابس نسائية عصري');
Storage::put('logos/generated.png', $image->content());
```

### Embeddings والبحث الدلالي

```php
$embedding = Ai::embed('ما هي سياسة الشحن؟');
$vector = $embedding->embeddings();

// البحث في قاعدة البيانات باستخدام cosine similarity
$results = DB::table('knowledge_base')
    ->orderByRaw('embedding <-> ?', [json_encode($vector)])
    ->limit(5)
    ->get();
```

### Laravel Boost (للتطوير بمساعدة AI)

```bash
composer require laravel/boost --dev
php artisan boost:install
```

يمنح Boost وكلاء AI:
- وصول لأكثر من **17,000** وثيقة من نظام Laravel
- أدوات للاستعلام عن قاعدة البيانات
- البحث في التوثيق
- تنفيذ الكود عبر Tinker
- قراءة سجلات المتصفح

---

## 11. الطوابير والجدولة والأحداث

### Queues (الطوابير)

```php
// إنشاء Job
php artisan make:job SendWelcomeEmail

// app/Jobs/SendWelcomeEmail.php
class SendWelcomeEmail implements ShouldQueue
{
    use Queueable;

    public int $tries = 3;
    public int $timeout = 60;

    public function __construct(private readonly User $user) {}

    public function handle(): void
    {
        Mail::to($this->user)->send(new WelcomeMailable($this->user));
    }

    public function failed(Throwable $exception): void
    {
        Log::error('فشل إرسال البريد: ' . $exception->getMessage());
    }
}

// إرسال الـ Job للطابور
SendWelcomeEmail::dispatch($user);
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(10));
SendWelcomeEmail::dispatch($user)->onQueue('emails');

// تشغيل Worker
php artisan queue:work
php artisan queue:work --queue=emails,default
php artisan horizon  // لـ Redis (مع Horizon)
```

### Task Scheduling (الجدولة)

```php
// routes/console.php أو app/Console/Kernel.php
Schedule::command('reports:generate')->dailyAt('08:00');
Schedule::job(new CleanupTempFiles)->weekly()->sundays()->at('02:00');
Schedule::call(function () {
    DB::table('sessions')->where('last_activity', '<', now()->subDays(30))->delete();
})->daily();

// تشغيل الجدولة (في cron)
* * * * * cd /path-to-project && php artisan schedule:run >> /dev/null 2>&1
```

### Events & Listeners

```php
// إنشاء Event و Listener
php artisan make:event OrderPlaced
php artisan make:listener SendOrderConfirmation --event=OrderPlaced

// app/Events/OrderPlaced.php
class OrderPlaced
{
    public function __construct(public readonly Order $order) {}
}

// app/Listeners/SendOrderConfirmation.php
class SendOrderConfirmation
{
    public function handle(OrderPlaced $event): void
    {
        Mail::to($event->order->customer)
            ->send(new OrderConfirmationMail($event->order));
    }
}

// إطلاق الحدث
event(new OrderPlaced($order));
// أو
OrderPlaced::dispatch($order);
```

### Broadcasting (الأحداث اللحظية)

```php
// app/Events/NewMessage.php
class NewMessage implements ShouldBroadcast
{
    public function broadcastOn(): array
    {
        return [new PrivateChannel('chat.' . $this->message->conversation_id)];
    }

    public function broadcastWith(): array
    {
        return [
            'id'      => $this->message->id,
            'content' => $this->message->content,
            'sender'  => $this->message->sender->name,
        ];
    }
}

// JavaScript (Laravel Echo)
Echo.private(`chat.${conversationId}`)
    .listen('NewMessage', (data) => {
        messages.push(data);
    });
```

---

## 12. الاختبار — Testing

### اختبارات HTTP

```php
// tests/Feature/ProductTest.php
class ProductTest extends TestCase
{
    use RefreshDatabase;

    public function test_authenticated_user_can_create_product(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->postJson('/api/products', [
            'name'        => 'قميص جديد',
            'price'       => 89.99,
            'category_id' => Category::factory()->create()->id,
        ]);

        $response->assertCreated()
                 ->assertJsonPath('name', 'قميص جديد');

        $this->assertDatabaseHas('products', ['name' => 'قميص جديد']);
    }

    public function test_unauthenticated_user_cannot_access_products(): void
    {
        $this->getJson('/api/products')->assertUnauthorized();
    }
}
```

### Pest (إطار الاختبار الحديث)

```php
// tests/Feature/ProductTest.php (Pest)
uses(RefreshDatabase::class);

it('can create a product', function () {
    $user = User::factory()->create();

    $response = actingAs($user)->postJson('/api/products', [
        'name'  => 'منتج جديد',
        'price' => 50.00,
    ]);

    $response->assertCreated();
    expect(Product::count())->toBe(1);
});

it('validates required fields', function () {
    actingAs(User::factory()->create())
        ->postJson('/api/products', [])
        ->assertUnprocessable()
        ->assertJsonValidationErrors(['name', 'price']);
});
```

### اختبار الـ Mocking

```php
// Mock للـ Mail
Mail::fake();
$this->post('/orders', $orderData);
Mail::assertSent(OrderConfirmationMail::class, function ($mail) use ($user) {
    return $mail->hasTo($user->email);
});

// Mock للـ Queue
Queue::fake();
$this->post('/register', $userData);
Queue::assertPushed(SendWelcomeEmail::class);

// Mock للـ Storage
Storage::fake('s3');
$response = $this->post('/upload', ['file' => UploadedFile::fake()->image('photo.jpg')]);
Storage::disk('s3')->assertExists('uploads/photo.jpg');
```

---

## 13. الحزم الرسمية

| الحزمة | الوصف | التثبيت |
|--------|-------|---------|
| **Sanctum** | API Authentication بالـ Tokens | `laravel/sanctum` |
| **Passport** | OAuth2 Server | `laravel/passport` |
| **Horizon** | مراقبة Redis Queues | `laravel/horizon` |
| **Telescope** | أداة Debug شاملة | `laravel/telescope` |
| **Scout** | Full-Text Search | `laravel/scout` |
| **Cashier** | Stripe Billing | `laravel/cashier` |
| **Reverb** | WebSocket Server | `laravel/reverb` |
| **Octane** | تسريع التطبيق (Swoole/FrankenPHP) | `laravel/octane` |
| **Sail** | Docker بيئة تطوير | `laravel/sail` |
| **Pint** | Code Style Fixer | `laravel/pint` |
| **Pulse** | مراقبة الأداء | `laravel/pulse` |
| **Pennant** | Feature Flags | `laravel/pennant` |
| **Folio** | File-based Routing | `laravel/folio` |
| **AI SDK** | تكامل الذكاء الاصطناعي | `laravel/ai` |

---

## 14. النشر والإنتاج

### قائمة التحقق قبل النشر

```bash
# 1. تحسين الإعداد
php artisan config:cache    # دمج ملفات الإعداد
php artisan route:cache     # تخزين المسارات مؤقتاً
php artisan view:cache      # تجميع Blade مسبقاً
php artisan event:cache     # تخزين الأحداث

# 2. تثبيت Dependencies بدون dev
composer install --optimize-autoloader --no-dev

# 3. تجميع الـ Assets
npm run build

# 4. تشغيل Migrations
php artisan migrate --force

# 5. تحديث الـ Storage Link
php artisan storage:link
```

### متغيرات البيئة الأساسية للإنتاج

```ini
APP_ENV=production
APP_DEBUG=false
APP_URL=https://myapp.com

# قاعدة البيانات
DB_CONNECTION=mysql
DB_HOST=db.myapp.com
DB_DATABASE=production_db

# الكاش
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

# Redis
REDIS_HOST=redis.myapp.com
REDIS_PASSWORD=secret

# Mail
MAIL_MAILER=smtp
MAIL_HOST=smtp.myapp.com
```

### Laravel Cloud / Forge / Vapor

```
┌─────────────────────────────────────────────┐
│                                             │
│  Laravel Cloud  → نشر تلقائي قابل للتوسع  │
│  Laravel Forge  → إدارة الخوادم            │
│  Laravel Vapor  → Serverless على AWS       │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 15. أوامر Artisan المرجعية

### أوامر إنشاء المكونات

```bash
# Models & Database
php artisan make:model Product -mcrf    # Model + Migration + Controller + Factory
php artisan make:migration add_stock_to_products_table
php artisan make:seeder ProductSeeder
php artisan make:factory ProductFactory

# HTTP
php artisan make:controller ProductController --resource --api
php artisan make:request StoreProductRequest
php artisan make:middleware CheckSubscription
php artisan make:resource ProductResource
php artisan make:collection ProductCollection

# Architecture
php artisan make:provider PaymentServiceProvider
php artisan make:event OrderPlaced
php artisan make:listener SendOrderConfirmation
php artisan make:observer ProductObserver --model=Product
php artisan make:policy ProductPolicy --model=Product
php artisan make:job ProcessPayment
php artisan make:command GenerateReports
php artisan make:mail OrderConfirmation --markdown
php artisan make:notification OrderShipped

# AI
php artisan make:agent CustomerSupportAgent
php artisan make:agent AnalysisAgent --structured

# Testing
php artisan make:test ProductTest
php artisan make:test ProductTest --unit
```

### أوامر الصيانة

```bash
# Cache
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan optimize:clear   # مسح كل الـ caches

# Database
php artisan migrate
php artisan migrate:fresh --seed
php artisan db:seed
php artisan tinker           # REPL تفاعلي

# Queue
php artisan queue:work
php artisan queue:restart
php artisan queue:failed     # عرض الـ Failed Jobs
php artisan queue:retry all  # إعادة الـ Failed Jobs

# Misc
php artisan about            # معلومات التطبيق
php artisan route:list       # قائمة المسارات
php artisan model:show User  # تفاصيل النموذج
php artisan schedule:list    # الجدولة المجدولة
php artisan down             # وضع الصيانة
php artisan up               # إلغاء وضع الصيانة
```

---

## 📌 مراجع سريعة

### Eloquent Relationships

```
hasOne          → User hasOne Profile
hasMany         → Post hasMany Comments
belongsTo       → Comment belongsTo Post
belongsToMany   → User belongsToMany Roles (pivot table)
hasManyThrough  → Country hasManyThrough Posts through Users
morphMany       → Image morphMany (Product, Post)
```

### HTTP Status Codes في Laravel

```php
response()->json($data, 200);   // OK
response()->json($data, 201);   // Created
response()->json(null, 204);    // No Content
response()->json($err, 400);    // Bad Request
response()->json($err, 401);    // Unauthorized
response()->json($err, 403);    // Forbidden
response()->json($err, 404);    // Not Found
response()->json($err, 422);    // Unprocessable Entity (Validation)
response()->json($err, 500);    // Server Error
```

### أهم Helper Functions

```php
// Strings
Str::slug('مرحبا بالعالم');      // 'mrḥba-balaālm'
Str::limit($text, 100);
Str::uuid();

// Arrays & Collections
collect([1,2,3])->filter()->map(fn($x) => $x * 2)->toArray();

// URL & Route
route('products.show', $product);
url('/images/photo.jpg');
asset('css/app.css');

// Environment & Config
