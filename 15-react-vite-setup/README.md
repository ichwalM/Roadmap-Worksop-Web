# 15. Setup React + Vite

React adalah **library** JavaScript (bukan framework penuh — ingat konsep ini dari modul 03) untuk membangun antarmuka pengguna (UI) berbasis komponen. Modul ini fokus instalasi dan mengenal struktur proyek React yang dibuat dengan **Vite** — build tool modern yang jauh lebih cepat dari alat lama (Create React App).

## Tujuan Belajar

- Paham kenapa React disebut library, bukan framework.
- Berhasil membuat proyek React baru dengan Vite.
- Paham struktur folder proyek Vite + React.
- Paham dasar JSX.

## 1. React: Library, Bukan Framework

Ingat tabel di modul 03: React tidak memaksa struktur folder tertentu, tidak menyediakan routing/HTTP client bawaan — kamu **pilih sendiri** library pendukung (React Router untuk routing, axios/fetch untuk HTTP, dst). Ini beda dengan Laravel yang sudah menyediakan semuanya "dalam satu paket" (routing, ORM, validasi) dengan struktur folder yang ditentukan.

| | Laravel (Framework) | React (Library) |
|---|---|---|
| Routing | Bawaan (`routes/web.php`) | Pilih sendiri (React Router, dll) |
| HTTP Client | Bawaan (Eloquent ke DB, `Http::` facade ke API luar) | Pilih sendiri (`fetch`, axios) |
| Struktur folder | Ditentukan framework | Bebas, tim yang menentukan konvensi |
| State management | Tidak relevan (server-side) | Bawaan dasar (`useState`), atau pilih library (Redux, Zustand) untuk kasus kompleks |

## 2. Instalasi Proyek React + Vite

```bash
npm create vite@latest mahasiswa-frontend -- --template react
cd mahasiswa-frontend
npm install
npm run dev
```

Buka URL yang muncul di terminal (biasanya `http://localhost:5173`).

## 3. Struktur Folder Proyek Vite + React

```
mahasiswa-frontend/
├── node_modules/       # Dependency terinstal (jangan commit ke Git)
├── public/             # Asset statis yang di-copy apa adanya (favicon, robots.txt)
├── src/                # Kode aplikasi
│   ├── assets/         # Gambar, font yang di-import dan diproses build tool
│   ├── App.jsx         # Komponen root aplikasi
│   ├── App.css
│   ├── main.jsx        # Entry point — titik awal render React ke halaman
│   └── index.css
├── index.html          # Satu-satunya file HTML (SPA — Single Page Application)
├── package.json        # Daftar dependency & script npm
├── vite.config.js       # Konfigurasi Vite
└── .gitignore
```

> Pembahasan lebih dalam tentang folder ini (`src/components`, `src/pages`, `src/hooks`, dst yang biasa ditambahkan seiring proyek membesar) ada di [modul 19 — Arsitektur Folder](../19-arsitektur-folder-laravel-react/README.md).

### `index.html` — Cuma 1 File HTML untuk Semua Halaman

```html
<!doctype html>
<html lang="id">
  <head>
    <meta charset="UTF-8" />
    <title>SIAKAD Frontend</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

`<div id="root"></div>` adalah **satu-satunya elemen** di HTML — semua UI nanti "disuntikkan" React ke dalam div ini lewat JavaScript. Inilah kenapa React disebut membangun **SPA (Single Page Application)**.

### `main.jsx` — Entry Point

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

`createRoot(...).render(<App />)` adalah titik di mana React mulai mengambil alih `<div id="root">` dan merender komponen `<App />` di dalamnya.

### `App.jsx` — Komponen Root

```jsx
function App() {
  return (
    <div>
      <h1>Halo, Sistem Informasi Akademik!</h1>
    </div>
  );
}

export default App;
```

## 4. JSX — HTML di Dalam JavaScript

JSX terlihat seperti HTML, tapi sebenarnya "gula sintaks" yang dikompilasi jadi pemanggilan fungsi JavaScript biasa.

```jsx
function Sapaan() {
  const nama = "Ahmad";
  const jumlahMahasiswa = 3;

  return (
    <div>
      {/* Komentar di JSX pakai kurung kurawal */}
      <h2>Halo, {nama}!</h2>
      <p>Ada {jumlahMahasiswa} mahasiswa terdaftar.</p>

      {jumlahMahasiswa > 0 ? (
        <p>Silakan pilih mahasiswa untuk lihat detail.</p>
      ) : (
        <p>Belum ada mahasiswa.</p>
      )}
    </div>
  );
}
```

### Aturan Penting JSX

| Aturan | Contoh |
|---|---|
| `{ }` untuk menyisipkan ekspresi JavaScript | `<p>{nama}</p>` |
| `class` → `className` (karena `class` reserved word di JS) | `<div className="card">` |
| Tag harus selalu ditutup, termasuk tag "kosong" | `<img />`, `<br />` |
| Harus ada **satu** elemen pembungkus (atau fragment `<>...</>`) | `<><h1>A</h1><p>B</p></>` |
| Atribut pakai camelCase | `onClick`, `onChange`, bukan `onclick` |

```jsx
// SALAH — 2 elemen root tanpa pembungkus
return (
  <h1>Judul</h1>
  <p>Isi</p>
);

// BENAR — dibungkus fragment <>...</>
return (
  <>
    <h1>Judul</h1>
    <p>Isi</p>
  </>
);
```

## 5. Menjalankan & Build

```bash
npm run dev      # jalankan dev server (hot reload otomatis saat file disimpan)
npm run build    # build untuk produksi, hasil di folder dist/
npm run preview  # jalankan hasil build secara lokal untuk uji coba sebelum deploy
```

## Studi Mini: Halaman Statis Daftar Mahasiswa

```jsx
// src/App.jsx
function App() {
  const mahasiswa = [
    { id: 1, stambuk: "20210011", name: "Ahmad Fauzi", jurusan: "Teknik Informatika" },
    { id: 2, stambuk: "20210012", name: "Siti Rahma", jurusan: "Sistem Informasi" },
    { id: 3, stambuk: "20210013", name: "Budi Santoso", jurusan: "Teknik Informatika" },
  ];

  return (
    <div style={{ padding: 24 }}>
      <h1>Daftar Mahasiswa</h1>
      <ul>
        {mahasiswa.map((m) => (
          <li key={m.id}>
            {m.stambuk} — {m.name} ({m.jurusan})
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

> Perhatikan `.map()` dipakai lagi persis seperti di modul 04 (JavaScript dasar) — dan **wajib** ada prop `key` unik di setiap item list (React memakainya untuk melacak perubahan tiap elemen secara efisien).

## Latihan

1. Buat proyek React + Vite baru bernama `mahasiswa-frontend`.
2. Ganti isi `App.jsx` dengan daftar mahasiswa statis seperti contoh di atas (data hardcoded dulu, belum fetch dari API — itu di modul 17).
3. Coba ubah salah satu teks di `App.jsx`, simpan file, amati hot-reload otomatis di browser tanpa refresh manual.
4. Tambahkan kondisi: kalau `jurusan === "Teknik Informatika"`, tampilkan teks dengan warna biru (pakai `style={{ color: 'blue' }}` inline dulu).

---
⬅️ [14. Setup API di Laravel](../14-setup-api-laravel/README.md) | ➡️ Lanjut ke [16. Komponen, Props, State](../16-react-komponen-props-state/README.md)
