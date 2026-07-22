# 04. JavaScript Dasar

JavaScript adalah bahasa yang membuat halaman web **interaktif** — sesuatu terjadi saat tombol diklik, form divalidasi sebelum submit, data diambil dari server tanpa reload halaman. Modul ini adalah **fondasi wajib** sebelum masuk ke React (modul 15+).

## Tujuan Belajar

- Paham sintaks dasar: variabel, tipe data, fungsi, kondisi, perulangan.
- Paham array & object, termasuk method modern (`map`, `filter`, `find`).
- Paham manipulasi DOM dan event handling.
- Paham **asynchronous JavaScript**: Promise, `async/await`, dan `fetch` — ini akan langsung dipakai lagi persis sama di React (modul 17).

## 1. Variabel & Tipe Data

```js
let nama = "Ahmad Fauzi"; // bisa diubah nilainya
const maxSks = 24;        // tidak bisa diubah (dipakai lebih sering di kode modern)
var lama = "hindari var"; // gaya lama, hindari di kode baru

console.log(typeof nama);   // "string"
console.log(typeof maxSks); // "number"

let aktif = true;            // boolean
let jurusan = null;          // sengaja kosong
let belumDiisi = undefined;  // belum diberi nilai
```

## 2. Fungsi

```js
// Function declaration
function sapa(nama) {
    return `Halo, ${nama}!`; // template literal, pakai backtick
}

// Arrow function (gaya modern, sering dipakai di React)
const sapaArrow = (nama) => {
    return `Halo, ${nama}!`;
};

// Arrow function singkat (implicit return)
const sapaSingkat = (nama) => `Halo, ${nama}!`;

console.log(sapa("Ahmad")); // "Halo, Ahmad!"
```

## 3. Kondisi & Perulangan

```js
const sks = 3;

if (sks === 0) {
    console.log("Mata kuliah tanpa SKS, tidak valid");
} else if (sks <= 2) {
    console.log("Mata kuliah ringan");
} else {
    console.log("Mata kuliah beban penuh");
}

// Perulangan
for (let i = 0; i < 3; i++) {
    console.log(`Iterasi ke-${i}`);
}
```

## 4. Array & Object — Bagian Terpenting Sebelum React

```js
const mahasiswa = [
    { id: 1, stambuk: "20210011", name: "Ahmad Fauzi", jurusan: "Teknik Informatika" },
    { id: 2, stambuk: "20210012", name: "Siti Rahma", jurusan: "Sistem Informasi" },
    { id: 3, stambuk: "20210013", name: "Budi Santoso", jurusan: "Teknik Informatika" },
];

// map: mengubah tiap item menjadi bentuk baru (dipakai untuk render list di React!)
const namaSaja = mahasiswa.map((m) => m.name);
console.log(namaSaja); // ["Ahmad Fauzi", "Siti Rahma", "Budi Santoso"]

// filter: menyaring item sesuai kondisi
const jurusanTI = mahasiswa.filter((m) => m.jurusan === "Teknik Informatika");
console.log(jurusanTI); // hanya Ahmad Fauzi & Budi Santoso

// find: mencari 1 item pertama yang cocok
const ahmad = mahasiswa.find((m) => m.name === "Ahmad Fauzi");
console.log(ahmad); // { id: 1, stambuk: "20210011", name: "Ahmad Fauzi", jurusan: "Teknik Informatika" }

// Destructuring — sangat sering dipakai di React
const { name, jurusan } = ahmad;
console.log(name, jurusan); // "Ahmad Fauzi" "Teknik Informatika"

// Spread operator — sering dipakai untuk update state di React
const ahmadPindahJurusan = { ...ahmad, jurusan: "Sistem Informasi" };
```

> **Kenapa `map`, `filter`, destructuring, dan spread ditekankan?** Karena pola inilah yang nanti dipakai berulang-ulang untuk merender daftar data dan mengelola state di React. Kalau ini sudah lancar, React di modul 15-17 akan terasa jauh lebih mudah.

## 5. Manipulasi DOM & Event

```html
<button id="btn-simpan">Simpan Mahasiswa</button>
<p id="status"></p>

<script>
    const tombol = document.getElementById("btn-simpan");
    const status = document.getElementById("status");

    tombol.addEventListener("click", () => {
        status.textContent = "Data mahasiswa berhasil disimpan!";
    });
</script>
```

## 6. Asynchronous JavaScript — Promise & `fetch`

Banyak operasi di web butuh waktu (ambil data dari server, baca file) — JS tidak menunggu diam, tapi memakai **Promise**.

```js
// fetch mengembalikan Promise
fetch("https://jsonplaceholder.typicode.com/users/1")
    .then((response) => response.json())
    .then((data) => console.log(data))
    .catch((error) => console.error("Gagal ambil data:", error));
```

Cara modern & lebih rapi: `async/await` (sintaks ini **persis sama** yang akan dipakai lagi di dalam komponen React):

```js
async function ambilDataUser() {
    try {
        const response = await fetch("https://jsonplaceholder.typicode.com/users/1");
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("Gagal ambil data:", error);
    }
}

ambilDataUser();
```

## Studi Mini: Render Daftar Mahasiswa dengan JS Murni

```html
<div id="daftar-mahasiswa"></div>

<script>
    const mahasiswa = [
        { stambuk: "20210011", name: "Ahmad Fauzi", jurusan: "Teknik Informatika" },
        { stambuk: "20210012", name: "Siti Rahma", jurusan: "Sistem Informasi" },
        { stambuk: "20210013", name: "Budi Santoso", jurusan: "Teknik Informatika" },
    ];

    const container = document.getElementById("daftar-mahasiswa");

    mahasiswa.forEach((m) => {
        const el = document.createElement("div");
        el.textContent = `${m.stambuk} — ${m.name} (${m.jurusan})`;
        container.appendChild(el);
    });
</script>
```

Perhatikan: ini **manual** — kita harus bikin elemen satu-satu dan tempel ke DOM. Nanti di React (modul 16), proses ini jauh lebih deklaratif memakai JSX + `.map()`.

## Latihan

1. Buat array 5 objek `mahasiswa` (stambuk, name, jurusan). Render daftar itu ke halaman HTML pakai `map`/`forEach` + DOM manipulation.
2. Tambahkan tombol filter: tampilkan hanya mahasiswa jurusan "Teknik Informatika" (pakai `.filter()`).
3. Buat fungsi `async` yang fetch data dari `https://jsonplaceholder.typicode.com/users` dan tampilkan nama-nama user ke halaman.

---
⬅️ [03. CSS Framework & Library](../03-css-framework-library/README.md) | ➡️ Lanjut ke [05. PHP Dasar & OOP](../05-php-dasar-oop/README.md)
