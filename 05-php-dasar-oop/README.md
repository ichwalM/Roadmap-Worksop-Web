# 05. PHP Dasar & OOP

Laravel ditulis dengan **PHP**, dan seluruh strukturnya (Model, Controller, Service, dll) memakai konsep **Object-Oriented Programming (OOP)**. Modul ini adalah fondasi wajib sebelum masuk ke Laravel (modul 06+).

## Tujuan Belajar

- Paham sintaks dasar PHP: variabel, fungsi, array, kondisi, perulangan.
- Paham konsep OOP: class, object, property, method, constructor.
- Paham `interface`, dan konsep `namespace` — dua hal yang sangat sering muncul di kode Laravel.

## 1. Sintaks Dasar PHP

```php
<?php

$nama = "Ahmad Fauzi";     // variabel diawali $
$sks = 3;
$aktif = true;

echo "Halo, $nama!";       // bisa langsung interpolasi dalam string
echo "Halo, " . $nama . "!"; // atau pakai concatenation (.)

function sapa($nama) {
    return "Halo, $nama!";
}

echo sapa("Budi");
```

### Array

```php
<?php

// Array biasa (list)
$jurusan = ["Teknik Informatika", "Sistem Informasi", "Ilmu Komputer"];
echo $jurusan[0]; // "Teknik Informatika"

// Associative array (mirip object di JS)
$mahasiswa = [
    "stambuk" => "20210011",
    "name" => "Ahmad Fauzi",
    "jurusan" => "Teknik Informatika",
];
echo $mahasiswa["name"]; // "Ahmad Fauzi"

// Array of associative array — pola paling sering dipakai
$daftarMahasiswa = [
    ["name" => "Ahmad Fauzi", "jurusan" => "Teknik Informatika"],
    ["name" => "Siti Rahma", "jurusan" => "Sistem Informasi"],
];

foreach ($daftarMahasiswa as $m) {
    echo $m["name"] . " - " . $m["jurusan"] . "\n";
}
```

### Kondisi & Perulangan

```php
<?php

$sks = 0;

if ($sks === 0) {
    echo "Belum ada SKS diisi";
} elseif ($sks <= 12) {
    echo "Beban ringan";
} else {
    echo "Beban penuh";
}

for ($i = 0; $i < 3; $i++) {
    echo "Iterasi ke-$i\n";
}
```

## 2. OOP — Class & Object

Class adalah **cetakan/blueprint**, object adalah **hasil dari cetakan itu**.

```php
<?php

class Mahasiswa
{
    // Property (data yang dimiliki object)
    public string $name;
    public string $jurusan;
    public int $totalSks = 0;

    // Constructor — dijalankan otomatis saat object dibuat
    public function __construct(string $name, string $jurusan)
    {
        $this->name = $name;
        $this->jurusan = $jurusan;
    }

    // Method (perilaku/fungsi milik object)
    public function sudahBolehSkripsi(): bool
    {
        return $this->totalSks >= 120;
    }

    public function tambahSks(int $jumlah): void
    {
        $this->totalSks += $jumlah;
    }
}

// Membuat object dari class
$ahmad = new Mahasiswa("Ahmad Fauzi", "Teknik Informatika");

echo $ahmad->name;                                   // "Ahmad Fauzi"
echo $ahmad->sudahBolehSkripsi() ? "ya" : "tidak";    // "tidak"

$ahmad->tambahSks(130);
echo $ahmad->sudahBolehSkripsi() ? "ya" : "tidak";    // "ya"
```

### Kenapa OOP Penting untuk Laravel?

Setiap **Model**, **Controller**, **Request**, dan **Service** di Laravel adalah **class**. Contoh nyata dari Laravel:

```php
class MahasiswaController extends Controller
{
    public function index()
    {
        // method di dalam class controller
    }
}
```

`extends Controller` berarti `MahasiswaController` **mewarisi (inheritance)** semua kemampuan dari class `Controller` bawaan Laravel — konsep OOP yang disebut **inheritance**.

## 3. Interface — Kontrak Perilaku

Interface mendefinisikan **method apa saja yang wajib ada**, tanpa menentukan bagaimana implementasinya.

```php
<?php

interface EksporInterface
{
    public function ekspor(array $data): string;
}

class EksporCsv implements EksporInterface
{
    public function ekspor(array $data): string
    {
        return "Data mahasiswa diekspor sebagai CSV";
    }
}

class EksporJson implements EksporInterface
{
    public function ekspor(array $data): string
    {
        return json_encode($data);
    }
}
```

Manfaatnya: kode yang memakai `EksporInterface` tidak peduli implementasi mana yang dipakai — bisa CSV, JSON, atau nanti Excel, tanpa mengubah kode pemanggil. Ini prinsip dasar yang membuat **Service Layer** (modul 10) bisa fleksibel dan mudah di-testing.

## 4. Namespace & Autoloading

```php
<?php

namespace App\Models;

class Mahasiswa
{
    // ...
}
```

`namespace` seperti "alamat folder" untuk class — mencegah tabrakan nama antar class yang mirip. Di Laravel, namespace **selalu mengikuti struktur folder**:

```
app/Models/Mahasiswa.php                    → namespace App\Models;
app/Http/Controllers/MahasiswaController.php → namespace App\Http\Controllers;
```

Untuk memakai class dari namespace lain, kita `use`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Mahasiswa; // import class

class MahasiswaController extends Controller
{
    public function index()
    {
        $mahasiswa = Mahasiswa::all(); // langsung bisa dipakai
    }
}
```

## Studi Mini: Class `Mahasiswa` dengan Koleksi

```php
<?php

class Mahasiswa
{
    public function __construct(
        public string $name,
        public string $jurusan,
        public int $totalSks,
    ) {}

    public function sudahBolehSkripsi(): bool
    {
        return $this->totalSks >= 120;
    }
}

$daftar = [
    new Mahasiswa("Ahmad Fauzi", "Teknik Informatika", 130),
    new Mahasiswa("Siti Rahma", "Sistem Informasi", 90),
    new Mahasiswa("Budi Santoso", "Teknik Informatika", 125),
];

// array_filter mirip .filter() di JavaScript
$bolehSkripsi = array_filter($daftar, fn($m) => $m->sudahBolehSkripsi());

foreach ($bolehSkripsi as $m) {
    echo $m->name . "\n";
}
```

Perhatikan: `array_filter` + `fn()` (arrow function PHP) **konsepnya identik** dengan `.filter()` + arrow function di JavaScript (modul 04) — begitu satu bahasa dikuasai, bahasa lain jadi lebih mudah karena polanya mirip.

## Latihan

1. Buat class `MataKuliah` dengan property `kode`, `namaMatakuliah`, `sks`, dan method `bebanBerat(): bool` (kembalikan `true` kalau `sks >= 3`).
2. Buat array berisi 5 object `MataKuliah`, lalu gunakan `array_filter` untuk menampilkan hanya yang `bebanBerat()`-nya `true`.
3. Buat interface `EksporInterface` (seperti contoh di atas) dan pakai untuk mengekspor daftar `Dosen` (property: `nip`, `name`) ke 2 format berbeda.

---
⬅️ [04. JavaScript Dasar](../04-javascript-dasar/README.md) | ➡️ Lanjut ke [06. Instalasi & Struktur Laravel](../06-instalasi-struktur-laravel/README.md)
