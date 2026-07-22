# 09. Request & Validation

Data yang masuk dari luar (form, API) **tidak boleh pernah dipercaya begitu saja**. Modul ini membahas `Request` object dan **Form Request class** — cara Laravel memvalidasi data secara terpusat dan rapi.

## Tujuan Belajar

- Paham cara mengambil data dari `Request` di Controller.
- Paham validasi inline vs **Form Request class** (dan kenapa yang kedua lebih baik untuk proyek yang berkembang).
- Paham aturan validasi umum dan pesan error custom.
- Paham `authorize()` untuk otorisasi di level request.

## 1. Mengambil Data dari Request

```php
use Illuminate\Http\Request;

public function store(Request $request)
{
    $name = $request->input('name');   // cara eksplisit
    $name = $request->name;            // cara magic property (sama hasilnya)
    $semua = $request->all();          // semua input sebagai array

    $hanyaBeberapa = $request->only(['stambuk', 'name']); // pilih kolom tertentu
    $kecuali = $request->except(['_token']);                // semua kecuali kolom tertentu
}
```

## 2. Validasi Inline (Cara Sederhana)

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'stambuk' => 'required|string|max:11',
        'name' => 'required|string|max:255',
        'jurusan' => 'required|string|max:255',
    ]);

    Mahasiswa::create($validated);

    return redirect()->route('mahasiswa.index');
}
```

Kalau validasi gagal, Laravel **otomatis** redirect balik dengan pesan error (tersedia di Blade lewat `$errors`) — tidak perlu `if/else` manual.

Ini cukup untuk kasus sederhana, tapi ketika aturan validasi makin kompleks (banyak field, custom rule, pesan error custom, otorisasi), Controller jadi gemuk. Solusinya: **Form Request class**.

## 3. Form Request Class — Cara yang Direkomendasikan

```bash
php artisan make:request StoreMahasiswaRequest
```

Hasil di `app/Http/Requests/StoreMahasiswaRequest.php`:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreMahasiswaRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // atau cek permission, misal: return $this->user()->can('create', Mahasiswa::class);
    }

    public function rules(): array
    {
        return [
            'stambuk' => 'required|string|max:11|unique:mst_mahasiswa,stambuk',
            'name' => 'required|string|max:255',
            'jurusan' => 'required|string|max:255',
        ];
    }

    public function messages(): array
    {
        return [
            'stambuk.required' => 'Stambuk wajib diisi.',
            'stambuk.unique' => 'Stambuk ini sudah terdaftar.',
        ];
    }
}
```

Memakainya di Controller — perhatikan betapa **bersihnya** Controller sekarang:

```php
use App\Http\Requests\StoreMahasiswaRequest;

public function store(StoreMahasiswaRequest $request)
{
    // Kalau kode sampai baris ini, artinya validasi SUDAH LOLOS otomatis
    Mahasiswa::create($request->validated());

    return redirect()->route('mahasiswa.index');
}
```

Laravel otomatis:
1. Menjalankan `authorize()` — kalau `false`, response `403 Forbidden`.
2. Menjalankan `rules()` — kalau gagal, redirect balik + pesan error, Controller **tidak pernah dijalankan**.
3. Kalau lolos keduanya, baru masuk ke body method `store()`.

> Kenapa ini penting untuk struktur "full" yang diminta di studi kasus (modul 12): memisahkan validasi ke class sendiri membuat Controller **hanya** berisi alur (orkestrasi), bukan detail aturan. Ini jadi kebiasaan penting sejak awal belajar.

## 4. Aturan Validasi yang Sering Dipakai

| Rule | Arti |
|---|---|
| `required` | Wajib diisi |
| `nullable` | Boleh kosong |
| `string` / `integer` / `boolean` / `numeric` | Tipe data |
| `max:255` / `min:3` | Batas panjang/nilai |
| `email` | Format email valid |
| `unique:mst_mahasiswa,stambuk` | Harus unik di tabel `mst_mahasiswa`, kolom `stambuk` |
| `exists:mst_matakuliah,id` | Harus ada di tabel `mst_matakuliah` (dipakai untuk foreign key) |
| `date` | Format tanggal valid |
| `in:Teknik Informatika,Sistem Informasi,Ilmu Komputer` | Harus salah satu dari daftar nilai |
| `confirmed` | Harus ada field `_confirmation` yang sama (misal `password_confirmation`) |

Contoh gabungan untuk request tambah mata kuliah:

```php
public function rules(): array
{
    return [
        'kode' => 'required|string|max:10|unique:mst_matakuliah,kode',
        'nama_matakuliah' => 'required|string|max:255',
        'sks' => 'required|integer|min:1|max:6',
    ];
}
```

## 5. Custom Rule

Untuk aturan yang tidak tersedia bawaan, contoh: memastikan kode mata kuliah mengikuti format standar (2 huruf + 3 angka, misal `IF101`).

```bash
php artisan make:rule FormatKodeMatakuliah
```

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\ValidationRule;
use Closure;

class FormatKodeMatakuliah implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (! preg_match('/^[A-Z]{2}[0-9]{3}$/', $value)) {
            $fail('Kode mata kuliah harus berformat 2 huruf + 3 angka, contoh: IF101.');
        }
    }
}
```

```php
// Dipakai di rules()
'kode' => ['required', new FormatKodeMatakuliah],
```

## Studi Mini: Request untuk Update

```bash
php artisan make:request UpdateMahasiswaRequest
```

```php
class UpdateMahasiswaRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'stambuk' => 'sometimes|required|string|max:11|unique:mst_mahasiswa,stambuk,' . $this->mahasiswa->id,
            'name' => 'sometimes|required|string|max:255',
            'jurusan' => 'sometimes|required|string|max:255',
        ];
    }
}
```

`sometimes` berarti aturan hanya dicek **kalau field itu ada** di request — berguna untuk update parsial (PATCH) di mana user tidak wajib kirim semua field. Perhatikan juga `unique:mst_mahasiswa,stambuk,' . $this->mahasiswa->id` — ini mengecualikan record yang sedang diedit sendiri dari pengecekan unik (kalau tidak, Laravel akan menganggap stambuk-nya sendiri sebagai "duplikat").

## Latihan

1. Buat `StoreMahasiswaRequest` dan `UpdateMahasiswaRequest` untuk model `Mahasiswa` dari modul 08.
2. Tambahkan custom message untuk minimal 2 rule.
3. Buat custom rule `FormatNip` yang memastikan NIP dosen hanya berisi digit dan panjangnya tepat 18 karakter, terapkan di `StoreDosenRequest`.

---
⬅️ [08. Model & Migration](../08-model-migration-database/README.md) | ➡️ Lanjut ke [10. Service Layer](../10-service-layer/README.md)
