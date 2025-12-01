# Laporan Praktikum Keamanan Sistem
## Topik: Broken Access Control
## Oleh: Nur Cayanto
## Pangkat : Praka Mar
## Kelas: T.I Cyber

## Pengujian Keamanan Web: Broken Access Control (BAC)

**Metode: OWASP Top 10 – Broken Access Control**

---

# 1. Pendahuluan

## 1.1 Latar Belakang

Keselamatan dan keamanan aplikasi web merupakan aspek yang sangat penting dalam pengembangan sistem informasi modern. Salah satu kerentanan paling berbahaya pada aplikasi web berdasarkan **OWASP Top 10** adalah **Broken Access Control**.
Kerentanan ini muncul ketika aplikasi gagal membatasi akses pengguna, sehingga seorang user dapat:

* mengakses data milik pengguna lain,
* mengakses fungsi administrator,
* memanipulasi parameter sensitif, atau
* mengakses direktori tersembunyi tanpa otorisasi.

OWASP mendefinisikan Broken Access Control sebagai kelemahan mekanisme kontrol akses yang memungkinkan pengguna untuk bertindak di luar hak istimewa yang dimilikinya. Karena dampaknya yang sangat besar, pengujian BAC menjadi kompetensi penting dalam bidang security testing dan ethical hacking.

---

## 1.2 Tujuan Praktikum

Tujuan dari praktikum ini adalah:

1. Memahami konsep Broken Access Control berdasarkan panduan OWASP.
2. Melakukan identifikasi dan eksploitasi BAC menggunakan aplikasi rentan (DVWA / Juice Shop).
3. Melakukan intercept, manipulasi parameter, dan fuzzing menggunakan **OWASP ZAP**.
4. Menganalisis bentuk kerentanan seperti:

   * Horizontal Access Control Failure
   * Vertical Access Control Failure
   * IDOR (Insecure Direct Object Reference)
   * Forced Browsing
5. Menyusun rekomendasi mitigasi keamanan untuk mencegah BAC pada aplikasi web.

---

## 1.3 Dasar Teori

### 1.3.1 OWASP Top 10

OWASP (Open Web Application Security Project) adalah organisasi nirlaba yang menyediakan panduan keamanan web.
Dalam daftar OWASP Top 10, Broken Access Control menempati rank **A01:2021** sebagai kerentanan **paling kritis dan umum**.

---

### 1.3.2 Broken Access Control

Broken Access Control terjadi ketika aplikasi tidak menerapkan pembatasan akses dengan benar.
Ciri-cirinya meliputi:

* Tidak ada validasi peran/role
* URL dapat dimanipulasi
* Data user dapat diakses dengan mengubah parameter
* Backend tidak memverifikasi session/ownership
* Direct object references (IDOR) tanpa otorisasi

---

### 1.3.3 Jenis-Jenis Broken Access Control (OWASP)

#### a. Horizontal Access Control

Pengguna dapat mengakses resource milik pengguna lain.
Contoh:

```
/view?user_id=100 → ubah ke ?user_id=101
```

#### b. Vertical Access Control

Pengguna role rendah bisa mengakses fitur admin.
Contoh:

```
/admin/dashboard
```

#### c. IDOR (Insecure Direct Object Reference)

Aplikasi menggunakan ID langsung tanpa validasi pemilik resource.

#### d. Forced Browsing

Mengakses URL tersembunyi tanpa autentikasi.
Contoh direktori:

```
/backup.zip  
/admin/  
/private/
```

---

## 1.4 Lingkup Kegiatan

Praktikum ini akan meliputi:

* Instalasi dan konfigurasi OWASP ZAP.
* Pengujian BAC pada aplikasi rentan (DVWA & Juice Shop).
* Intercept HTTP request dan manipulasi parameter.
* Fuzzing ID untuk menemukan objek sensitif.
* Pengujian akses API menggunakan Juice Shop.
* Analisis kerentanan dan dokumentasi bukti pengujian.

---

# 2. Peralatan & Lingkungan Uji

Bagian ini menjelaskan seluruh tools, software, serta environment yang digunakan untuk melakukan pengujian Broken Access Control berdasarkan pedoman OWASP. Seluruh konfigurasi dijelaskan secara detail agar dapat direplikasi dengan mudah.

---

## 2.1 Perangkat Lunak yang Digunakan

| No | Nama Software                      | Versi  | Fungsi                                                  |
| -- | ---------------------------------- | ------ | ------------------------------------------------------- |
| 1  | **OWASP ZAP**                      | 2.14+  | Intercept, scanning, fuzzing, dan eksploitasi BAC       |
| 2  | **DVWA – Damn Vulnerable Web App** | Latest | Target aplikasi rentan untuk uji IDOR & BAC             |
| 3  | **OWASP Juice Shop**               | Latest | Target aplikasi modern untuk uji API-level BAC          |
| 4  | **XAMPP**                          | 7–8    | Menjalankan DVWA secara lokal                           |
| 5  | **Docker** (opsional)              | 24+    | Alternatif menjalankan Juice Shop secara container      |
| 6  | **Browser Firefox**                | Latest | Digunakan karena kompatibilitas tinggi dengan ZAP proxy |
| 7  | **Kali Linux / Windows**           | —      | Sistem operasi pengujian keamanan                       |

> Catatan: Semua tools bebas digunakan selama dapat menjalankan HTTP proxy dan aplikasi target.

---

## 2.2 Tools Pendukung

Selain software utama, beberapa tools pendukung digunakan:

* **Curl** → Pengujian API sederhana
* **Burp Suite (opsional)** → Pembanding ZAP
* **Terminal / CMD** → Menjalankan Docker Juice Shop
* **Notepad++ / VSCode** → Dokumentasi & pemeriksaan request

---

## 2.3 Arsitektur Lingkungan Pengujian

Struktur sederhana pengujian BAC:

```
Browser → OWASP ZAP Proxy → Target Aplikasi (DVWA/Juice Shop)
```

Penjelasan:

* Traffic dari browser diarahkan ke ZAP (127.0.0.1:8080).
* ZAP mencatat seluruh request → memungkinkan manipulasi parameter.
* ZAP meneruskan request ke aplikasi target.
* Response yang diterima dianalisis untuk menemukan BAC.

---

## 2.4 Konfigurasi OWASP ZAP sebagai Proxy

### **Langkah 1 – Jalankan OWASP ZAP**

* Buka aplikasi ZAP
* Pilih **Start with Default Settings**
* Biarkan semua konfigurasi default aktif

### **Langkah 2 – Atur Browser Melalui Proxy**

Pada Firefox:

1. Buka **Settings → Network Settings**
2. Pilih **Manual Proxy Configuration**
3. Input:

```
HTTP Proxy: 127.0.0.1
Port: 8080
```

4. Centang **Use this proxy for all protocols**
5. Simpan

### **Langkah 3 – Verifikasi**

* Buka browser → akses `http://example.com`
* Jika ZAP mencatat request pada tab **History** → Proxy berhasil

---

## 2.5 Instalasi Aplikasi Target

### **A. Menjalankan DVWA via XAMPP (Windows/Linux)**

#### Langkah 1 — Download DVWA

Download melalui GitHub:
`https://github.com/digininja/DVWA`

#### Langkah 2 — Ekstrak ke folder:

```
xampp/htdocs/dvwa/
```

#### Langkah 3 — Konfigurasi Database

Edit file:

```
config/config.inc.php
```

Ubah kredensial menjadi:

```
DB_USER = 'root';
DB_PASSWORD = '';
```

#### Langkah 4 — Jalankan XAMPP

* Start **Apache**
* Start **MySQL**

#### Langkah 5 — Setup DVWA

Akses:

```
http://localhost/dvwa/setup.php
```

Klik **Create / Reset Database**

Selesai—DVWA siap digunakan.

---

### **B. Menjalankan OWASP Juice Shop via Docker (Rekomendasi)**

#### Langkah 1 — Pull Image

```
docker pull bkimminich/juice-shop
```

#### Langkah 2 — Jalankan Container

```
docker run -d -p 3000:3000 bkimminich/juice-shop
```

#### Langkah 3 — Akses Juice Shop

```
http://localhost:3000/
```

Juice Shop siap untuk uji API Broken Access Control.

---

## 2.6 Penyesuaian Tingkat Keamanan DVWA

Agar eksploitasi Broken Access Control lebih realistis:

1. Login DVWA:

```
admin : password
```

2. Masuk ke menu:

```
DVWA Security → Set to Low
```

Alasannya:

* Mode **Low** memudahkan eksploit IDOR, Forced Browsing, dan Horizontal BAC.
* Mode **High** memiliki filtering sehingga sulit digunakan untuk studi dasar.

---

## 2.7 Verifikasi Semua Komponen Berjalan

Checklist sebelum memulai praktikum:

* [x] ZAP terbuka dan berjalan sebagai proxy
* [x] Browser sudah mengarah ke 127.0.0.1:8080
* [x] DVWA atau Juice Shop sudah dapat diakses
* [x] Traffic browser muncul di tab **ZAP → History**
* [x] Internet *tidak* diperlukan (running offline)

Jika semua tercentang, maka lingkungan uji telah siap.

---

## 2.8 Ringkasan Bagian Ini

Pada bagian ini telah dijelaskan:

* Tools dan software yang digunakan
* Infrastruktur pengujian BAC
* Konfigurasi proxy OWASP ZAP
* Instalasi DVWA dan Juice Shop
* Persiapan dan verifikasi environment

Dengan seluruh lingkungan siap, praktikum selanjutnya akan fokus pada eksploitasi Broken Access Control secara langsung.

---

## 3. Identifikasi Broken Access Control

Pada tahap ini, kita akan menganalisis bagaimana kerentanan Broken Access Control dapat ditemukan pada aplikasi web, menggunakan pendekatan sistematis berdasarkan rekomendasi OWASP Testing Guide dan OWASP Top 10.

---

### **3.1 Tujuan Pengujian**

* Menentukan apakah aplikasi memiliki akses yang tidak diproteksi dengan benar.
* Mengetahui apakah pengguna dapat mengakses fungsi atau data yang seharusnya tidak boleh diakses.
* Mengidentifikasi pola kelemahan seperti IDOR, privilege escalation, insecure direct object reference, dan bypassing authorization.

---

### **3.2 Metodologi Identifikasi**

Pengujian ini mengikuti pendekatan OWASP dengan fokus pada aspek berikut:

#### **3.2.1 Pengujian Kontrol Akses Horisontal**

Mengidentifikasi apakah pengguna dapat mengakses data milik pengguna lain.

**Contoh skenario:**

* User A masuk ke aplikasi dan memiliki URL:
  `/user/profile?id=1001`
* User A lalu mengubah ID menjadi milik orang lain:
  `/user/profile?id=1002`

Jika data pengguna lain dapat muncul, maka terjadi **IDOR (Insecure Direct Object Reference)**.

**Indikasi kerentanan:**

* Tidak ada validasi session terhadap resource milik user.
* Parameter ID langsung digunakan pada query di backend.

---

#### **3.2.2 Pengujian Kontrol Akses Vertikal**

Mengidentifikasi apakah user biasa dapat mengakses fungsi admin.

**Contoh langkah:**

1. Login sebagai user biasa.
2. Coba akses URL yang biasanya hanya dimiliki admin:
   `/admin/dashboard`
   `/admin/user-management`
3. Coba intercept request menggunakan Burp Suite dan ubah parameter role:
   `"role": "user"` → `"role": "admin"`

**Indikasi kerentanan:**

* Tidak ada pengecekan role pada backend.
* Hanya mengandalkan hidden input atau JavaScript.

---

#### **3.2.3 Menguji Direct Access pada File & Endpoint**

Broken Access Control sering muncul pada folder yang tidak diproteksi.

**Contoh pengujian:**

* Akses folder seperti:
  `/backup/`
  `/private/`
  `/config/`

* Akses file sensitif:
  `/backup/db.sql`
  `/admin/export.xlsx`

Jika dapat diakses tanpa autentikasi, maka terjadi BAC.

---

#### **3.3 Identifikasi Menggunakan Burp Suite**

Burp Suite adalah alat paling umum untuk pengujian Broken Access Control.

**Langkah praktikum:**

##### **3.3.1 Menggunakan Burp Proxy**

* Set browser ke Burp Proxy.
* Login sebagai user dengan hak rendah.
* Akses berbagai fitur aplikasi.
* Burp akan menangkap request secara otomatis.

##### **3.3.2 Menggunakan Repeater untuk Manipulasi Parameter**

* Kirim request ke Repeater.
* Ubah parameter seperti:

  * `user_id`
  * `role`
  * `group`
  * `isAdmin=true`
* Eksekusi ulang request dan perhatikan respon.

##### **3.3.3 Menggunakan Intruder untuk Attack Pattern**

Digunakan untuk fuzzing ID pada skenario IDOR.

Contoh:

```
/api/user?id=§1001§
```

Setelah itu:

* Gunakan payload angka 1000–1100
* Evaluasi respons yang berbeda

Jika beberapa ID dapat diakses → aplikasi rentan IDOR.

---

### **3.4 Dokumentasi Temuan**

Semua temuan harus dicatat dengan format berikut:

#### **3.4.1 Format Dokumentasi Temuan**

* **ID Temuan**: BAC-01
* **Jenis Kerentanan**: IDOR
* **Endpoint Terdampak**: `/user/profile?id=1002`
* **Level Risiko**: Tinggi
* **Bukti Screenshot / Response Burp**
* **Deskripsi Singkat**
* **Dampak**: Paparan data sensitif milik user lain
* **Rekomendasi Perbaikan**

---

### **3.5 Kesimpulan Bagian Identifikasi**

* Pada tahap ini kita menentukan apakah aplikasi memiliki kontrol akses yang lemah.
* Fokus utama adalah mengevaluasi endpoint, parameter, role-based access, serta validasi backend.
* Semua hasil temuan harus dikumpulkan dan disiapkan untuk tahap analisis keamanan berikutnya.

---

# 4. Eksploitasi Broken Access Control

Pada bagian ini, kita akan melakukan eksploitasi secara langsung terhadap beberapa jenis kerentanan Broken Access Control menggunakan aplikasi DVWA dan OWASP Juice Shop. Semua langkah dijelaskan secara rinci agar dapat diikuti sebagai praktikum.

---

# 4.1 Eksploitasi IDOR (Insecure Direct Object Reference) — DVWA

IDOR adalah jenis BAC di mana aplikasi menggunakan parameter langsung (ID, user_id, nomor dokumen, dll.) tanpa validasi kepemilikan resource.

---

## **4.1.1 Tujuan**

* Menguji apakah pengguna dapat mengakses data pengguna lain hanya dengan mengubah parameter pada URL.
* Menganalisis respon aplikasi menggunakan OWASP ZAP.

---

## **4.1.2 Langkah Praktikum**

### **Langkah 1 — Login sebagai user biasa**

Akses DVWA → login:

```
Username: admin  
Password: password
```

Masuk ke menu:

```
Vulnerabilities → Insecure IDOR
```

Kamu akan melihat URL seperti:

```
http://localhost/dvwa/vulnerabilities/idor/?id=1
```

Halaman menampilkan data pengguna dengan ID 1.

---

### **Langkah 2 — Aktifkan OWASP ZAP Proxy**

Pastikan browser telah melalui:

```
Proxy: 127.0.0.1:8080
```

ZAP akan otomatis menangkap seluruh request.

---

### **Langkah 3 — Manipulasi Parameter (Eksploitasi IDOR)**

Ubah parameter `id` pada URL:

```
id=1 → id=2  
id=2 → id=3
```

Amati hasil:

* Jika muncul data pengguna lain → **IDOR Terdeteksi**.
* ZAP History akan menampilkan semua request yang berbeda.

---

### **Langkah 4 — Analisis pada OWASP ZAP**

Pada tab **History**:

* Pilih request `...?id=2`
* Buka tab **Response**
* Amati apakah aplikasi menampilkan data milik user lain.

ZAP juga memungkinkan:

* Menjalankan ulang request
* Modifikasi parameter via **Request Editor**
* Membandingkan respon sebelumnya vs sesudah modifikasi

---

### **4.1.3 Kesimpulan Eksploitasi IDOR**

Jika aplikasi menampilkan data pengguna lain melalui perubahan parameter sederhana, maka DVWA rentan terhadap IDOR.

---

---

# 4.2 Eksploitasi Horizontal Privilege Escalation

Eksploitasi ini fokus pada kemampuan seorang user untuk mengakses data user lain dalam konteks level privilege yang sama.

---

## **4.2.1 Tujuan**

* Menguji apakah user dapat mengambil alih akun lain.
* Mengetahui apakah session atau token divalidasi dengan benar.

---

## **4.2.2 Langkah Praktikum**

### **Langkah 1 — Simulasikan User A & User B**

Gunakan dua browser / dua sesi:

* **Browser A:** Login sebagai user A
* **Browser B:** Login sebagai user B

Masing-masing memiliki profil:

```
/user/profile.php?id=1  
/user/profile.php?id=2
```

---

### **Langkah 2 — Ambil Request User A di ZAP**

* Akses `/profile?id=1`
* Kirim ke **Repeater** (Send to Repeater)

---

### **Langkah 3 — Manipulasi ID menjadi ID user B**

Di Repeater, ubah:

```
id=1 → id=2
```

Klik **Send**.

### **Analisis Hasil:**

* Jika user A melihat data user B → Horizontal BAC / IDOR terjadi.
* Jika aplikasi tetap menampilkan hanya data user A → sistem aman.

---

# 4.3 Eksploitasi Vertical Privilege Escalation (Akses Admin)

Kerentanan terjadi jika user biasa dapat mengakses fungsi admin atau endpoint sensitif.

---

## **4.3.1 Tujuan**

* Menguji apakah ada URL admin yang tidak diproteksi.
* Mengetahui apakah role validasi terjadi hanya pada sisi frontend.

---

## **4.3.2 Langkah Praktikum**

### **Langkah 1 — Login sebagai user biasa**

Login DVWA sebagai:

```
admin / password
```

Ini user biasa, bukan admin sesungguhnya (simulasi pada DVWA).

---

### **Langkah 2 — Coba mengakses URL Admin**

Masukkan URL berikut secara manual:

```
http://localhost/dvwa/admin/
http://localhost/dvwa/admin/user-management.php
http://localhost/dvwa/admin/config.php
```

### **Analisis:**

* Jika halaman dapat terbuka → BAC vertical terkonfirmasi.
* Jika ditolak dengan HTTP 403 → aplikasi aman.

---

### **Langkah 3 — Manipulasi Role melalui Request**

Pada ZAP Repeater, ubah parameter:

```
role=user → role=admin
```

Atau:

```
is_admin=0 → is_admin=1
```

Jika server menerima dan memberikan akses admin, maka terjadi **vertical privilege escalation**.

---

# 4.4 Forced Browsing — Akses Langsung ke File Rahasia

Forced browsing adalah teknik di mana penyerang mencoba mengakses file atau folder yang tersembunyi secara langsung.

---

## **4.4.1 Tujuan**

* Mengetahui apakah folder sensitif dapat diakses tanpa autentikasi.

---

## **4.4.2 Langkah Praktikum**

Coba akses folder DVWA:

```
/config/  
/backup/  
/uploads/  
/secret/  
/dev/  
```

Jika ZAP menerima response **200 OK**, berarti:

* File dapat diakses tanpa autentikasi
* Konfigurasi file permission salah
* Ini termasuk BAC kategori “Unprotected Resource”

---

# 4.5 Eksploitasi Broken Access Control pada API — Juice Shop

OWASP Juice Shop memiliki API rentan untuk eksploitasi.

---

## **4.5.1 Tujuan**

* Melakukan eksploitasi API-level BAC
* Menguji kontrol akses pada endpoint REST

---

## **4.5.2 Langkah Praktikum (API Testing)**

### **Langkah 1 — Intersep Request Login**

Masuk ke Juice Shop → buka **Login**:

```
URL: POST /rest/user/login
```

ZAP menampilkan:

```
{ "email": "test@test.com", "password": "123456" }
```

---

### **Langkah 2 — Ambil JWT Token**

Response API akan mengembalikan token JWT:

```
"authentication": { "token": "eyJhbGciOi..." }
```

Simpan token ini.

---

### **Langkah 3 — Akses Endpoint Sensitif Menggunakan Token Biasa**

Coba akses endpoint admin:

```
GET /rest/admin/application-configuration
Authorization: Bearer <UserToken>
```

Analisis:

* Jika API mengembalikan data → vertical BAC terjadi.
* Jika API memblokir → aman.

---

### **Langkah 4 — Modifikasi Payload Token (JWT Tampering)**

Jika token tidak divalidasi dengan benar:

* Ubah role di JWT payload menjadi `admin`
* Encode ulang dengan base64

Jika server tidak memverifikasi signature → sukses bypass admin.

---

# 4.6 Ringkasan Bagian Eksploitasi

Pada bagian ini telah dilakukan eksploitasi:

* **IDOR (Horizontal Access Control Failure)**
* **Vertical Privilege Escalation**
* **Forced Browsing / Unprotected Resource**
* **API Broken Access Control (JWT Abuse)**

Eksploitasi dilakukan menggunakan DVWA & Juice Shop dengan bantuan OWASP ZAP.

Hasil eksploitasi ini akan digunakan untuk bagian berikutnya: **Analisis Kerentanan & Dampak Keamanan**.

# 5. Analisis Dampak & Risiko Keamanan

Pada bagian ini, kita akan melakukan analisis mendalam terhadap hasil eksploitasi Broken Access Control yang telah dilakukan pada DVWA dan OWASP Juice Shop. Analisis difokuskan pada dampak keamanan, risiko bagi organisasi, dan bagaimana kerentanan tersebut dapat dieksploitasi di dunia nyata.

---

# 5.1 Metode Analisis Risiko

Analisis risiko pada praktikum ini menggunakan pendekatan:

### **a. Identifikasi Kerentanan**

Menentukan jenis BAC yang ditemukan:

* IDOR
* Horizontal Privilege Escalation
* Vertical Privilege Escalation
* Forced Browsing
* API Authorization Bypass
* JWT Tampering

### **b. Vektor Serangan**

Bagaimana kerentanan dapat dieksploitasi oleh penyerang.

### **c. Dampak Keamanan (Impact)**

Menilai konsekuensi terhadap aplikasi dan pengguna:

* Confidentiality
* Integrity
* Availability

### **d. Skor Risiko (Severity)**

Menggunakan pendekatan OWASP Risk Rating:

* Low
* Medium
* High
* Critical

---

# 5.2 Analisis Dampak Per Jenis Eksploitasi

---

## **5.2.1 IDOR — Insecure Direct Object Reference**

### **Vektor Serangan**

Penyerang mengubah parameter ID pada URL:

```
?id=1 → ?id=2 → ?id=3 → ...
```

### **Dampak**

* Data pribadi pengguna dapat diakses tanpa autentikasi kuat.
* Penyerang dapat mengambil data sensitif seperti:

  * email
  * alamat
  * histori transaksi
  * biodata
* Dapat menyebabkan pelanggaran **Privacy** dan **Data Protection Law** (GDPR, UU PDP Indonesia).

### **Severity:** **High**

Karena memungkinkan pencurian massal data pengguna.

---

## **5.2.2 Horizontal Privilege Escalation**

### **Vektor Serangan**

User biasa mampu mengakses data user lain.

### **Dampak**

* Kebocoran data antar pengguna.
* Potensi impersonation (penyamaran identitas).
* Manipulasi profil pengguna lain.

### **Severity:** **High**

Sangat berbahaya pada aplikasi keuangan, perbankan, atau kampus.

---

## **5.2.3 Vertical Privilege Escalation**

### **Vektor Serangan**

User biasa memodifikasi role → menjadi admin.

Contoh:

```
role=user → role=admin  
is_admin=0 → is_admin=1
```

### **Dampak**

* Pengambilalihan panel admin.
* Penghapusan data/dokumen sistem.
* Manipulasi konfigurasi aplikasi.
* Penuh akses terhadap seluruh resource server.

### **Severity:** **Critical**

Kerentanan paling fatal karena memberikan kontrol penuh pada sistem aplikasi.

---

## **5.2.4 Forced Browsing (Unprotected Resource)**

### **Vektor Serangan**

Akses langsung folder sensitif:

```
/backup/  
/admin/  
/private/  
/uploads/
```

### **Dampak**

* Kebocoran file cadangan (backup.zip)
* Potensi terbukanya *source code*
* Ekspos API secret keys
* File upload tanpa limit → upload malware

### **Severity:** **Medium – High**

Tergantung sensitifitas file yang ditemukan.

---

## **5.2.5 API Broken Access Control — Juice Shop**

### **Vektor Serangan**

User mencoba mengakses endpoint admin menggunakan token biasa:

```
GET /rest/admin/application-configuration  
Authorization: Bearer <UserToken>
```

Jika berhasil, berarti ada authorization flaw.

### **Dampak**

* Pengambilalihan data melalui API
* Pemalsuan request API
* Dump database jika endpoint sensitif terbuka
* Manipulasi konfigurasi server

### **Severity:** **Critical**

Karena API biasanya terhubung langsung ke database internal.

---

## **5.2.6 JWT Tampering (Token Manipulation)**

Jika signature token tidak divalidasi:

* Penyerang dapat decode token
* Mengubah payload → `role=admin`
* Encode ulang tanpa signature valid

### **Dampak**

* Akses admin penuh
* Pengambilalihan akun tingkat tinggi
* Bypass seluruh mekanisme autentikasi frontend

### **Severity:** **Critical**

Masuk kategori high-impact API-level BAC.

---

# 5.3 Dampak Terhadap Organisasi

Broken Access Control dapat menyebabkan:

### **1. Kebocoran Data Massal**

Termasuk:

* biodata pengguna
* nomor identitas
* data akademik
* riwayat transaksi
* file internal

### **2. Kerugian Finansial**

* tuntutan hukum
* biaya investigasi forensik
* kerusakan reputasi

### **3. Pengambilalihan Sistem**

Penyerang dapat menghapus, memodifikasi, atau mencuri seluruh database.

### **4. Pelanggaran Regulasi**

Termasuk:

* **GDPR (EU)**
* **UU 27 Tahun 2022 tentang PDP (Indonesia)**

### **5. Hilangnya Kepercayaan Pengguna**

Aplikasi dianggap tidak aman → pengguna meninggalkan platform.

---

# 5.4 Kesimpulan Analisis Risiko

Broken Access Control termasuk salah satu kerentanan paling berbahaya dalam OWASP Top 10 karena:

* Sangat mudah dieksploitasi bahkan oleh pemula
* Dampaknya mencakup kebocoran data skala besar
* Sering terjadi akibat validasi authorization yang lemah
* Vertical privilege escalation dan JWT tampering dapat memberikan akses admin penuh

Pemahaman terhadap dampak BAC sangat penting bagi developer, auditor, dan mahasiswa agar dapat menerapkan mekanisme kontrol akses yang aman.

---

# 5. Pengujian Broken Access Control Menggunakan OWASP ZAP

Bagian ini berisi langkah detail melakukan pengujian Broken Access Control menggunakan **OWASP ZAP** dengan teknik:

* Intercept Request
* Manipulasi Parameter (IDOR / Vertical / Horizontal Access Control)
* Forced Browsing
* Fuzzing Parameter
* Menggunakan fitur Spider & Active Scan secara terkontrol

---

# 5.1 Persiapan Sebelum Pengujian

Sebelum memulai proses eksploitasi, pastikan:

1. **Browser telah terhubung dengan ZAP Proxy**

   * Proxy ZAP biasanya di:

     ```
     127.0.0.1:8080
     ```
   * Pastikan ZAP menampilkan trafik HTTP dari aplikasi (DVWA / Juice Shop).

2. **Aplikasi target dalam mode low/medium security**
   Baik DVWA maupun Juice Shop harus dalam kondisi:

   * Login sebagai user biasa (bukan admin)
   * Mengaktifkan fitur yang memuat parameter ID

3. **Session Management Aktif**

   * Gunakan fitur “Use Authentication” jika diperlukan.

---

# 5.2 Intercept Request & Modifikasi Parameter

Pada tahap ini, kita akan melakukan teknik dasar untuk menguji BAC:

### Langkah:

1. Buka aplikasi target (contoh DVWA – Vulnerabilities → IDOR).
2. Klik salah satu item, misalnya:

   ```
   http://localhost/dvwa/vulnerabilities/idor/?id=1
   ```
3. ZAP akan menampilkan request di panel *History*.
4. Klik kanan → **"Break"** untuk melakukan intercept.
5. Ubah parameter ID:

   ```
   id=1 → id=2 → id=3 → dst.
   ```
6. Klik "Continue" di ZAP.

### Tujuan:

* Menguji apakah server memvalidasi kepemilikan data.
* Mengidentifikasi IDOR atau Horizontal Access Control Failure.

### Hasil yang Diharapkan:

Jika berhasil, server akan menampilkan data milik user lain.

---

# 5.3 Pengujian Vertical Access Control

### Kasus:

User biasa mencoba mengakses fungsi admin.

### Langkah:

1. Login sebagai user "regular" di DVWA/Juice Shop.
2. Buka:

   ```
   /admin
   /administrator
   /admin/users
   /api/admin
   ```
3. Periksa respons:

   * `200 OK` → kerentanan **Vertical Access Control**
   * `302 Redirect` → cek apakah redirect dapat dilewati
   * `403 Forbidden` → aman

### Teknik Bypass:

* Ubah role/cookie secara manual melalui ZAP.
* Modifikasi header:

  ```
  X-User-Role: admin
  ```
* Tambahkan admin=1 pada parameter.

---

# 5.4 Forced Browsing Menggunakan ZAP

Forced browsing bertujuan menemukan direktori/file yang tidak ter-link dalam UI.

### Langkah:

1. Klik tab **"Forced Browse (DirBuster)"**.
2. Pilih target → klik **Start Scan**.
3. Pilih wordlist bawaan:

   * directory-list-lowercase-2.3-medium.txt

### Hasil yang dicari:

* `/backup.zip`
* `/config.php.bak`
* `/admin/`
* `/private/`

Jika dapat diakses tanpa autentikasi → kerentanan **Forced Browsing**.

---

# 5.5 Fuzzing Parameter (ID Enumeration)

Teknik fuzzing digunakan untuk menemukan ID yang valid atau sensitif.

### Langkah:

1. Pilih request yang memuat parameter ID.
2. Klik kanan → **Attack → Fuzz**.
3. Pilih parameter:

   ```
   id
   user_id
   productId
   order
   ```
4. Tambahkan payload:

   * Payload → Number Generator → 1–500
5. Jalankan Fuzzing.

### Hasil yang Diharapkan:

* Respons dengan ukuran berbeda (indikasi data berbeda)
* HTTP Code 200 untuk ID yang tidak seharusnya diakses
* Informasi sensitif (nama, email, transaksi)

---

# 5.6 Spidering Struktur Aplikasi

Spider berguna memetakan halaman yang mungkin terlewat.

### Langkah:

1. Klik kanan pada domain → **Attack → Spider**.
2. Izinkan ZAP merayapi semua link.

### Tujuan:

* Menemukan endpoint tersembunyi yang dapat mengarah ke BAC.

---

# 5.7 Active Scan (Pengujian Otomatis)

❗ **Catatan:** Gunakan Active Scan hanya pada aplikasi lab (DVWA / Juice Shop).
Jangan gunakan pada sistem real karena bersifat agresif.

### Langkah:

1. Klik domain → **Attack → Active Scan**.
2. Tunggu hasil pada tab Alerts.

### Hasil Penting:

* IDOR
* Missing Role Validation
* Unrestricted File Access
* Path Traversal
* Access control issues

---

# 5.8 Dokumentasi Bukti Uji

Pada setiap kerentanan yang ditemukan, dokumentasikan:

1. Screenshot request/response di ZAP
2. URL yang dimodifikasi
3. Payload yang digunakan (ID fuzzing, request manipulations)
4. Dampak yang terlihat
5. Potensi risiko (High, Medium, Low)

---

# 5.9 Kesimpulan Pengujian

Setelah pengujian dengan OWASP ZAP, biasanya ditemukan:

* Data user lain dapat diakses → IDOR
* User biasa dapat membuka halaman admin → Vertical Access Control Failure
* Direktori sensitif dapat diakses → Forced Browsing
* Endpoint API tidak memverifikasi kepemilikan resource → Insecure Direct Object Reference
* Tidak ada pengecekan role pada server-side → Broken Access Control

---

# 6. Rekomendasi Mitigasi & Perbaikan Keamanan Broken Access Control

Bagian ini menjelaskan langkah mitigasi komprehensif berdasarkan hasil pengujian pada DVWA dan OWASP Juice Shop. Semua rekomendasi mengacu pada standar OWASP ASVS (Application Security Verification Standard) dan OWASP Top 10.

---

# 6.1 Prinsip Utama Mitigasi Broken Access Control

Untuk mencegah Broken Access Control, aplikasi harus menerapkan:

### **1. Server-Side Authorization Enforcement**

Tidak boleh hanya memeriksa hak akses di frontend atau Javascript.

### **2. Zero Trust Access**

Setiap request harus divalidasi kembali hak aksesnya, tanpa pengecualian.

### **3. Defense in Depth**

Kontrol akses harus berlapis: per halaman, per fungsi, per objek, dan per API.

---

# 6.2 Rekomendasi Mitigasi Berdasarkan Jenis Kerentanan

---

## **6.2.1 Mitigasi untuk IDOR**

IDOR adalah salah satu temuan utama baik di DVWA maupun Juice Shop.

### **Solusi:**

1. **Validasi Ownership di Backend**

   ```pseudo
   if resource.owner_id != session.user_id:
       return 403 Forbidden
   ```

2. **Gunakan Indirect Object References**
   Mengganti ID asli dengan UUID/random token.

   * 123 → `9f71c0d1-52a1-4b0c-af26-e93a2c8f4a56`

3. **Implementasi Access Control Matrix**
   Setiap resource memiliki aturan siapa yang berhak mengaksesnya.

4. **Gunakan API Gateway yang memiliki Access Policy**
   Misalnya:

   * Kong API Gateway
   * NGINX Ingress Rules
   * AWS IAM Policy untuk API Gateway

---

## **6.2.2 Mitigasi Horizontal Access Control Failure**

### **Solusi:**

* Validasi apakah pengguna hanya dapat mengakses datanya sendiri.
* Gunakan session-based ownership checking.
* Jangan sertakan parameter yang dapat dimanipulasi untuk memilih data user lain.
* Gunakan query berbasis user context:

```sql
SELECT * FROM orders WHERE user_id = :session_user_id
```

---

## **6.2.3 Mitigasi Vertical Access Control Failure**

### **Solusi:**

### **a. Implement Role-Based Access Control (RBAC)**

Contoh role:

* User
* Moderator
* Admin

### **b. Validasi Role di Server**

Contoh pseudocode:

```pseudo
if session.role != "admin":
    return 403
```

### **c. Hilangkan endpoint admin dari UI user biasa**

* Jangan tampilkan menu admin pada user biasa.
* Validasi tetap wajib dilakukan di backend.

### **d. Gunakan Permission Matrix**

Contoh:

| Fungsi             | User | Admin |
| ------------------ | ---- | ----- |
| Lihat Data Sendiri | ✓    | ✓     |
| Lihat Semua Data   | ✗    | ✓     |
| Kelola Pengguna    | ✗    | ✓     |

---

## **6.2.4 Mitigasi Forced Browsing**

### **Solusi:**

1. Blokir akses langsung ke direktori sensitif:

   ```
   /backup/
   /private/
   /config/
   ```

2. Gunakan file `.htaccess` (Apache):

   ```
   Deny from all
   ```

3. Untuk NGINX:

   ```
   location /backup/ {
       deny all;
   }
   ```

4. Simpan file sensitif *di luar* root direktori web.

5. Aktifkan “Directory Listing Off”.

---

## **6.2.5 Mitigasi API Authorization Bypass**

API Juice Shop banyak menggunakan endpoint yang dapat disalahgunakan jika kontrol lemah.

### **Solusi:**

1. Validasi hak akses berdasarkan token JWT.

2. Jangan bergantung pada data role dari frontend.

3. Gunakan middleware khusus authorization:

   * Express.js → `express-jwt` + custom RBAC middleware
   * Laravel → Gates & Policies
   * Django → DRF Permissions

4. Gunakan rate limiting:

   ```
   100 requests / minute / user
   ```

5. Gunakan API Firewall:

   * Wallarm
   * Cloudflare API Shield

---

## **6.2.6 Mitigasi JWT Tampering**

### **Solusi:**

1. Gunakan **HS256** atau **RS256** dengan secret key kuat (≥ 32 karakter).
2. Validasi signature dengan benar pada server.
3. Jangan pernah mengizinkan algoritma:

   ```
   alg: none
   ```
4. Jangan simpan role sensitif hanya di token → lakukan revalidasi role di server.
5. Atur JWT Expiry pendek:

   ```
   exp: 15 minutes
   ```

---

# 6.3 Praktik Terbaik (Best Practices) Anti-Broken Access Control

### **1. Implementasi Access Control di Server**

Tidak boleh hanya berdasarkan UI.

### **2. Gunakan Security Framework**

* Spring Boot Security
* Laravel Authorization Policy
* Express.js RBAC Middleware

### **3. Gunakan Parameter Binding Aman**

Hindari query ID manual yang dapat dimanipulasi.

### **4. Logging & Monitoring**

Log setiap aktivitas seperti:

* akses file sensitif
* perubahan parameter ID
* error 403 yang berulang

### **5. Security Testing Rutin**

Gunakan:

* OWASP ZAP
* Burp Suite
* Nikto
* Dependency Checker

---

# 6.4 Kesimpulan Mitigasi

Penerapan mitigasi Broken Access Control harus dilakukan pada **level aplikasi, server, dan API** secara konsisten.
Kontrol akses tidak boleh ditaruh di satu lapisan saja.
Dengan menerapkan mitigasi yang benar, risiko kebocoran data dan pengambilalihan akun dapat dikurangi secara signifikan.

---

# 7. Kesimpulan Akhir

Pada praktikum ini, telah dilakukan serangkaian pengujian terhadap kerentanan **Broken Access Control** menggunakan dua aplikasi vulnerable, yaitu **DVWA (Damn Vulnerable Web Application)** dan **OWASP Juice Shop**. Melalui eksploitasi berbagai bentuk BAC, diperoleh pemahaman mendalam tentang bagaimana kelemahan kontrol akses dapat memengaruhi keamanan aplikasi modern.

---

# 7.1 Rangkuman Temuan Utama

Pengujian berhasil mengidentifikasi beberapa jenis Broken Access Control, antara lain:

### **1. IDOR (Insecure Direct Object Reference)**

* Sangat mudah dieksploitasi hanya dengan mengganti parameter ID.
* Berpotensi membuka data sensitif pengguna lain.

### **2. Horizontal Privilege Escalation**

* User biasa dapat mengakses data milik user lain.
* Membuktikan tidak adanya validasi kepemilikan resource.

### **3. Vertical Privilege Escalation**

* User dapat memodifikasi role menjadi admin.
* Berisiko menyebabkan pengambilalihan seluruh sistem.

### **4. Forced Browsing**

* Direktori sensitif seperti `/backup/`, `/admin/`, dan `/uploads/` dapat diakses tanpa autentikasi.

### **5. API Authorization Bypass**

* Endpoint admin pada Juice Shop dapat dipanggil menggunakan token user biasa.

### **6. JWT Tampering**

* Token dapat dimodifikasi untuk mengubah hak akses.

Semua temuan ini menunjukkan bahwa kontrol akses yang tidak terimplementasi dengan baik dapat menyebabkan kebocoran data maupun kompromi sistem secara menyeluruh.

---

# 7.2 Dampak yang Dihasilkan dari Eksploitasi

Berdasarkan analisis risiko:

* **Dampak Confidentiality**: Data pengguna terancam bocor.
* **Dampak Integrity**: Data dapat dimodifikasi tanpa izin.
* **Dampak Availability**: Penyerang berpotensi merusak atau menghapus data.

Eksploitasi Broken Access Control sering kali berujung pada **Critical Severity**, terutama untuk kasus vertical privilege escalation dan manipulasi JWT.

---

# 7.3 Evaluasi Terhadap Sistem yang Diuji

Dari hasil eksploitasi DVWA dan Juice Shop dapat disimpulkan:

* Aplikasi dengan kontrol akses lemah mudah diserang dengan teknik sederhana.
* API membutuhkan mekanisme autentikasi dan otorisasi yang lebih ketat.
* Developer sering kali hanya mengandalkan pemeriksaan di frontend — ini merupakan kesalahan fatal.

---

# 7.4 Pelajaran yang Dipelajari

Praktikum ini memberikan wawasan penting bahwa:

### **1. Kontrol Akses Harus Diperiksa di Server**

Frontend tidak dapat dipercaya sepenuhnya.

### **2. Hak Akses Harus Divalidasi per Request**

Setiap request harus memastikan apakah user berhak mengakses data tersebut.

### **3. Keamanan Aplikasi Tidak Hanya Tentang Autentikasi**

Authorization jauh lebih penting dalam mencegah penyalahgunaan.

### **4. Pengujian Keamanan Harus Berbasis Skenario Nyata**

Simulasi attacker memberikan pemahaman yang lebih akurat terhadap risiko.

---

# 7.5 Kesimpulan Umum

Broken Access Control merupakan salah satu kerentanan paling berbahaya dalam OWASP Top 10 karena:

* Sering terjadi akibat kesalahan logika sederhana.
* Sangat mudah dieksploitasi oleh attacker.
* Dampaknya mencakup akses penuh terhadap sistem.
* Dapat menyebabkan kebocoran data skala besar dan kerugian organisasi.

Melalui praktikum ini, mahasiswa diharapkan memahami bagaimana cara menemukan, mengeksploitasi, dan memitigasi BAC sehingga dapat membangun aplikasi yang lebih aman.

---

# 7.6 Rekomendasi Lanjutan

Untuk mendalami materi lebih jauh, disarankan untuk:

* Menguji aplikasi nyata menggunakan Burp Suite Professional.
* Mempraktikkan OWASP ASVS, terutama pada bagian Authorization.
* Melakukan red team/blue team exercise.
* Membangun role-based access control (RBAC) pada proyek pribadi.

---

**Laporan praktikum ini menjadi dasar untuk memahami kelemahan paling kritis pada aplikasi modern, serta bagaimana menerapkan kontrol akses yang benar sebagai langkah pencegahan utama.**
