# 🏗️ دليل المعمارية التصميمية لـ Laravel 13 — shadcn/ui + Tailwind CSS

> النسخة 1.0 — أفضل الأدوات والمكتبات لمعمارية التصميم والواجهات في Laravel 13

---

## 📋 جدول المحتويات

1. [المعمارية المقترحة (Recommended Stack)](#1-المعمارية-المقترحة)
2. [shadcn/ui — المكونات الأساسية](#2-shadcnui)
3. [Tailwind CSS v4 — نظام التصميم](#3-tailwind-css-v4)
4. [Inertia.js + React — الجسر بين Backend و Frontend](#4-inertiajs--react)
5. [مكتبات Laravel UI الممتازة (2026)](#5-مكتبات-laravel-ui)
6. [هيكل المشروع المقترح](#6-هيكل-المشروع)
7. [سير العمل المقترح (Workflow)](#7-سير-العمل)
8. [Design Tokens — التكامل مع Laravel](#8-design-tokens)
9. [المصادر والمراجع](#9-المصادر)

---

## 1. المعمارية المقترحة

| الطبقة | التقنية | الدور |
|--------|---------|-------|
| **Backend** | Laravel 13 | إدارة المنطق والبيانات |
| **Frontend** | React 19 | واجهة المستخدم التفاعلية |
| **Bridge** | Inertia.js | التنقل بدون API منفصل |
| **Styling** | Tailwind CSS v4 | نظام التصميم |
| **Components** | shadcn/ui | مكونات UI جاهزة وقابلة للتخصيص |
| **Build** | Vite | بناء وتجميع الملفات |

---

## 2. shadcn/ui

### التثبيت

```bash
# إنشاء مشروع Laravel مع React Starter Kit
laravel new my-app

# تشغيل shadcn مع قالب Laravel
pnpm dlx shadcn@latest init --template laravel
```

### أو باستخدام shadcn/create:

```bash
pnpm dlx shadcn@latest init --preset [CODE] --template laravel
```

### المميزات

- ✅ 700+ كتلة جاهزة (Shadcn Blocks)
- ✅ 1000+ متغير من المكونات
- ✅ Shadcn Builder + Figma Design System
- ✅ دعم كامل لـ Tailwind CSS v4
- ✅ CLI v4 مع دعم Laravel رسمياً

### إضافة مكونات

```bash
# إضافة أي مكون من shadcn
npx shadcn@latest add button dialog dropdown-menu

# معاينة ما سيتم إضافته (بدون تنفيذ)
npx shadcn@latest add button --dry-run

# التحقق من التحديثات
npx shadcn@latest add button --diff
```

---

## 3. Tailwind CSS v4

### التثبيت

```bash
npm i tailwindcss @tailwindcss/vite
```

### التكوين (resources/css/app.css)

```css
@import "tailwindcss";

@theme {
    /* الألوان */
    --color-primary: #0f172a;
    --color-primary-foreground: #f8fafc;
    --color-brand-500: oklch(0.62 0.18 252);

    /* الطباعة */
    --font-sans: "Inter", system-ui, sans-serif;
    --font-display: "Satoshi", sans-serif;
    --font-arabic: "Noto Sans Arabic", sans-serif;

    /* المسافات */
    --spacing-sidebar: 16rem;

    /* الزوايا */
    --radius-lg: 0.625rem;
    --radius-xl: 1rem;

    /* الظلال */
    --shadow-soft: 0 12px 40px rgb(15 23 42 / 0.14);

    /* نقاط التوقف */
    --breakpoint-3xl: 120rem;
}
```

### أفضل الممارسات (2026)

| ✅ افعل | ❌ لا تفعل |
|---------|-----------|
| استخدم `@theme` لتعريف Design Tokens | تجنب القيم العشوائية (arbitrary values) كنظام تصميم ثانٍ |
| استخدم `@utility` للأنماط المخصصة | لا تعتمد على `tailwind.config.js` القديم |
| فعّل Prettier plugin لترتيب الكلاسات | لا تكرر الأنماط في كل مكون |
| نظم الألوان بـ OKLCH للدقة | تجنب أسماء الألوان العشوائية |

---

## 4. Inertia.js + React

### التثبيت

```bash
# تثبيت Inertia
composer require inertiajs/inertia-laravel
php artisan inertia:middleware

# تثبيت React + Vite
npm install react react-dom @vitejs/plugin-react @inertiajs/react
```

### إعداد vite.config.js

```javascript
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'
import path from 'path'

export default defineConfig({
  resolve: {
    alias: { '@': path.resolve(__dirname, 'resources/js') },
  },
  plugins: [
    laravel({
      input: ['resources/css/app.css', 'resources/js/app.jsx'],
      refresh: true,
    }),
    tailwindcss(),
    react(),
  ],
})
```

### إعداد app.blade.php (Root View)

```html
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title inertia>{{ config('app.name', 'Laravel') }}</title>
    @viteReactRefresh
    @vite(['resources/css/app.css', 'resources/js/app.jsx'])
    @inertiaHead
</head>
<body class="font-sans antialiased">
    @inertia
</body>
</html>
```

### إعداد app.jsx (Entry Point)

```jsx
import { createInertiaApp } from '@inertiajs/react'
import { createRoot } from 'react-dom/client'
import './bootstrap'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.jsx', { eager: true })
    return pages[`./Pages/${name}.jsx`]
  },
  setup({ el, App, props }) {
    createRoot(el).render(<App {...props} />)
  },
})
```

---

## 5. مكتبات Laravel UI الممتازة (2026)

| المكتبة | الوصف | التوافق | الاستخدام |
|---------|-------|---------|-----------|
| **Mary** | مكونات Blade رائعة لـ Livewire 3 + DaisyUI + Tailwind | Laravel v13 | مشاريع Livewire |
| **TallStackUI** | مجموعة قوية من مكونات Blade لـ Livewire | Laravel v13 | مشاريع Livewire |
| **Blade UI Kit** | مكتبة SVG Icons لـ Blade | Laravel v13 | أيقونات Blade |
| **Tabler Blade** | Starter Kit مبني على Tabler HTML Template | Laravel v13 | لوحات التحكم |
| **Tailwind Merge Laravel** | حل تلقائي لتعارضات Tailwind CSS classes | Laravel v13 | جميع المشاريع |

### تثبيت Mary (Livewire)

```bash
composer require robsontenorio/mary
php artisan mary:install
```

### تثبيت Tailwind Merge

```bash
composer require gehrisandro/tailwind-merge-laravel
```

---

## 6. هيكل المشروع المقترح

```
📁 my-laravel-app/
├── 📁 app/
│   ├── 📁 Http/
│   │   ├── 📁 Controllers/
│   │   └── 📁 Middleware/
│   │       └── HandleInertiaRequests.php
│   └── 📁 Models/
│
├── 📁 resources/
│   ├── 📁 js/
│   │   ├── 📁 Pages/              ← صفحات Inertia (React)
│   │   │   ├── Dashboard.jsx
│   │   │   └── Auth/
│   │   │       └── Login.jsx
│   │   ├── 📁 components/
│   │   │   ├── 📁 ui/             ← مكونات shadcn/ui
│   │   │   │   ├── button.jsx
│   │   │   │   ├── dialog.jsx
│   │   │   │   └── dropdown-menu.jsx
│   │   │   └── 📁 layout/         ← مكونات التخطيط
│   │   │       ├── Sidebar.jsx
│   │   │       └── Navbar.jsx
│   │   ├── 📁 hooks/              ← Custom Hooks
│   │   ├── 📁 lib/                ← Utilities
│   │   │   └── utils.js
│   │   ├── app.jsx                ← Entry Point
│   │   └── bootstrap.js
│   │
│   ├── 📁 css/
│   │   └── app.css                ← @theme tokens
│   │
│   └── 📁 views/
│       └── app.blade.php          ← Root View
│
├── 📁 routes/
│   └── web.php
│
├── vite.config.js
├── tailwind.config.js (legacy)
└── package.json
```

---

## 7. سير العمل المقترح (Workflow)

### المرحلة 1: التصميم

```
Figma
  ├── استخدم plugin shadcn/ui
  ├── تصدير Design Tokens
  └── تعريف المكونات والمتغيرات
```

### المرحلة 2: التطوير

```bash
# 1. إنشاء المشروع
laravel new my-app

# 2. تثبيت shadcn/ui
pnpm dlx shadcn@latest init --template laravel

# 3. إضافة المكونات المطلوبة
npx shadcn@latest add button card dialog form

# 4. تطوير الصفحات
# resources/js/Pages/Dashboard.jsx

# 5. تشغيل خادم التطوير
npm run dev
```

### المرحلة 3: التحقق والنشر

```bash
# بناء للإنتاج
npm run build

# نشر
php artisan serve
```

---

## 8. Design Tokens — التكامل مع Laravel

### تعريف Tokens في CSS

```css
/* resources/css/app.css */
@import "tailwindcss";

@theme {
    /* === الألوان === */
    --color-primary-50: #f8fafc;
    --color-primary-100: #f1f5f9;
    --color-primary-200: #e2e8f0;
    --color-primary-300: #cbd5e1;
    --color-primary-400: #94a3b8;
    --color-primary-500: #64748b;
    --color-primary-600: #475569;
    --color-primary-700: #334155;
    --color-primary-800: #1e293b;
    --color-primary-900: #0f172a;
    --color-primary-950: #020617;

    /* === العلامة التجارية === */
    --color-brand: oklch(0.62 0.18 252);
    --color-brand-light: oklch(0.72 0.15 252);
    --color-brand-dark: oklch(0.52 0.2 252);

    /* === الطباعة === */
    --font-sans: "Inter", system-ui, -apple-system, sans-serif;
    --font-display: "Satoshi", "Inter", sans-serif;
    --font-mono: "JetBrains Mono", "Fira Code", monospace;
    --font-arabic: "Noto Sans Arabic", "Tajawal", sans-serif;

    /* === المسافات === */
    --spacing-1: 0.25rem;
    --spacing-2: 0.5rem;
    --spacing-3: 0.75rem;
    --spacing-4: 1rem;
    --spacing-6: 1.5rem;
    --spacing-8: 2rem;
    --spacing-12: 3rem;
    --spacing-16: 4rem;
    --spacing-sidebar: 16rem;

    /* === الزوايا === */
    --radius-sm: 0.375rem;
    --radius-md: 0.5rem;
    --radius-lg: 0.625rem;
    --radius-xl: 1rem;
    --radius-2xl: 1.5rem;
    --radius-full: 9999px;

    /* === الظلال === */
    --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
    --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
    --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
    --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1);
    --shadow-soft: 0 12px 40px rgb(15 23 42 / 0.14);

    /* === نقاط التوقف === */
    --breakpoint-sm: 40rem;
    --breakpoint-md: 48rem;
    --breakpoint-lg: 64rem;
    --breakpoint-xl: 80rem;
    --breakpoint-2xl: 96rem;
    --breakpoint-3xl: 120rem;

    /* === الحركة === */
    --animate-fade-in: fade-in 0.3s ease-out;
    --animate-slide-up: slide-up 0.4s ease-out;
    --animate-scale-in: scale-in 0.2s ease-out;
}
```

### استخدام Tokens في React Components

```jsx
// مثال: Button Component
export function Button({ variant = 'primary', size = 'md', children, ...props }) {
  const variants = {
    primary: 'bg-brand text-white hover:bg-brand-dark',
    secondary: 'bg-primary-100 text-primary-900 hover:bg-primary-200',
    ghost: 'hover:bg-primary-100 text-primary-700',
  }

  const sizes = {
    sm: 'px-3 py-1.5 text-sm rounded-md',
    md: 'px-4 py-2 text-base rounded-lg',
    lg: 'px-6 py-3 text-lg rounded-xl',
  }

  return (
    <button
      className={`font-sans font-medium transition-all duration-200 ${variants[variant]} ${sizes[size]}`}
      {...props}
    >
      {children}
    </button>
  )
}
```

---

## 9. المصادر والمراجع

| المصدر | الرابط |
|--------|--------|
| shadcn/ui Laravel Docs | https://ui.shadcn.com/docs/installation/laravel |
| Laravel Daily Packages | https://laraveldaily.com/packages |
| Tailwind CSS Best Practices | https://benjamincrozat.com/tailwind-css |
| Laravel 13 Scaffolding Guide | https://gist.github.com/bhaidar/26f91387a4effae6131c775e0619f10b |
| shadcn/ui Components | https://ui.shadcn.com/docs/components |
| Inertia.js Laravel | https://inertiajs.com/ |
| Tailwind CSS v4 Docs | https://tailwindcss.com/docs |

---

## 📝 ملاحظات هامة

> **لا عشوائية، لا اجتهاد شخصي، ولغة برمجية موحدة بين المصمم والمطور.**

1. **تجنب القيم العشوائية**: لا تستخدم `w-[123px]` أو `text-[#1a2b3c]` مباشرة. استخدم Design Tokens.
2. **وحدة التسمية**: اتبع نظام تسمية موحد (BEM أو Utility-first).
3. **قابلية الصيانة**: كل مكون يجب أن يكون مستقلاً وقابلاً لإعادة الاستخدام.
4. **التوثيق**: وثق أي تخصيص أو تعديل على المكتبات.
5. **RTL Support**: تأكد من دعم اللغة العربية (dir="rtl") في جميع المكونات.

---

*تم إعداد هذا الدليل بناءً على أفضل الممارسات لعام 2026*
