# [PortSwigger Academy] SQL Injection - Lab #13

## 📌 Detail Lab
* **Nama Tantangan:** Visible error-based SQL injection
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi) - Error-Based
* **Tujuan Lab:** Mengekstrak password milik user `administrator` dengan cara memicu pesan error database spesifik yang menampilkan informasi sensitif tersebut langsung pada layar aplikasi web, lalu login ke sistem.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi menggunakan cookie `TrackingId` dalam kueri SQL backend. Jika kita memasukkan karakter yang merusak sintaksis, aplikasi tidak hanya menghasilkan error, tetapi juga menampilkan pesan kesalahan dari database secara detail ke halaman pengguna.

Kita bisa memanfaatkan fungsi pengondisian tipe data atau fungsi matematika database untuk sengaja menciptakan error logika. Ketika database memproses kueri ilegal tersebut, ia akan menyertakan hasil sub-kueri kita ke dalam teks pesan error yang ditampilkan di layar.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Memverifikasi Celah dan Menampilkan Error
Suntikkan karakter petik tunggal (`'`) pada cookie `TrackingId` melalui Burp Repeater untuk melihat apakah pesan error muncul di layar:
* **Respons:** Aplikasi menampilkan pesan error SQL mentah, yang mengonfirmasi bahwa backend menggunakan database **PostgreSQL**.

### Memicu Error untuk Mengekstrak Data (Data Leakage via Error)
Pada PostgreSQL, kita bisa memicu error konversi tipe data dengan menggunakan fungsi `CAST()`. Kita akan memaksa database mengubah hasil kueri string (data password) menjadi tipe data boolean. Karena konversi ini mustahil secara logika, PostgreSQL akan memicu error yang membocorkan isi data tersebut.

Masukkan payload berikut pada parameter `TrackingId` :
Untuk mengecek isi data pertama di column username
* **Payload:** `'AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--`
Hasil nya kita dapat pesan eror yang berisi nama 'administrator' :
`ERROR: invalid input syntax for type integer: "administrator"`

Masukkan payload berikut pada parameter `TrackingId`:
* **Payload:** `'AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`

Request HTTP final di Burp Repeater:
```http
GET / HTTP/2
Host: 0a06003503c2f818800adaa700670019.web-security-academy.net
Cookie: TrackingId='AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

### Analisis Hasil Teks Error
Setelah request dikirim, periksa halaman respons. Aplikasi akan menampilkan pesan error fatal dari PostgreSQL yang berisi password teks murni :
`Internal Server Error: ERROR: invalid input syntax for type integer: "5ysllawyrpn49z5hadyk"`

Analisis: Karena database gagal mengubah teks password "5ysllawyrpn49z5hadyk" menjadi angka, database secara otomatis menyebutkan nilai teks yang gagal dikonversi tersebut di dalam laporan error-nya.

### Hasil Eksploitasi
Hasil Akhir: Salin string password yang bocor di dalam pesan error tersebut 5ysllawyrpn49z5hadyk, buka halaman login aplikasi web target, lalu masuk menggunakan username administrator. Login sukses dan lab dinyatakan Solved.