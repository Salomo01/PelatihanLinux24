# Modul 4 - Filesystem Chiho (maimai SEGA)

## *Soal*

Pada modul ini, saya diminta untuk membuat sebuah **filesystem virtual** menggunakan FUSE yang merepresentasikan dunia *maimai* dari SEGA. Filesystem ini terdiri dari **7 area (chiho)**, dan setiap chiho memiliki aturan manipulasi file yang berbeda-beda. Tujuan dari soal ini adalah untuk mengimplementasikan aturan tersebut dalam bentuk operasi filesystem seperti `read`, `write`, `create`, `unlink`, dll.

---

## 📌 Fitur Utama

### 📁 Struktur Area (Chiho)

Filesystem terdiri dari 7 direktori/area utama:

```
fuse_dir/
├── starter/
├── metro/
├── dragon/
├── blackrose/
├── heaven/
├── youth/
└── 7sref/
```


Setiap area memiliki perlakuan khusus terhadap file yang disimpan di dalamnya.

---

## *Penjelasan Setiap Area*

### 🅰️ Starter Chiho (Area Pemula)

📝 Semua file disimpan dengan ekstensi tambahan `.mai`, namun saat dilihat di FUSE (mount point), ekstensi ini **disembunyikan**.

🔧 Implementasi:
- Menambahkan `.mai` pada saat `create` dan `unlink`.
- Menghapus ekstensi saat `readdir` untuk tampilan yang bersih.

---

### 🅱️ Metropolis Chiho (World's End)

📝 File disimpan dengan **nama yang telah di-shift** berdasarkan posisi karakter dalam namanya. Misalnya: `ener.txt` disimpan sebagai `eogu.txt`.

🔧 Implementasi:
- Fungsi `shift_file_name()` untuk menggantikan setiap karakter dengan karakter hasil penambahan `(i % 256)`.
- Digunakan saat `create`, `read`, `write`, dan `unlink`.

---

### 🅲️ Dragon Chiho (World Tree)

📝 Isi file akan disimpan dalam bentuk terenkripsi menggunakan algoritma **ROT13**, yaitu penggeseran alfabet sebanyak 13 posisi.

🔧 Implementasi:
- Fungsi `rot_13()` digunakan untuk mengenkripsi dan mendekripsi buffer selama proses `read` dan `write`.

---

### 🅳️ Blackrose Chiho (Black Rose Area)

📝 File disimpan **dalam bentuk biner murni** tanpa enkripsi ataupun encoding tambahan.

🔧 Implementasi:
- File diperlakukan secara default tanpa transformasi nama atau isi.

---

### 🅴️ Heaven Chiho (Tenkai Area)

📝 Semua file disimpan menggunakan enkripsi **AES-256-CBC**, dengan IV (Initialization Vector) yang ditulis di awal file.

🔧 Implementasi:
- Fungsi `aes_encrypt()` dan `aes_decrypt()` digunakan pada saat file dibaca atau ditulis.
- File hasil enkripsi disimpan, sementara plaintext digunakan sebagai file sementara.

---

### 🅵️ Youth Chiho (Skystreet)

📝 Semua file yang disimpan akan dikompres secara otomatis menggunakan **GZIP** untuk menghemat storage.

🔧 Implementasi:
- Fungsi `compress_to_gzip()` digunakan saat `release` file.
- Fungsi `decompress_gzip()` digunakan sebelum `read`.

---

### 🅶️ 7sRef Chiho (Prism Area)

📝 Area ini adalah gateway untuk mengakses seluruh chiho lain dengan sistem penamaan khusus `[area]_[filename]`.

📌 Contoh:

```
/fuse_dir/7sref/starter_guide.txt → /fuse_dir/starter/guide.txt
/fuse_dir/7sref/metro_data.log → /fuse_dir/metro/data.log
```


🔧 Implementasi:
- Fungsi `map_7sref_to_real()` digunakan untuk menerjemahkan path menjadi file asli sesuai area dan nama file.
- Digunakan pada semua operasi seperti `getattr`, `read`, `write`, `create`, dan `unlink`.

---

## 📋 Operasi Filesystem yang Didukung

Berikut adalah daftar fungsi yang diimplementasikan:
- `getattr` → Mengambil atribut file atau folder.
- `readdir` → Membaca isi direktori.
- `open` / `release` → Membuka dan menutup file.
- `read` / `write` → Membaca dan menulis isi file (dengan transformasi).
- `create` / `unlink` → Membuat dan menghapus file.

---

## ⚙️ Cara Kompilasi dan Jalankan

### 1. Kompilasi Program
```
gcc -Wall -o maimai_fs maimai_fs.c `pkg-config fuse3 --cflags --libs` -lcrypto -lz
```

### 2. Jalankan Filesystem
```
./maimai_fs fuse_dir/

```

### 3. Unmount Filesystem
```
fusermount3 -u fuse_dir/

```
