# 14. Setup API di Laravel

Sekarang kita ubah fitur "Mahasiswa" dari modul 12 (yang server-rendered lewat Blade) menjadi **REST API** — bahan yang nanti akan dikonsumsi React di modul 17-18.

## Tujuan Belajar

- Paham beda `routes/web.php` vs `routes/api.php`.
- Bisa membuat **API Resource Controller** yang mengembalikan JSON.
- Paham **API Resource class** untuk membentuk struktur JSON yang rapi & konsisten.
- Paham otentikasi API dengan **Laravel Sanctum**.
- Paham **CORS** dan kenapa itu penting saat frontend & backend beda origin/port.

## 1. `routes/web.php` vs `routes/api.php`

| | `routes/web.php` | `routes/api.php` |
|---|---|---|
| Middleware default | `web` (session, cookie, CSRF) | `api` (stateless, tanpa CSRF) |
| Response biasa | HTML (Blade view) | JSON |
| Prefix URL | Tidak ada prefix | Otomatis diawali `/api` |
| Dipakai untuk | Website server-rendered, form dengan session | Dikonsumsi frontend terpisah (React, mobile app) |

```php
// routes/api.php
use App\Http\Controllers\Api\MahasiswaController;
use Illuminate\Support\Facades\Route;

Route::apiResource('mahasiswa', MahasiswaController::class);
```

`Route::apiResource()` mirip `Route::resource()` tapi **tanpa** 2 route yang hanya relevan untuk HTML (`create` dan `edit`, karena API tidak butuh "halaman form") — hanya 5 route: `index`, `store`, `show`, `update`, `destroy`.

```bash
php artisan route:list --path=api
```

## 2. API Controller Terpisah

Kebiasaan baik: pisahkan namespace Controller API dari Controller web, supaya tidak tercampur.

```bash
php artisan make:controller Api/MahasiswaController --api --model=Mahasiswa
```

```php
<?php
// app/Http/Controllers/Api/MahasiswaController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreMahasiswaRequest;
use App\Http\Requests\UpdateMahasiswaRequest;
use App\Http\Resources\MahasiswaResource;
use App\Models\Mahasiswa;
use App\Services\MahasiswaService;

class MahasiswaController extends Controller
{
    public function __construct(
        protected MahasiswaService $mahasiswaService,
    ) {}

    public function index()
    {
        $mahasiswa = $this->mahasiswaService->semua();

        return MahasiswaResource::collection($mahasiswa);
    }

    public function store(StoreMahasiswaRequest $request)
    {
        $mahasiswa = $this->mahasiswaService->buat($request->validated());

        return new MahasiswaResource($mahasiswa);
    }

    public function show(Mahasiswa $mahasiswa)
    {
        return new MahasiswaResource($mahasiswa);
    }

    public function update(UpdateMahasiswaRequest $request, Mahasiswa $mahasiswa)
    {
        $mahasiswa = $this->mahasiswaService->perbarui($mahasiswa, $request->validated());

        return new MahasiswaResource($mahasiswa);
    }

    public function destroy(Mahasiswa $mahasiswa)
    {
        $this->mahasiswaService->hapus($mahasiswa);

        return response()->json(null, 204);
    }
}
```

> Perhatikan: `MahasiswaService` dan `StoreMahasiswaRequest`/`UpdateMahasiswaRequest` yang dibuat di modul 09-10 **dipakai ulang persis sama** di sini — inilah manfaat konkret memisahkan logic bisnis ke Service sejak awal (modul 10). Controller API dan Controller web sama-sama tipis, tidak ada logic yang diduplikasi.

## 3. API Resource — Bentuk JSON yang Konsisten

Tanpa Resource, `return $mahasiswa;` akan mengembalikan **semua kolom** apa adanya (termasuk kolom yang mungkin tidak ingin diekspos). API Resource memberi kontrol penuh atas bentuk JSON output.

```bash
php artisan make:resource MahasiswaResource
```

```php
<?php
// app/Http/Resources/MahasiswaResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class MahasiswaResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'stambuk' => $this->stambuk,
            'name' => $this->name,
            'jurusan' => $this->jurusan,
            'angkatan' => $this->angkatan(), // bisa panggil method dari Model
            'created_at' => $this->created_at->format('Y-m-d'),
        ];
    }
}
```

Hasil JSON untuk `index()` (dibungkus otomatis dalam key `data`):

```json
{
  "data": [
    {
      "id": 1,
      "stambuk": "20210011",
      "name": "Ahmad Fauzi",
      "jurusan": "Teknik Informatika",
      "angkatan": "2021",
      "created_at": "2026-07-01"
    }
  ]
}
```

## 4. Otentikasi API dengan Laravel Sanctum

Untuk endpoint yang butuh login (misal `store`/`update`/`destroy` hanya boleh admin akademik), Laravel Sanctum menyediakan **token-based authentication** yang ringan — cocok untuk SPA (Single Page Application) React yang berjalan di domain terpisah/sama.

```bash
composer require laravel/sanctum
php artisan install:api
```

Melindungi route:

```php
// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('mahasiswa', MahasiswaController::class)
        ->except(['index', 'show']); // index & show tetap publik
});

Route::apiResource('mahasiswa', MahasiswaController::class)->only(['index', 'show']);
```

Endpoint login untuk mendapatkan token (contoh sederhana):

```php
// app/Http/Controllers/Api/AuthController.php
public function login(Request $request)
{
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        return response()->json(['message' => 'Kredensial tidak valid'], 401);
    }

    $token = $user->createToken('api-token')->plainTextToken;

    return response()->json(['token' => $token]);
}
```

Klien (React, Postman) menyertakan token ini di setiap request selanjutnya:
```
Authorization: Bearer <token-yang-didapat>
```

Mengambil user yang sedang login dari token:
```php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

## 5. CORS — Kenapa Penting Saat Frontend Beda Port/Domain

Saat React (modul 15, biasanya jalan di `http://localhost:5173`) memanggil API Laravel (`http://localhost:8000`), browser akan **memblokir** request tersebut secara default karena beda *origin* (protokol+domain+port) — ini disebut **CORS (Cross-Origin Resource Sharing)** policy, sebuah mekanisme keamanan bawaan browser.

Konfigurasi di `config/cors.php`:

```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],

'allowed_methods' => ['*'],

'allowed_origins' => ['http://localhost:5173'], // origin React dev server

'allowed_headers' => ['*'],

'supports_credentials' => true, // wajib true kalau pakai Sanctum SPA auth (cookie-based)
```

> **Jangan** pakai `allowed_origins => ['*']` (wildcard) di produksi kalau `supports_credentials` juga `true` — kombinasi ini berisiko keamanan. Selalu daftarkan origin frontend secara eksplisit.

## 6. Menguji API dengan Postman/Thunder Client

```
GET http://localhost:8000/api/mahasiswa
Accept: application/json
```

```
POST http://localhost:8000/api/mahasiswa
Accept: application/json
Content-Type: application/json
Authorization: Bearer <token>

{
  "stambuk": "20210013",
  "name": "Budi Santoso",
  "jurusan": "Teknik Informatika"
}
```

## Latihan

1. Buat `Api\MahasiswaController`, `MahasiswaResource`, dan daftarkan `Route::apiResource` di `routes/api.php`, memakai ulang `MahasiswaService` dan Form Request dari modul 12.
2. Uji semua 5 endpoint (`index`, `store`, `show`, `update`, `destroy`) lewat Postman/Thunder Client.
3. Pasang Sanctum, lindungi endpoint `store`/`update`/`destroy` supaya butuh token, biarkan `index`/`show` publik.
4. Setup CORS supaya nanti bisa diakses dari React dev server di `http://localhost:5173`.
5. (Opsional) Ulangi §2-3 untuk `Api\DosenController` dan `Api\MataKuliahController`.

---
⬅️ [13. Konsep REST API](../13-konsep-api-rest/README.md) | ➡️ Lanjut ke [15. Setup React + Vite](../15-react-vite-setup/README.md)
