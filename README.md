# 🚀 Update di Next.js 16

Next.js 16 membawa peningkatan signifikan dalam performa dan arsitektur aplikasi. Dokumen ini menjelaskan dua fitur utama: **React Compiler** yang kini stabil dan **penyempurnaan sistem routing**.

---

## ⚡ React Compiler Kini Stabil

React Compiler kini **stabil dan terintegrasi langsung** di Next.js 16. Compiler ini secara otomatis melakukan **memoization pada komponen** yang tidak perlu dirender ulang.

### 🎯 Keuntungan Utama

Dengan React Compiler, kita **tidak lagi perlu menggunakan `useMemo` atau `useCallback` secara manual** untuk sebagian besar kasus. Compiler secara otomatis menganalisis dependensi dan mengoptimalkan perilaku rendering.

### 🔧 Cara Mengaktifkan

**1. Install Plugin Babel:**

```bash
npm install babel-plugin-react-compiler@latest
```

**2. Konfigurasi Next.js:**

Buka file `next.config.ts` dan tambahkan konfigurasi berikut:

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
    reactCompiler: true,
    experimental: {
        turbopackFileSystemCacheForDev: true,
    },
};

export default nextConfig;
```

---

## 🗺️ Penyempurnaan Sistem Routing

Next.js 16 membawa peningkatan besar pada sistem routing yang sebelumnya diperkenalkan di App Router (Next.js 13). Sistem ini kini lebih **fleksibel**, **modular**, dan **terintegrasi penuh** dengan konsep nested layout.

### 📂 Struktur Routing Modern

Setiap folder di dalam `app/` secara otomatis mewakili segment route. File-file khusus memiliki fungsi tersendiri:

| File | Fungsi |
|------|--------|
| `page.tsx` | Halaman yang dirender untuk segment tersebut |
| `layout.tsx` | Layout untuk segment dan semua turunannya |
| `loading.tsx` | Ditampilkan saat data sedang dimuat |
| `error.tsx` | Ditampilkan saat terjadi error pada segment |
| `not-found.tsx` | Menangani halaman 404 khusus segment |

### 🧩 Layout Bersarang (Nested Layout)

Layout bersarang memungkinkan setiap segment memiliki tampilan dan struktur sendiri tanpa mengulang kode global.

#### Root Layout (`app/layout.tsx`)
- Layout global yang membungkus seluruh aplikasi
- Berisi elemen umum seperti `<Header />`, `<Footer />`, atau theme provider

#### Nested Layout (`app/blog/layout.tsx`)
- Hanya berlaku untuk semua halaman di dalam `/blog`
- Tetap dibungkus oleh root layout

**Hierarki Render:**

```
RootLayout → BlogLayout → Page
```

### 🔒 Layout Terisolasi (Independent Layout)

Next.js 16 juga mendukung layout yang **tidak mewarisi root layout**, cocok untuk halaman seperti dashboard admin atau halaman login.

**Cara Membuat Layout Terisolasi:**

1. Buat folder dengan tanda kurung, contoh: `app/(isolated)/dashboard/`
2. Tambahkan `layout.tsx` di dalamnya

```tsx
export const dynamic = 'force-static'; // opsional
export const metadata = { title: 'Dashboard' };

export default function DashboardLayout({ children }) {
    return (
        <html>
        <body>
        <Sidebar />
        <main>{children}</main>
        </body>
        </html>
    );
}
```

> 📌 **Catatan:** Folder dengan tanda kurung `()` seperti `(isolated)` akan dianggap sebagai **layout group**. Jika berisi `layout.tsx` sendiri, maka layout tersebut tidak lagi mewarisi root layout utama.

### ✅ Keuntungan Sistem Routing Baru

- **Lebih modular** dan mudah diatur per-segment
- **Layout tetap persisten** antar navigasi, menjaga state dan performa
- Dapat membuat **layout independen** tanpa mengganggu tampilan global
- Mendukung **data fetching per route segment** dengan lebih efisien

---

## 🛡️ Error Handling dengan Hierarki

Next.js 16 menyediakan sistem **error handling hierarkis** yang memungkinkan penanganan error secara granular berdasarkan segment route. Sistem ini bekerja dengan prinsip **error boundary** yang menangkap error di level terdekat.

### 📍 Cara Kerja Error Hierarchy

Error akan ditangkap oleh file `error.tsx` yang **paling dekat** dengan segment yang mengalami error. Jika tidak ada, maka akan naik ke parent segment hingga menemukan error handler.

**Contoh Struktur:**

```
app/
├── error.tsx              # Global error handler
├── layout.tsx
├── page.tsx
└── admin/
    ├── error.tsx          # Admin-specific error handler
    ├── layout.tsx
    ├── page.tsx
    └── users/
        ├── page.tsx       # Akan menggunakan admin/error.tsx
        └── [id]/
            ├── error.tsx  # User detail error handler
            └── page.tsx
```

### 🎯 Alur Penangkapan Error

**Hierarki Penanganan:**

```
app/admin/users/[id]/error.tsx  →  app/admin/error.tsx  →  app/error.tsx
     (Paling prioritas)              (Cadangan level 2)      (Global fallback)
```

#### Skenario 1: Error di `/admin/users/[id]`
Error akan ditangkap oleh `app/admin/users/[id]/error.tsx` karena paling dekat.

#### Skenario 2: Error di `/admin/users` (tanpa error.tsx di users/)
Error akan ditangkap oleh `app/admin/error.tsx` karena naik satu level.

#### Skenario 3: Error di `/admin` (tanpa error.tsx di admin/)
Error akan ditangkap oleh `app/error.tsx` sebagai global fallback.

### 🔨 Implementasi Error Handler

**Global Error Handler (`app/error.tsx`):**

```tsx
'use client';

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <div className="text-center">
        <h2 className="text-2xl font-bold text-red-600">
          Terjadi Kesalahan Global
        </h2>
        <p className="mt-2 text-gray-600">{error.message}</p>
        <button
          onClick={reset}
          className="mt-4 rounded bg-blue-500 px-4 py-2 text-white"
        >
          Coba Lagi
        </button>
      </div>
    </div>
  );
}
```

**Segment-Specific Error Handler (`app/admin/error.tsx`):**

```tsx
'use client';

import { useEffect } from 'react';

export default function AdminError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log error ke service monitoring
    console.error('Admin Error:', error);
  }, [error]);

  return (
    <div className="rounded-lg border border-red-200 bg-red-50 p-6">
      <h2 className="text-xl font-semibold text-red-700">
        Error di Area Admin
      </h2>
      <p className="mt-2 text-sm text-red-600">
        {error.message || 'Terjadi kesalahan pada dashboard admin'}
      </p>
      <div className="mt-4 flex gap-2">
        <button
          onClick={reset}
          className="rounded bg-red-600 px-4 py-2 text-white hover:bg-red-700"
        >
          Reset Error
        </button>
        <button
          onClick={() => window.location.href = '/admin'}
          className="rounded border border-red-600 px-4 py-2 text-red-600"
        >
          Kembali ke Dashboard
        </button>
      </div>
    </div>
  );
}
```

### 🎨 Root Error Handler Khusus

Untuk menangani error di **root layout** itu sendiri, gunakan `global-error.tsx`:

```tsx
'use client';

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>Aplikasi Mengalami Error Kritis</h2>
        <button onClick={reset}>Reset Aplikasi</button>
      </body>
    </html>
  );
}
```

> ⚠️ **Penting:** `global-error.tsx` harus menyertakan tag `<html>` dan `<body>` karena menggantikan root layout sepenuhnya.

### 📊 Perbandingan Error Files

| File | Lokasi | Fungsi | Wajib `'use client'` |
|------|--------|--------|---------------------|
| `error.tsx` | Semua segment | Menangkap error di segment dan child-nya | ✅ Ya |
| `global-error.tsx` | Root (`app/`) | Menangkap error di root layout | ✅ Ya |
| `not-found.tsx` | Semua segment | Menangani 404 khusus segment | ❌ Tidak |

### ✨ Best Practices

1. **Gunakan Error Boundary Spesifik** - Buat `error.tsx` di segment kritis seperti `/admin`, `/dashboard`, atau `/checkout`
2. **Log Error ke Monitoring Service** - Gunakan `useEffect` untuk mengirim error ke Sentry, LogRocket, dll
3. **Berikan Aksi Pemulihan** - Selalu sediakan tombol `reset()` atau navigasi alternatif
4. **Tampilkan Pesan User-Friendly** - Hindari menampilkan stack trace teknis ke user
5. **Testing Error States** - Gunakan Error Boundary untuk testing dengan melempar error secara manual

### 🔍 Testing Error Handler

Untuk menguji error handler, buat komponen yang melempar error:

```tsx
// app/admin/test-error/page.tsx
'use client';

export default function TestErrorPage() {
  return (
    <button onClick={() => {
      throw new Error('Test error di admin section');
    }}>
      Trigger Error
    </button>
  );
}
```

---

## 🧠 Kesimpulan

Next.js 16 bukan sekadar pembaruan performa, tetapi juga penyempurnaan arsitektur:

- ⚡ **React Compiler** meningkatkan efisiensi render secara otomatis
- 🗺️ **Routing dan Layout** menjadi lebih fleksibel dan terstruktur
- 🛡️ **Error Handling Hierarkis** memberikan kontrol granular atas penanganan error
- 🎨 Pengembang kini memiliki **kendali penuh** atas alur UI, komposisi halaman, dan error recovery

Dengan fitur-fitur ini, Next.js 16 memberikan fondasi yang lebih kuat untuk membangun aplikasi web modern yang cepat, resilient, dan scalable.

---

**Selamat mengembangkan dengan Next.js 16! 🎉**