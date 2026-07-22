# 07. Routing & Controller

Route adalah **pintu masuk** setiap request ke aplikasi Laravel. Controller adalah tempat **logic penanganan request** ditulis.

## Tujuan Belajar

- Paham cara mendefinisikan route (GET, POST, PUT, DELETE).
- Paham route parameter dan route model binding.
- Bisa membuat Controller, termasuk **resource controller**.
- Paham konsep middleware secara ringkas.

## 1. Route Dasar

File: `routes/web.php`

```php
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return 'Halo dari Laravel!';
});

Route::get('/mahasiswa', function () {
    return view('mahasiswa.index');
});
```

Untuk melihat semua route yang aktif:
```bash
php artisan route:list
```

## 2. Route Menuju Controller

Daripada menaruh logic langsung di closure, arahkan ke method Controller:

```php
use App\Http\Controllers\MahasiswaController;

Route::get('/mahasiswa', [MahasiswaController::class, 'index']);
Route::get('/mahasiswa/{id}', [MahasiswaController::class, 'show']);
Route::post('/mahasiswa', [MahasiswaController::class, 'store']);
Route::put('/mahasiswa/{id}', [MahasiswaController::class, 'update']);
Route::delete('/mahasiswa/{id}', [MahasiswaController::class, 'destroy']);
```

## 3. Resource Route — 1 Baris untuk 7 Route CRUD

Laravel punya shortcut untuk pola CRUD standar:

```php
Route::resource('mahasiswa', MahasiswaController::class);
```

Baris ini otomatis membuat 7 route berikut:

| Method | URL | Controller Method | Fungsi |
|---|---|---|---|
| GET | `/mahasiswa` | `index` | Tampilkan semua data |
| GET | `/mahasiswa/create` | `create` | Tampilkan form tambah |
| POST | `/mahasiswa` | `store` | Simpan data baru |
| GET | `/mahasiswa/{id}` | `show` | Tampilkan 1 data |
| GET | `/mahasiswa/{id}/edit` | `edit` | Tampilkan form edit |
| PUT/PATCH | `/mahasiswa/{id}` | `update` | Update data |
| DELETE | `/mahasiswa/{id}` | `destroy` | Hapus data |

Generate controller yang sudah punya kerangka 7 method ini sekaligus:
```bash
php artisan make:controller MahasiswaController --resource
```

## 4. Membuat Controller

```bash
php artisan make:controller MahasiswaController --resource
```

Hasil di `app/Http/Controllers/MahasiswaController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Mahasiswa;
use Illuminate\Http\Request;

class MahasiswaController extends Controller
{
    public function index()
    {
        $mahasiswa = Mahasiswa::all();
        return view('mahasiswa.index', compact('mahasiswa'));
    }

    public function show(Mahasiswa $mahasiswa)
    {
        return view('mahasiswa.show', compact('mahasiswa'));
    }

    public function create()
    {
        return view('mahasiswa.create');
    }

    public function store(Request $request)
    {
        Mahasiswa::create($request->all());
        return redirect()->route('mahasiswa.index');
    }

    // edit(), update(), destroy() mengikuti pola yang sama
}
```

> Perhatikan nama variabel `$mahasiswa` (huruf kecil) untuk parameter route-model-binding, dan `Mahasiswa` (PascalCase) untuk nama class Model — konvensi penamaan ini konsisten dipakai Laravel di seluruh dokumentasi resminya.

## 5. Route Model Binding — Fitur "Ajaib" Laravel

Perhatikan `show(Mahasiswa $mahasiswa)` di atas — Laravel **otomatis** mencari record berdasarkan `{id}` di URL dan inject-kan object model-nya. Tanpa ini, kamu harus tulis manual:

```php
// Tanpa route model binding (cara manual)
public function show($id)
{
    $mahasiswa = Mahasiswa::findOrFail($id);
    return view('mahasiswa.show', compact('mahasiswa'));
}

// Dengan route model binding (otomatis)
public function show(Mahasiswa $mahasiswa)
{
    return view('mahasiswa.show', compact('mahasiswa'));
}
```

Laravel mencocokkan nama parameter route (`{mahasiswa}`) dengan nama variabel di method — kalau tidak ketemu, otomatis melempar **404** (tidak perlu `try/catch` manual).

## 6. Named Route

```php
Route::get('/mahasiswa', [MahasiswaController::class, 'index'])->name('mahasiswa.index');
```

Dipakai supaya tidak hardcode URL di banyak tempat:
```php
// Di controller
return redirect()->route('mahasiswa.index');

// Di Blade view
<a href="{{ route('mahasiswa.index') }}">Lihat Semua</a>
```
Kalau nanti URL `/mahasiswa` diganti jadi `/data-mahasiswa`, cukup ubah di `routes/web.php` — semua `route('mahasiswa.index')` di kode lain otomatis ikut berubah.

## 7. Middleware (Sekilas)

Middleware adalah "penjaga gerbang" sebelum request sampai ke Controller — misalnya cek apakah user sudah login.

```php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth');

Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/mahasiswa', [MahasiswaController::class, 'index']);
    Route::get('/dosen', [DosenController::class, 'index']);
});
```

> Contoh nyata middleware custom biasanya ada di `app/Http/Middleware/` — misalnya untuk membatasi akses berdasarkan role (mahasiswa/dosen/admin akademik).

## Studi Mini: Route + Controller Sederhana

```php
// routes/web.php
Route::resource('mahasiswa', MahasiswaController::class);
```

```php
// app/Http/Controllers/MahasiswaController.php
namespace App\Http\Controllers;

class MahasiswaController extends Controller
{
    public function index()
    {
        // Data sementara (belum pakai database — itu di modul 08)
        $mahasiswa = [
            ['stambuk' => '20210011', 'name' => 'Ahmad Fauzi', 'jurusan' => 'Teknik Informatika'],
            ['stambuk' => '20210012', 'name' => 'Siti Rahma', 'jurusan' => 'Sistem Informasi'],
        ];

        return view('mahasiswa.index', compact('mahasiswa'));
    }
}
```

## Latihan

1. Buat `Route::resource('mahasiswa', ...)` dan `MahasiswaController` dengan data array statis (belum perlu database).
2. Tambahkan named route dan gunakan `route()` helper di sebuah link Blade sederhana.
3. Buat middleware sederhana bernama `CekJamOperasional` yang menolak akses kalau jam sekarang di luar 08:00-17:00 (`php artisan make:middleware CekJamOperasional`), lalu pasang di satu route.

---
⬅️ [06. Instalasi & Struktur Laravel](../06-instalasi-struktur-laravel/README.md) | ➡️ Lanjut ke [08. Model & Migration](../08-model-migration-database/README.md)
