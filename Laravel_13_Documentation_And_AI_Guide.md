# وثيقة مرجعية شاملة: إطار عمل Laravel 13.x وميزات الذكاء الاصطناعي (Agent Ready Framework)

## 1. مقدمة عن Laravel 13.x
إطار عمل Laravel 13.x هو أحدث إصدار مستقر (مخطط له لعام 2026) مبني على لغة PHP 8.5+، ويُعرف بكونه **"The clean stack for Artisans and agents"**. يركز هذا الإصدار بشكل أساسي على دمج تقنيات الذكاء الاصطناعي التوليدي، وتسهيل عمل الوكلاء البرمجيين (AI Agents)، مع الحفاظ على القوة والمرونة المعهودة في إدارة قواعد البيانات، الطوابير (Queues)، والمهام المجدولة.

---

## 2. ميزة الجاهزية للوكلاء الذكاء الاصطناعي (Agent Ready Framework)
تمت إعادة صياغة وهيكلة العديد من الجوانب في Laravel 13 ليكون متوافقاً بشكل كامل مع بيئات التطوير المدعومة بالذكاء الاصطناعي مثل **Cursor** و **Claude Code**.

* **الاصطلاحات الثابتة (Opinionated Conventions):** يمتلك Laravel بنية متوقعة وهيكلية واضحة (مثل أماكن الـ Controllers، والـ Migrations). عندما يطلب المطور من وكيل الذكاء الاصطناعي إنشاء عنصر جديد، يعرف الوكيل بدقة المكان الصحيح لوضعه وبنيته البرمجية دون تخمين.
* **السياق الممتد والتوثيق الوثيق:** يوفر الإطار للوكلاء سياقاً غنياً بفضل الصياغة التعبيرية (Expressive Syntax) مثل علاقات Eloquent، و Form Requests، والـ Middleware، مما يتيح توليد كود دقيق واحترافي يشبه كود المبرمجين المحترفين.
* **دليل التثبيت والتهيئة للوكلاء (AI Playbook):** يمكن توجيه وكلاء الذكاء الاصطناعي لقراءة دليل خاص وموحد عبر الرابط المخصص للوكلاء: `https://laravel.com/for/agents`.

---

## 3. الأداة الثورية: Laravel Boost
تعد حزمة **Laravel Boost** الجسر الأساسي الذي يربط بين وكلاء الذكاء الاصطناعي وتطبيق Laravel الخاص بك، حيث تمد الوكلاء بسياق مخصص يعتمد على إصدار المشروع والحزم المستخدمة.

### مميزات Laravel Boost:
1. **توفير أدوات متخصصة (15+ Specialized Tools):** تمنح الوكيل القدرة على معرفة الحزم المثبتة، الاستعلام من قاعدة البيانات، البحث في توثيق Laravel، قراءة سجلات المتصفح (Browser Logs)، توليد الاختبارات، وتنفيذ الكود مباشرة عبر `Tinker`.
2. **التوثيق الموجه (Vectorized Documentation):** تتيح للوكلاء الوصول إلى أكثر من 17,000 قطعة توثيق مُمثلة برمزية متجهة (Vectorized) خاصة بالإصدارات المحددة للمشروع.
3. **الإرشادات المخصصة (Custom AI Guidelines):** يمكن للمطور إضافة إرشادات مخصصة للذكاء الاصطناعي داخل المشروع عبر إنشاء ملفات `.blade.php` أو `.md` في المسار `.ai/guidelines/*` لتؤخذ في الاعتبار تلقائياً عند تشغيل الأداة.

### تثبيت Laravel Boost:
تتوافق الحزمة مع إصدارات Laravel 10، 11، 12، و 13 التي تعمل على إصدار PHP 8.1 أو أعلى. يتم التثبيت عبر Composer كاعتمادية تطويرية:
```bash
composer require laravel/boost --dev
```
ثم تشغيل أداة التثبيت التفاعلية لتحديد بيئة التطوير (IDE) والوكلاء المتاحين:
```bash
php artisan boost:install
```

---

## 4. التثبيت والتهيئة الأولية (Installation & Setup)

### متطلبات البيئة الأساسية:
يتطلب Laravel 13 وجود PHP 8.5، و Composer، بالإضافة إلى Node.js و NPM (أو Bun) لإدارة الأصول الأمامية (Frontend Assets).

### التثبيت السريع للبيئة عبر السكربت الموحد (php.new):
* **macOS:**
  ```bash
  /bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.5)"
  ```
* **Windows (PowerShell كمسؤول):**
  ```powershell
  Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.5'))
  ```
* **Linux:**
  ```bash
  /bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.5)"
  ```

### إنشاء تطبيق جديد:
يتم إنشاء المشروع عبر التثبيت العالمي للمثبت أو تشغيله مباشرة:
```bash
composer global require laravel/installer
laravel new example-app
```
*أثناء الإنشاء، سيقوم المثبت بسؤالك تفاعلياً عن إطار الاختبار المفضل (Testing Framework)، قاعدة البيانات (Database)، وحزمة البدء (Starter Kit).*

### تشغيل خادم التطوير الموحد:
في Laravel 13، يتيح أمر `composer run dev` تشغيل خادم Laravel المحلي، ومراقب الطوابير (Queue Worker)، وخادم Vite للأصول الأمامية دفعة واحدة:
```bash
cd example-app
npm install && npm run build
composer run dev
```
سيكون التطبيق متاحاً مباشرة عبر الرابط: `http://localhost:8000`.

---

## 5. بيئة التطوير Laravel Herd
أصبحت أداة **Laravel Herd** الخيار القياسي والأسرع لتطوير تطبيقات Laravel على macOS و Windows، حيث تأتي مدمجة ومحملة بـ PHP و Nginx والعديد من الأدوات المساعدة مثل `composer`, `laravel`, `node`, `npm`, `nvm`.

* **نطاقات التطوير الملقاة (Parked Directories):** بمجرد وضع المشروع في مجلد `~/Herd` (على الماك) أو `%USERPROFILE%\Herd` (على ويندوز)، يتم ربطه تلقائياً بنطاق محلي يحمل الاسم `.test` (مثال: `my-app.test`) عبر نظام الـ `dnsmasq`.
* **ميزات Herd Pro:** توفر واجهة رسومية لإدارة قواعد البيانات (MySQL, PostgreSQL)، نظام الكاش (Redis)، مستعرض البريد المحلي (Mail Viewing)، ومراقبة السجلات (Log Monitoring).

---

## 6. بنية التهيئة وقواعد البيانات (Configuration & Databases)

* **ملفات الإعدادات:** يتم تخزين كافة ملفات الإعدادات داخل مجلد `config/`.
* **الملف البيئي (`.env`):** يعتمد Laravel على إعدادات متغيرة طبقاً لبيئة التشغيل (محلي أو إنتاجي). افتراضياً، يأتي Laravel 13 مهيأً للعمل مع قاعدة بيانات **SQLite**، ويقوم بإنشاء ملف `database/database.sqlite` تلقائياً وتشغيل التهجيرات (Migrations) الأولية.
* **التحويل لقواعد بيانات أخرى (مثل MySQL):** يتم تعديل متغيرات البيئة في ملف `.env` كالتالي:
  ```env
  DB_CONNECTION=mysql
  DB_HOST=127.0.0.1
  DB_PORT=3306
  DB_DATABASE=laravel
  DB_USERNAME=root
  DB_PASSWORD=
  ```
  ثم تنفيذ الأمر لبناء الجداول:
  ```bash
  php artisan migrate
  ```

---

## 7. مسارات استخدام Laravel (Application Archetypes)
يمكن توظيف Laravel 13 في نمطين رئيسيين بناءً على المعمارية المستهدفة:

1. **إطار عمل متكامل (Full Stack Framework):**
   * توجيه الطلبات ومعالجة البيانات وعرض واجهات المستخدم باستخدام **Blade Templates**، أو دمج تقنيات هجينة لصفحة واحدة (SPA) مثل **Inertia.js** (مع React أو Vue) أو **Livewire**، والاستفادة الكاملة من **Vite** لتجميع الملفات.
2. **خلفية برمجية للـ APIs فقط (API Backend):**
   * يعمل Laravel كخلفية برمجية صلبة لتطبيقات الموبايل أو أطر عمل الويب المنفصلة (مثل Next.js). يتم الاعتماد هنا على حزم مثل **Laravel Sanctum** لإدارة المصادقة (Authentication) واستخدام الـ Eloquent ORM لتوفير البيانات عبر نقاط اتصال سريعة ومعالجة العمليات المعقدة عبر الطوابير (Queues).

---

## 8. أدوات الذكاء الاصطناعي المتقدمة في التوثيق (مؤشرات للتوسيع المستقبلي)
يتضمن التوثيق الرسمي لـ Laravel 13 تبويبات جديدة مخصصة بالكامل للذكاء الاصطناعي وتشمل:
* **AI SDK:** لتطوير وبرمجة نماذج الذكاء الاصطناعي مباشرة من داخل التطبيق.
* **MCP (Model Context Protocol):** لدعم بروتوكول سياق النماذج الموحد وتسهيل تبادل البيانات بين التطبيق والنماذج الذكية بشكل قياسي.
