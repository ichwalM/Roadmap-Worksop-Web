# 17. Fetch API dari React

Sekarang saatnya menyambungkan React ke data **sungguhan** — bukan lagi array hardcoded, tapi data dari API Laravel yang dibangun di modul 14.

## Tujuan Belajar

- Paham `fetch` bawaan browser vs library `axios`.
- Bisa mengambil data dari API saat komponen dimuat (`useEffect` + `useState`).
- Paham pola **loading state** dan **error state** — wajib ada di setiap fetch data produksi.
- Bisa mengirim data (POST/PUT/DELETE) dari form React ke API.
- Paham custom hook untuk fetch yang bisa dipakai ulang.

## 1. `fetch` — Bawaan Browser

```jsx
import { useState, useEffect } from 'react';

function DaftarMahasiswa() {
  const [mahasiswa, setMahasiswa] = useState([]);

  useEffect(() => {
    fetch('http://localhost:8000/api/mahasiswa')
      .then((response) => response.json())
      .then((json) => setMahasiswa(json.data)); // ingat: API Resource membungkus dalam key "data"
  }, []); // array kosong = jalankan sekali saat komponen pertama render

  return (
    <ul>
      {mahasiswa.map((m) => (
        <li key={m.id}>{m.stambuk} — {m.name}</li>
      ))}
    </ul>
  );
}
```

## 2. `axios` — Library Populer, Lebih Praktis

```bash
npm install axios
```

```jsx
import axios from 'axios';
import { useState, useEffect } from 'react';

function DaftarMahasiswa() {
  const [mahasiswa, setMahasiswa] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:8000/api/mahasiswa')
      .then((response) => setMahasiswa(response.data.data));
  }, []);

  return (
    <ul>
      {mahasiswa.map((m) => (
        <li key={m.id}>{m.stambuk} — {m.name}</li>
      ))}
    </ul>
  );
}
```

### `fetch` vs `axios`

| | `fetch` | `axios` |
|---|---|---|
| Instalasi | Bawaan browser, tidak perlu install | `npm install axios` |
| Parse JSON | Manual (`response.json()`) | Otomatis (`response.data`) |
| Error HTTP (4xx/5xx) | **Tidak** otomatis dianggap error (harus cek `response.ok` manual) | **Otomatis** masuk ke `catch` |
| Konfigurasi default (base URL, header) | Manual tiap request | Bisa dibuat instance dengan config bersama |

Contoh menyiapkan instance axios dengan base URL (praktik umum di proyek nyata):

```js
// src/api/axios.js
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8000/api',
  headers: { Accept: 'application/json' },
});

export default api;
```

```jsx
import api from '../api/axios';

useEffect(() => {
  api.get('/mahasiswa').then((res) => setMahasiswa(res.data.data));
}, []);
```

## 3. Loading & Error State — Wajib di Produksi

Tanpa ini, user hanya melihat halaman kosong saat data belum datang, atau blank screen tanpa penjelasan saat API error.

```jsx
import { useState, useEffect } from 'react';
import api from '../api/axios';

function DaftarMahasiswa() {
  const [mahasiswa, setMahasiswa] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    api.get('/mahasiswa')
      .then((response) => {
        setMahasiswa(response.data.data);
      })
      .catch((err) => {
        setError('Gagal memuat data mahasiswa. Coba lagi nanti.');
        console.error(err);
      })
      .finally(() => {
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Memuat data...</p>;
  if (error) return <p style={{ color: 'red' }}>{error}</p>;

  return (
    <ul>
      {mahasiswa.map((m) => (
        <li key={m.id}>{m.stambuk} — {m.name}</li>
      ))}
    </ul>
  );
}
```

Menggunakan `async/await` (gaya yang sama dengan modul 04) sebagai alternatif `.then()`:

```jsx
useEffect(() => {
  const ambilData = async () => {
    try {
      const response = await api.get('/mahasiswa');
      setMahasiswa(response.data.data);
    } catch (err) {
      setError('Gagal memuat data mahasiswa.');
    } finally {
      setLoading(false);
    }
  };

  ambilData();
}, []);
```

## 4. Mengirim Data — POST/PUT/DELETE

```jsx
import { useState } from 'react';
import api from '../api/axios';

function FormTambahMahasiswa({ onBerhasil }) {
  const [form, setForm] = useState({ stambuk: '', name: '', jurusan: '' });
  const [errors, setErrors] = useState({});
  const [submitting, setSubmitting] = useState(false);

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setSubmitting(true);
    setErrors({});

    try {
      const response = await api.post('/mahasiswa', form);
      onBerhasil(response.data.data); // kirim data baru ke komponen induk
      setForm({ stambuk: '', name: '', jurusan: '' }); // reset form
    } catch (err) {
      if (err.response?.status === 422) {
        setErrors(err.response.data.errors); // pesan validasi dari Laravel
      } else {
        setErrors({ umum: ['Terjadi kesalahan, coba lagi.'] });
      }
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="stambuk" value={form.stambuk} onChange={handleChange} placeholder="Stambuk" />
      {errors.stambuk && <p style={{ color: 'red' }}>{errors.stambuk[0]}</p>}

      <input name="name" value={form.name} onChange={handleChange} placeholder="Nama Lengkap" />
      <input name="jurusan" value={form.jurusan} onChange={handleChange} placeholder="Jurusan" />

      <button type="submit" disabled={submitting}>
        {submitting ? 'Menyimpan...' : 'Simpan'}
      </button>
    </form>
  );
}
```

> `err.response.data.errors` berbentuk seperti ini karena Laravel **otomatis** mengembalikan `422 Unprocessable Entity` dengan struktur `{ "message": "...", "errors": { "stambuk": ["Stambuk ini sudah terdaftar."] } }` saat Form Request (modul 09) gagal validasi — struktur ini konsisten baik dipanggil dari Postman maupun dari React.

Menghapus data:
```jsx
const hapusMahasiswa = async (id) => {
  if (!confirm('Yakin hapus?')) return;
  await api.delete(`/mahasiswa/${id}`);
  setMahasiswa(mahasiswa.filter((m) => m.id !== id)); // update state lokal tanpa fetch ulang
};
```

## 5. Custom Hook — Fetch yang Bisa Dipakai Ulang

Kalau pola fetch+loading+error di atas dipakai berulang di banyak komponen, ekstrak jadi **custom hook** (fungsi yang namanya diawali `use`, memakai Hook lain di dalamnya).

```jsx
// src/hooks/useMahasiswa.js
import { useState, useEffect } from 'react';
import api from '../api/axios';

export function useMahasiswa() {
  const [mahasiswa, setMahasiswa] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    api.get('/mahasiswa')
      .then((res) => setMahasiswa(res.data.data))
      .catch(() => setError('Gagal memuat data.'))
      .finally(() => setLoading(false));
  }, []);

  return { mahasiswa, loading, error };
}
```

```jsx
// Dipakai di komponen mana pun, jadi sangat ringkas
import { useMahasiswa } from '../hooks/useMahasiswa';

function DaftarMahasiswa() {
  const { mahasiswa, loading, error } = useMahasiswa();

  if (loading) return <p>Memuat...</p>;
  if (error) return <p>{error}</p>;

  return (
    <ul>
      {mahasiswa.map((m) => <li key={m.id}>{m.name}</li>)}
    </ul>
  );
}
```

## Latihan

1. Buat custom hook `useMahasiswa()` seperti contoh di atas, pakai di komponen `DaftarMahasiswa`.
2. Buat `FormTambahMahasiswa` yang, setelah berhasil `POST`, menambahkan data baru ke list tanpa reload/fetch ulang seluruh data.
3. Tambahkan tombol hapus di tiap item list yang memanggil `DELETE /api/mahasiswa/{id}`.
4. Coba matikan server Laravel (`Ctrl+C` di terminal `php artisan serve`), lalu refresh React — pastikan pesan error state (bukan blank page) yang muncul.

---
⬅️ [16. Komponen, Props, State](../16-react-komponen-props-state/README.md) | ➡️ Lanjut ke [18. Studi Kasus: Integrasi Full-Stack](../18-studi-kasus-fullstack-integrasi/README.md)
