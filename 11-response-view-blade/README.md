# 11. Response & View/Blade

Setelah Controller memproses request (dibantu Request & Service), langkah terakhir adalah **mengembalikan response** ke browser — bisa berupa halaman HTML (Blade), redirect, atau JSON.

## Tujuan Belajar

- Paham Blade templating: variabel, kondisi, perulangan, layout/komponen.
- Paham berbagai jenis Response: view, redirect, JSON, download.
- Paham flash message untuk feedback ke user setelah aksi (create/update/delete).

## 1. Blade — Templating Engine Laravel

File Blade berakhiran `.blade.php`, ditaruh di `resources/views/`.

```php
// Controller
public function index()
{
    $mahasiswa = Mahasiswa::all();
    return view('mahasiswa.index', compact('mahasiswa'));
    // setara dengan: view('mahasiswa.index', ['mahasiswa' => $mahasiswa])
}
```

```blade
{{-- resources/views/mahasiswa/index.blade.php --}}
<h1>Daftar Mahasiswa</h1>

<ul>
    @foreach ($mahasiswa as $m)
        <li>
            {{ $m->stambuk }} — {{ $m->name }} ({{ $m->jurusan }})
            @if ($m->jurusan === 'Teknik Informatika')
                <span class="text-blue-600">(TI)</span>
            @endif
        </li>
    @endforeach
</ul>

@if ($mahasiswa->isEmpty())
    <p>Belum ada mahasiswa terdaftar.</p>
@endif
```

### Sintaks Blade Penting

| Sintaks | Fungsi |
|---|---|
| `{{ $variabel }}` | Cetak variabel (otomatis di-escape, aman dari XSS) |
| `{!! $html !!}` | Cetak HTML mentah (hati-hati, hanya untuk data terpercaya) |
| `@if / @elseif / @else / @endif` | Kondisi |
| `@foreach / @endforeach` | Perulangan |
| `@csrf` | Token keamanan wajib di setiap `<form method="POST">` |
| `@method('PUT')` | Karena HTML form hanya support GET/POST, ini "menyamarkan" method PUT/DELETE |
| `{{-- komentar --}}` | Komentar (tidak tampil di HTML output) |

## 2. Layout & Komponen — Hindari Duplikasi HTML

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>@yield('title', 'SIAKAD Mini')</title>
</head>
<body>
    <header>@include('partials.navbar')</header>

    <main>
        @yield('content')
    </main>

    <footer>&copy; 2026 SIAKAD Mini</footer>
</body>
</html>
```

```blade
{{-- resources/views/mahasiswa/index.blade.php --}}
@extends('layouts.app')

@section('title', 'Daftar Mahasiswa')

@section('content')
    <h1>Daftar Mahasiswa</h1>
    @foreach ($mahasiswa as $m)
        <p>{{ $m->name }}</p>
    @endforeach
@endsection
```

Blade Component (lebih modern, mirip komponen React yang akan dipelajari di modul 16 — konsep "potong UI jadi bagian yang bisa dipakai ulang" ini **sama** di kedua dunia):

```bash
php artisan make:component MahasiswaCard
```

```blade
{{-- resources/views/components/mahasiswa-card.blade.php --}}
<div class="border rounded-lg p-4">
    <h3 class="font-bold">{{ $name }}</h3>
    <p>Stambuk: {{ $stambuk }} · {{ $jurusan }}</p>
    {{ $slot }} {{-- konten tambahan dari pemanggil --}}
</div>
```

```blade
{{-- Dipakai di halaman lain --}}
<x-mahasiswa-card :stambuk="$m->stambuk" :name="$m->name" :jurusan="$m->jurusan">
    <a href="{{ route('mahasiswa.show', $m) }}">Lihat Detail</a>
</x-mahasiswa-card>
```

## 3. Jenis-Jenis Response

```php
// 1. View (HTML)
return view('mahasiswa.index', compact('mahasiswa'));

// 2. Redirect
return redirect()->route('mahasiswa.index');
return redirect('/mahasiswa');
return back(); // kembali ke halaman sebelumnya

// 3. Redirect dengan flash message
return redirect()->route('mahasiswa.index')
    ->with('success', 'Data mahasiswa berhasil ditambahkan!');

// 4. JSON (dipakai untuk API — dibahas detail di modul 14)
return response()->json(['message' => 'Berhasil', 'data' => $mahasiswa]);

// 5. Response dengan status code custom
return response()->json(['error' => 'Tidak ditemukan'], 404);

// 6. Download file
return response()->download(storage_path('app/laporan-mahasiswa.pdf'));
```

## 4. Flash Message — Feedback ke User

```php
// Controller
return redirect()->route('mahasiswa.index')
    ->with('success', 'Data mahasiswa berhasil disimpan!');
```

```blade
{{-- Tampilkan di layout, biasanya di atas <main> --}}
@if (session('success'))
    <div class="bg-green-100 text-green-800 p-4 rounded mb-4">
        {{ session('success') }}
    </div>
@endif

@if ($errors->any())
    <div class="bg-red-100 text-red-800 p-4 rounded mb-4">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

`session('success')` bekerja karena Laravel menyimpan flash data **satu kali pakai** — otomatis hilang setelah request berikutnya, cocok untuk pesan "berhasil disimpan" yang hanya perlu tampil sekali.

## Studi Mini: Halaman Index + Create dengan Feedback

```php
// Controller
public function store(StoreMahasiswaRequest $request, MahasiswaService $service)
{
    $service->daftarkan($request->validated());

    return redirect()->route('mahasiswa.index')
        ->with('success', 'Data mahasiswa baru berhasil ditambahkan!');
}
```

```blade
{{-- resources/views/mahasiswa/create.blade.php --}}
@extends('layouts.app')

@section('content')
    <h1>Tambah Mahasiswa</h1>

    <form method="POST" action="{{ route('mahasiswa.store') }}">
        @csrf
        <input type="text" name="stambuk" value="{{ old('stambuk') }}" placeholder="Stambuk">
        @error('stambuk')
            <p class="text-red-600">{{ $message }}</p>
        @enderror

        <input type="text" name="name" value="{{ old('name') }}" placeholder="Nama Lengkap">
        @error('name')
            <p class="text-red-600">{{ $message }}</p>
        @enderror

        <input type="text" name="jurusan" value="{{ old('jurusan') }}" placeholder="Jurusan">

        <button type="submit">Simpan</button>
    </form>
@endsection
```

`old('stambuk')` mengisi kembali input dengan nilai sebelumnya kalau validasi gagal — supaya user tidak perlu ketik ulang semua dari nol.

## Latihan

1. Buat layout `layouts/app.blade.php` dengan navbar dan area flash message.
2. Buat view `mahasiswa/index.blade.php` dan `mahasiswa/create.blade.php` yang `@extends` layout tersebut.
3. Tambahkan flash message "Berhasil ditambahkan" setelah `store()`, dan tampilkan validation error dengan `@error`.
4. Buat 1 Blade Component `<x-mahasiswa-card>` dan pakai di `index.blade.php` lewat `@foreach`.

---
⬅️ [10. Service Layer](../10-service-layer/README.md) | ➡️ Lanjut ke [12. Studi Kasus: CRUD Lengkap](../12-studi-kasus-crud-mvc-service/README.md)
