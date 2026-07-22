# 10. Service Layer

Ini modul yang sering dilewatkan tutorial Laravel pemula — padahal justru inilah yang membedakan proyek "asal jalan" dengan proyek yang **scalable** dan mudah dites. Studi kasus di modul 12 akan memakai pola ini secara penuh.

## Tujuan Belajar

- Paham masalah yang terjadi kalau semua logic bisnis ditaruh di Controller ("Fat Controller").
- Paham apa itu Service class dan kapan menggunakannya.
- Bisa memisahkan logic bisnis dari Controller ke Service.
- Paham dependency injection sederhana lewat constructor.

## 1. Masalah: "Fat Controller"

Contoh Controller yang **semua logic-nya digabung jadi satu** (anti-pattern):

```php
class MahasiswaController extends Controller
{
    public function store(Request $request)
    {
        $validated = $request->validate([
            'stambuk' => 'required|string|max:11',
            'name' => 'required|string|max:255',
            'jurusan' => 'required|string|max:255',
        ]);

        $sudahAda = Mahasiswa::where('stambuk', $validated['stambuk'])->exists();

        if ($sudahAda) {
            return back()->withErrors(['stambuk' => 'Stambuk sudah terdaftar']);
        }

        $mahasiswa = Mahasiswa::create($validated);

        // Kirim email selamat datang
        Mail::to($request->email)->send(new MahasiswaBaruDiterima($mahasiswa));

        // Catat log aktivitas
        Log::info("Mahasiswa baru terdaftar: {$mahasiswa->name}");

        // Invalidate cache statistik jumlah mahasiswa per jurusan
        Cache::forget('statistik_mahasiswa_per_jurusan');

        return redirect()->route('mahasiswa.index');
    }
}
```

Masalahnya:
- Method `store()` melakukan **terlalu banyak hal** — validasi, cek duplikat, simpan data, kirim email, logging, cache — semua tercampur.
- Sulit di-**test** tanpa menjalankan seluruh HTTP request.
- Kalau logic yang sama dibutuhkan di tempat lain (misal dari command console import data mahasiswa massal, atau dari API di modul 14), harus **copy-paste**.

## 2. Solusi: Service Layer

Service class menampung **logic bisnis**, Controller hanya jadi "penghubung" antara HTTP request dan Service.

```bash
php artisan make:class Services/MahasiswaService
```

```php
<?php

namespace App\Services;

use App\Models\Mahasiswa;
use App\Mail\MahasiswaBaruDiterima;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Cache;
use Illuminate\Validation\ValidationException;

class MahasiswaService
{
    public function daftarkan(array $data, ?string $email = null): Mahasiswa
    {
        $sudahAda = Mahasiswa::where('stambuk', $data['stambuk'])->exists();

        if ($sudahAda) {
            throw ValidationException::withMessages([
                'stambuk' => 'Stambuk ini sudah terdaftar.',
            ]);
        }

        $mahasiswa = Mahasiswa::create($data);

        if ($email) {
            Mail::to($email)->send(new MahasiswaBaruDiterima($mahasiswa));
        }

        Log::info("Mahasiswa baru terdaftar: {$mahasiswa->name}");

        Cache::forget('statistik_mahasiswa_per_jurusan');

        return $mahasiswa;
    }
}
```

Controller menjadi jauh lebih ringkas:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreMahasiswaRequest;
use App\Services\MahasiswaService;

class MahasiswaController extends Controller
{
    public function __construct(
        protected MahasiswaService $mahasiswaService,
    ) {}

    public function store(StoreMahasiswaRequest $request)
    {
        $this->mahasiswaService->daftarkan($request->validated(), $request->input('email'));

        return redirect()->route('mahasiswa.index')
            ->with('success', 'Data mahasiswa berhasil disimpan!');
    }
}
```

## 3. Dependency Injection — Kenapa `MahasiswaService` Muncul di Constructor?

Laravel punya **Service Container** yang otomatis membuatkan object dependency yang dibutuhkan sebuah class — kamu cukup "minta" lewat type-hint di constructor, Laravel yang menyediakan objectnya.

```php
public function __construct(
    protected MahasiswaService $mahasiswaService,
) {}
```

Ini disebut **constructor injection**. Manfaatnya:
1. Controller tidak perlu tahu **bagaimana** cara membuat `MahasiswaService` (`new MahasiswaService()`), cukup deklarasikan butuh apa.
2. Saat testing, `MahasiswaService` bisa diganti dengan **mock/fake** tanpa mengubah kode Controller.

## 4. Kapan Perlu Service, Kapan Tidak?

| Situasi | Rekomendasi |
|---|---|
| Cuma `Model::all()` lalu tampilkan ke view | Cukup di Controller langsung, tidak perlu Service |
| Ada logic bisnis: hitung, validasi silang antar tabel, kirim notifikasi, update banyak tabel sekaligus | Pindahkan ke Service |
| Logic yang sama dipakai di 2+ tempat (Controller web, API Controller, Artisan Command) | **Wajib** Service, supaya tidak duplikasi |

> Jangan bikin Service untuk semua hal secara membabi buta — kalau method Controller cuma 3 baris tanpa logic bisnis, biarkan saja di Controller. Service ada untuk mengatasi kompleksitas, bukan untuk formalitas.

Contoh nyata: proyek Laravel produksi biasanya punya `app/Services/ImageOptimizationService.php` — logic konversi & optimasi gambar dipisah dari Controller supaya bisa dipakai ulang di beberapa modul tanpa duplikasi kode. Prinsipnya persis sama dengan `MahasiswaService` di atas.

## Studi Mini: `MataKuliahService`

```php
<?php

namespace App\Services;

use App\Models\MataKuliah;
use Illuminate\Support\Collection;

class MataKuliahService
{
    public function daftarBebanBerat(): Collection
    {
        return MataKuliah::where('sks', '>=', 3)
            ->orderBy('nama_matakuliah')
            ->get();
    }

    public function buat(array $data): MataKuliah
    {
        return MataKuliah::create($data);
    }

    public function perbarui(MataKuliah $mataKuliah, array $data): MataKuliah
    {
        $mataKuliah->update($data);
        return $mataKuliah->fresh();
    }
}
```

```php
class MataKuliahController extends Controller
{
    public function __construct(
        protected MataKuliahService $mataKuliahService,
    ) {}

    public function index()
    {
        $matakuliah = $this->mataKuliahService->daftarBebanBerat();
        return view('matakuliah.index', compact('matakuliah'));
    }

    public function store(StoreMataKuliahRequest $request)
    {
        $this->mataKuliahService->buat($request->validated());
        return redirect()->route('matakuliah.index');
    }
}
```

## Latihan

1. Buat `MahasiswaService` seperti contoh di atas, gunakan di `MahasiswaController` dari modul 07-09.
2. Tambahkan method `cariBerdasarkanJurusan(string $jurusan)` di `MahasiswaService`.
3. Refactor: pindahkan pengecekan `unique:mst_mahasiswa,stambuk` dari modul 09 supaya konsisten dipakai baik dari web maupun nanti dari API (modul 14) — bandingkan mana yang lebih tepat: validasi di Form Request atau pengecekan bisnis di Service.

---
⬅️ [09. Request & Validation](../09-request-validation/README.md) | ➡️ Lanjut ke [11. Response & View/Blade](../11-response-view-blade/README.md)
