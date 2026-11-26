# Panduan Pengujian REST API FlipPlay di Postman

Dokumentasi ini berisi langkah-langkah pengecekan dan pengujian seluruh endpoint **FlipPlay REST API** (`http://127.0.0.1:8000`) menggunakan **Postman**, mencakup skenario *happy path* maupun *failure path*.

## Persiapan Sebelum Pengujian

1. Pastikan server database MySQL aktif dan migrasi Laravel sudah dijalankan.
2. Pastikan minimal tersedia data pengguna uji di database (misalnya user `Alya` dengan `id: 1`).
3. Jalankan server pengembangan Laravel di terminal proyek:
   ```bash
   php artisan serve
   ```
4. Pada setiap request di Postman, pastikan header berikut aktif:
   - `Accept`: `application/json`
   - `Content-Type`: `application/json` (untuk request yang mengirimkan *raw JSON body* seperti `POST` dan `PATCH`)

---

## Urutan Skenario & Endpoint Pengujian

### 1. Tambah Media — Uji Validasi Duplikat (`422 Unprocessable Entity`)
- **Method & URL**: `POST http://127.0.0.1:8000/api/media`
- **Body (JSON)**:
  ```json
  {
    "title": "Laskar Pelangi",
    "type": "movie",
    "creator": "Riri Riza",
    "year": 2008,
    "duration": 120
  }
  ```
- **Ekspektasi**: Jika judul media sudah terdaftar di database, API mengembalikan status **`422`** beserta pesan validasi duplikat.

### 2. Tambah Media Baru (`201 Created`)
- **Method & URL**: `POST http://127.0.0.1:8000/api/media`
- **Body (JSON)**:
  ```json
  {
    "title": "Habibie & Ainun",
    "type": "movie",
    "creator": "Faozan Rizal",
    "year": 2012,
    "duration": 118
  }
  ```
- **Ekspektasi**: Mengembalikan status **`201`** dan objek `data` media yang baru dibuat (contohnya `id: 2`).

### 3. Cari & Daftar Media dengan Query Parameter (`200 OK`)
- **Method & URL**: `GET http://127.0.0.1:8000/api/media?title=Habibie`
- **Ekspektasi**: Mengembalikan status **`200`** dan menampilkan daftar media yang cocok dengan kata kunci `title`.

### 4. Lihat Detail Satu Media (`200 OK`)
- **Method & URL**: `GET http://127.0.0.1:8000/api/media/2`
- **Ekspektasi**: Mengembalikan status **`200`** dan menampilkan detail lengkap media dengan `id: 2`.

### 5. Perbarui Data Media (`200 OK`)
- **Method & URL**: `PATCH http://127.0.0.1:8000/api/media/2`
- **Body (JSON)**:
  ```json
  {
    "duration": 125
  }
  ```
- **Ekspektasi**: Mengembalikan status **`200`** dan nilai `duration` pada respons berubah menjadi `125`.

### 6. Tambah Media ke Koleksi User (`201 Created`)
- **Method & URL**: `POST http://127.0.0.1:8000/api/users/1/collection`
- **Body (JSON)**:
  ```json
  {
    "media_id": 2
  }
  ```
- **Ekspektasi**: Mengembalikan status **`201`** dengan pesan konfirmasi berhasil ditambahkan ke koleksi user serta daftar `collection` terbaru.

### 7. Uji Penolakan Duplikat di Koleksi User (`409 Conflict`)
- **Method & URL**: `POST http://127.0.0.1:8000/api/users/1/collection`
- **Body (JSON)**:
  ```json
  {
    "media_id": 2
  }
  ```
- **Ekspektasi**: Mengirim ulang request yang sama akan mengembalikan status **`409`** dengan pesan `"Media sudah ada di koleksi."`.

### 8. Lihat Daftar Koleksi User (`200 OK`)
- **Method & URL**: `GET http://127.0.0.1:8000/api/users/1/collection`
- **Ekspektasi**: Mengembalikan status **`200`** dan menampilkan informasi user beserta array `collection` miliknya.

### 9. Hapus Media dari Koleksi User (`200 OK`)
- **Method & URL**: `DELETE http://127.0.0.1:8000/api/users/1/collection/2`
- **Ekspektasi**: Mengembalikan status **`200`** dengan pesan `"Media berhasil dihapus dari koleksi."` dan `collection` kembali kosong (`[]`).

### 10. Hapus Data Media (`200 OK`)
- **Method & URL**: `DELETE http://127.0.0.1:8000/api/media/2`
- **Ekspektasi**: Mengembalikan status **`200`** dengan pesan `"Media berhasil dihapus"`.