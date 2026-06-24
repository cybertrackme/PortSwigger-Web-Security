# [PortSwigger Academy] SQL Injection - Lab #1

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menggunakan kerentanan SQLi pada filter kategori untuk menampilkan seluruh produk, termasuk produk yang belum dirilis (hidden data).

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memiliki fitur filter kategori produk. Input dari pengguna pada parameter `?category=` langsung digabungkan ke dalam kueri SQL di sisi backend tanpa adanya proses sanitasi.

### Perkiraan Query SQL Asli Backend:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```
---

## 2. Langkah eksplotasi & PoC
Tangkap Request menggunakan Burp Suite Proxy:
lalu klik salah satu filter untuk mengambil request endpoint, kategori disini saya memilih Gifts pada aplikasi web target. Kirim request HTTP yang tertangkap ke Burp Repeater.
```
GET /filter?category=Gifts HTTP/2
Host: 0acd003603b207e3817a2017009800e5.web-security-academy.net
```

### Fuzzing & Analisis Error:
Suntikkan karakter petik tunggal (') di ujung parameter untuk menguji respons database: /filter?category=Gifts'. Aplikasi merespons dengan status HTTP 500 Internal Server Error. Hal ini membuktikan bahwa input kita berhasil merusak sintaksis kueri SQL di backend (indikasi awal celah SQLi).

### Injeksi Payload untuk Bypass Logika:
Untuk memanipulasi query agar selalu menghasilkan nilai benar sekaligus mengabaikan kondisi pengecekan rilis produk, masukkan payload ini pada parameter kategori: 
```
GET /filter?category=Gifts' OR 1=1 -- HTTP/1.1
Host: 0acd003603b207e3817a2017009800e5.web-security-academy.net
```
Hasil Eksekusi Database:
Di sisi database backend, query dimanipulasi menjadi:
```
SELECT * FROM products WHERE category = 'Gifts' OR 1=1 '--' AND released = 1
```

Kondisi OR 1=1 memaksa seluruh baris data pada tabel produk bernilai TRUE.
Karakter -- bertindak sebagai perintah comment yang mematikan sisa query di belakangnya (AND released = 1).

Hasil Akhir: Aplikasi menampilkan seluruh produk yang ada di database, termasuk produk tersembunyi yang belum dirilis. Lab berhasil diselesaikan (Solved).
