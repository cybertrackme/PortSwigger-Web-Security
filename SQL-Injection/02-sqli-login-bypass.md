# [PortSwigger Academy] SQL Injection - Lab #2

## 📌 Detail Lab
* **Nama Tantangan:** SQL injection vulnerability allowing login bypass
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** SQL Injection (SQLi)
* **Tujuan Lab:** Melakukan bypass pada mekanisme login aplikasi untuk masuk ke dalam sistem sebagai user `administrator` tanpa mengetahui password-nya.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memiliki fitur autentikasi berupa formulir login. Celah keamanan terjadi karena input dari pengguna pada parameter `username` langsung digabungkan ke dalam kueri SQL di sisi backend tanpa adanya proses sanitasi atau validasi.

### Perkiraan Kueri SQL Asli Backend:
```sql
SELECT * FROM users WHERE username = 'INPUT_USER' AND password = 'INPUT_PASSWORD'
```

---

## 2. Langkah Eksploitasi & PoC
Buka halaman login pada aplikasi target, masukkan data acak pada kolom login (contoh username: test dan password: test), lalu tangkap request tersebut menggunakan Burp Suite Proxy. Kirim request HTTP POST yang tertangkap ke Burp Repeater.
```
POST /login HTTP/2
Host: 0a81002c030408bf8161c5fe00a6009b.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

username=test&password=test
```

### Fuzzing & Analisis Error
Uji parameter username dengan menambahkan karakter petik tunggal ('). Aplikasi merespons dengan status HTTP 500 Internal Server Error atau indikasi error lainnya. Hal ini mengonfirmasi bahwa karakter tersebut berhasil merusak struktur sintaksis kueri database pada backend.

### Injeksi Payload untuk Bypass Autentikasi
Untuk memanipulasi kueri agar aplikasi hanya mengecek keberadaan username administrator dan mengabaikan seluruh instruksi pengecekan password di belakangnya, masukkan payload berikut pada parameter username:
```
POST /login HTTP/2
Host: 0a81002c030408bf8161c5fe00a6009b.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

username=administrator'--&password=test
```

### Hasil Eksekusi Database
```
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'test'
```
Karakter petik tunggal (') menutup input untuk kolom username secara paksa.
Karakter -- bertindak sebagai perintah comment yang mematikan dan mengabaikan sisa kueri pemeriksaan password di belakangnya (' AND password = 'test').

Hasil Akhir: Database mengembalikan hasil bahwa user administrator ditemukan (bernilai TRUE). Aplikasi merespons dengan HTTP 302 Found (Redirect) dan memberikan session cookie baru. Login bypass berhasil dan lab dinyatakan Solved.

---

## 3. Rekomendasi Perbaikan
Untuk mencegah serangan Login Bypass berbasis SQLi, tim developer wajib mengimplementasikan Parameterized Queries (Prepared Statements) pada fungsi autentikasi.

Contoh Implementasi Perbaikan Kode (PHP PDO):

Daripada menggunakan kode rentan yang menggabungkan input secara langsung:
// KODE RENTAN (VULNERABLE)
```
$query = "SELECT * FROM users WHERE username = '" . $_POST['username'] . "' AND password = '" . $_POST['password'] . "'";
$db->query($query);
```

Developer harus mengubahnya menggunakan Prepared Statement:
// KODE AMAN (SECURE)
```
$stmt = $db->prepare('SELECT * FROM users WHERE username = :username AND password = :password');
$stmt->execute([
    'username' => $_POST['username'],
    'password' => $_POST['password']
]);
```