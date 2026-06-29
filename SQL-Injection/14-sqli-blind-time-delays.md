# [PortSwigger Academy] SQL Injection - Lab #14

## 📌 Detail Lab
* **Nama Tantangan:** Blind SQL injection with time delays
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi) - Blind Time-Based
* **Tujuan Lab:** Membuktikan adanya celah keamanan SQL Injection pada parameter cookie dengan cara menyuntikkan payload yang memicu jeda waktu respons (*time delay*) selama 10 detik dari database.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Ketika aplikasi web sama sekali tidak menunjukkan perubahan visual (baik teks maupun kode status HTTP) saat diberikan input logika yang berbeda, kita dapat menggunakan teknik **Time-Based Blind SQLi**. 

Teknik ini memanfaatkan fungsi internal database yang dirancang untuk menunda eksekusi kueri selama durasi tertentu. Karena aplikasi web harus menunggu database selesai memproses seluruh kueri sebelum mengirimkan respons HTTP kembali ke browser, kita bisa mendeteksi keberadaan celah ini dengan mengukur berapa lama waktu yang dibutuhkan server untuk merespons.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Karakteristik Time Delay Berdasarkan DBMS
Setiap jenis database memiliki fungsi penunda waktu yang berbeda-beda:
* **PostgreSQL:** `pg_sleep(detik)`
* **MySQL:** `sleep(detik)`
* **Microsoft SQL Server:** `WAITFOR DELAY 'jam:menit:detik'`

Karena arsitektur lab ini menggunakan **PostgreSQL**, kita akan memanfaatkan fungsi `pg_sleep()`.

### Menyuntikkan Payload Uji Jeda Waktu
Buka Burp Repeater, lalu modifikasi nilai pada cookie `TrackingId` dengan menambahkan operator penggabung kueri teks serta fungsi `pg_sleep(10)` untuk menginstruksikan database agar menunda proses selama 10 detik.

* **Payload:** `' || pg_sleep(10)--`

Request HTTP final di Burp Repeater:
```http
GET / HTTP/2
Host: 0a81009b035b08418417735b001400c9.web-security-academy.net
Cookie: TrackingId=nGQJjA124hYnDxmw'|| pg_sleep(10)--
```

### Analisis Hasil Pengukuran Waktu (Response Time)
Kirim request tersebut dan perhatikan indikator waktu respons (Response time) di sudut kanan bawah aplikasi Burp Suite:

    Request Normal: Biasanya selesai diproses dalam waktu beberapa milidetik saja.

    Request Hasil Injeksi: Server membutuhkan waktu 10.000+ milidetik (10 detik) sebelum akhirnya mengembalikan respons HTTP 200 OK ke browser.

Hasil Akhir: Jeda waktu respons yang signifikan ini menjadi bukti tak terbantahkan bahwa kueri SQL buatan kita berhasil dieksekusi secara langsung oleh mesin database PostgreSQL backend. Lab berhasil diselesaikan (Solved).