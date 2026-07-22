# 16. Komponen, Props, State

Inti dari React adalah memecah UI menjadi **komponen** kecil yang bisa dipakai ulang — mirip konsep Blade Component di modul 11, tapi jauh lebih hidup karena bisa punya **state** (data yang berubah dan otomatis me-render ulang tampilan).

## Tujuan Belajar

- Paham apa itu komponen dan cara membuatnya.
- Paham **props** — cara mengirim data dari komponen induk ke anak.
- Paham **state** (`useState`) — data yang bisa berubah dan memicu re-render.
- Paham **event handling** di React.
- Paham `useEffect` untuk menjalankan efek samping (side effect).

## 1. Komponen — Blok Bangunan UI

Komponen adalah fungsi JavaScript yang mengembalikan JSX.

```jsx
// src/components/MahasiswaCard.jsx
function MahasiswaCard() {
  return (
    <div className="border rounded-lg p-4">
      <h3 className="font-bold">Ahmad Fauzi</h3>
      <p>Stambuk: 20210011</p>
    </div>
  );
}

export default MahasiswaCard;
```

```jsx
// src/App.jsx
import MahasiswaCard from './components/MahasiswaCard';

function App() {
  return (
    <div>
      <h1>Daftar Mahasiswa</h1>
      <MahasiswaCard />
      <MahasiswaCard />
    </div>
  );
}
```

Masalahnya: kedua `<MahasiswaCard />` di atas menampilkan data **yang sama persis** ("Ahmad Fauzi", stambuk 20210011) — karena datanya hardcoded di dalam komponen. Solusinya: **props**.

## 2. Props — Mengirim Data ke Komponen Anak

Props (properties) adalah cara komponen induk "mengirim" data ke komponen anak — mirip parameter fungsi.

```jsx
// src/components/MahasiswaCard.jsx
function MahasiswaCard({ stambuk, name, jurusan }) {
  return (
    <div className="border rounded-lg p-4">
      <h3 className="font-bold">{name}</h3>
      <p>Stambuk: {stambuk}</p>
      <p>{jurusan}</p>
    </div>
  );
}

export default MahasiswaCard;
```

```jsx
// src/App.jsx
import MahasiswaCard from './components/MahasiswaCard';

function App() {
  return (
    <div>
      <h1>Daftar Mahasiswa</h1>
      <MahasiswaCard stambuk="20210011" name="Ahmad Fauzi" jurusan="Teknik Informatika" />
      <MahasiswaCard stambuk="20210012" name="Siti Rahma" jurusan="Sistem Informasi" />
      <MahasiswaCard stambuk="20210013" name="Budi Santoso" jurusan="Teknik Informatika" />
    </div>
  );
}
```

**Props bersifat read-only** — komponen anak **tidak boleh** mengubah props yang diterima. Kalau data perlu berubah, itu tugas **state**.

### Kombinasi Props + `.map()` — Pola Paling Sering Dipakai

```jsx
function App() {
  const mahasiswa = [
    { id: 1, stambuk: "20210011", name: "Ahmad Fauzi", jurusan: "Teknik Informatika" },
    { id: 2, stambuk: "20210012", name: "Siti Rahma", jurusan: "Sistem Informasi" },
    { id: 3, stambuk: "20210013", name: "Budi Santoso", jurusan: "Teknik Informatika" },
  ];

  return (
    <div>
      <h1>Daftar Mahasiswa</h1>
      {mahasiswa.map((m) => (
        <MahasiswaCard key={m.id} stambuk={m.stambuk} name={m.name} jurusan={m.jurusan} />
      ))}
    </div>
  );
}
```

Cara singkat mengirim semua properti object sebagai props sekaligus (spread — ingat modul 04):
```jsx
{mahasiswa.map((m) => (
  <MahasiswaCard key={m.id} {...m} />
))}
```

## 3. State — Data yang Bisa Berubah

`useState` adalah **Hook** (fungsi khusus React yang diawali `use`) untuk menyimpan data yang bisa berubah di dalam komponen, dan **otomatis me-render ulang** tampilan saat data itu berubah.

```jsx
import { useState } from 'react';

function Counter() {
  const [jumlah, setJumlah] = useState(0); // nilai awal 0

  return (
    <div>
      <p>Jumlah mahasiswa terdaftar: {jumlah}</p>
      <button onClick={() => setJumlah(jumlah + 1)}>Tambah</button>
      <button onClick={() => setJumlah(0)}>Reset</button>
    </div>
  );
}
```

Penjelasan `useState(0)`:
- Mengembalikan **array 2 elemen**: `jumlah` (nilai saat ini) dan `setJumlah` (fungsi untuk mengubah nilai).
- Memanggil `setJumlah(...)` **memberitahu React** untuk render ulang komponen dengan nilai baru — tidak seperti variabel biasa yang diubah langsung.

### State dengan Object/Array

```jsx
import { useState } from 'react';

function FormMahasiswa() {
  const [form, setForm] = useState({ stambuk: '', name: '', jurusan: '' });

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value }); // spread + update 1 field
  };

  return (
    <form>
      <input name="stambuk" value={form.stambuk} onChange={handleChange} placeholder="Stambuk" />
      <input name="name" value={form.name} onChange={handleChange} placeholder="Nama" />
      <p>Preview: {form.name} ({form.stambuk}) — {form.jurusan}</p>
    </form>
  );
}
```

`{ ...form, [e.target.name]: e.target.value }` — pola spread ini **selalu membuat object baru**, tidak pernah mengubah state lama secara langsung (disebut *immutability*) — ini aturan penting di React supaya perubahan bisa terdeteksi dengan benar.

## 4. Event Handling

```jsx
function Tombol() {
  const handleClick = () => {
    alert('Tombol diklik!');
  };

  return <button onClick={handleClick}>Klik Saya</button>;
}
```

| Event HTML biasa | Event React |
|---|---|
| `onclick` | `onClick` |
| `onchange` | `onChange` |
| `onsubmit` | `onSubmit` |

```jsx
function FormSederhana() {
  const handleSubmit = (e) => {
    e.preventDefault(); // cegah reload halaman (default behavior HTML form)
    console.log('Form dikirim!');
  };

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Kirim</button>
    </form>
  );
}
```

## 5. `useEffect` — Menjalankan Efek Samping

`useEffect` menjalankan kode **setelah** komponen selesai render — dipakai untuk hal-hal "di luar" render murni: fetch data (modul 17), set judul halaman, subscribe event, dll.

```jsx
import { useState, useEffect } from 'react';

function JudulHalaman({ judul }) {
  useEffect(() => {
    document.title = judul; // efek samping: mengubah tab browser
  }, [judul]); // dijalankan ulang HANYA kalau `judul` berubah

  return <h1>{judul}</h1>;
}
```

Dependency array `[judul]` menentukan **kapan** effect dijalankan ulang:
```jsx
useEffect(() => { ... }, []);        // hanya sekali, saat komponen pertama kali muncul
useEffect(() => { ... }, [judul]);   // setiap kali `judul` berubah
useEffect(() => { ... });            // setiap kali komponen render ulang (jarang dipakai, hati-hati)
```

## Studi Mini: Daftar Mahasiswa Interaktif (Filter Jurusan)

```jsx
import { useState } from 'react';
import MahasiswaCard from './components/MahasiswaCard';

function App() {
  const [mahasiswa] = useState([
    { id: 1, stambuk: "20210011", name: "Ahmad Fauzi", jurusan: "Teknik Informatika" },
    { id: 2, stambuk: "20210012", name: "Siti Rahma", jurusan: "Sistem Informasi" },
    { id: 3, stambuk: "20210013", name: "Budi Santoso", jurusan: "Teknik Informatika" },
  ]);
  const [hanyaTI, setHanyaTI] = useState(false);

  const ditampilkan = hanyaTI
    ? mahasiswa.filter((m) => m.jurusan === "Teknik Informatika")
    : mahasiswa;

  return (
    <div>
      <h1>Daftar Mahasiswa</h1>

      <label>
        <input
          type="checkbox"
          checked={hanyaTI}
          onChange={(e) => setHanyaTI(e.target.checked)}
        />
        Tampilkan jurusan Teknik Informatika saja
      </label>

      {ditampilkan.map((m) => (
        <MahasiswaCard key={m.id} stambuk={m.stambuk} name={m.name} jurusan={m.jurusan} />
      ))}
    </div>
  );
}
```

Perhatikan `.filter()` dipakai lagi (modul 04) — kali ini hasilnya langsung memicu **re-render** UI setiap kali checkbox diklik, tanpa satu baris pun DOM manipulation manual.

## Latihan

1. Buat komponen `MahasiswaCard` yang menerima props `stambuk`, `name`, `jurusan`, dan tombol "Detail".
2. Buat state `jumlahDilihat` dan tombol "👁 Lihat" di tiap card yang menambah counter saat diklik.
3. Buat form tambah mahasiswa sederhana (stambuk, nama, jurusan) yang menampilkan ringkasan data secara real-time saat diketik (seperti contoh `FormMahasiswa` di atas), pakai `onSubmit` untuk menampilkan `alert` ringkasan akhir.

---
⬅️ [15. Setup React + Vite](../15-react-vite-setup/README.md) | ➡️ Lanjut ke [17. Fetch API dari React](../17-react-fetch-api-axios/README.md)
