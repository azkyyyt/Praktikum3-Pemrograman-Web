# Product Manager - Integrasi PHP, MySQL & UI Styling

Aplikasi web manajemen inventoris produk sederhana yang dibangun menggunakan **PHP Native (PDO)**, **MySQL**, serta **CSS UI Styling (Box Model & Flexbox)**. 

Aplikasi ini dikembangkan sesuai dengan kriteria dan standar pada **Praktikum Pemrograman Web - Pertemuan 3**, mengutamakan validasi data di sisi server, pencegahan submit ganda melalui pola PRG, serta aspek keamanan dari ancaman *SQL Injection*, *Cross-Site Scripting (XSS)*, dan *Cross-Site Request Forgery (CSRF)*.

---

##  Struktur Direktori Proyek

```text
product-manager/
├── config/
│   └── db.php              # File koneksi PDO ke database MySQL
├── database/
│   └── store_db.sql        # Schema database dan tabel MySQL
├── public/
│   ├── assets/
│   │   └── style.css       # Styling CSS (Box Model & Flexbox responsif)
│   ├── index.php           # READ: Menampilkan daftar produk (Card Flexbox) + Notifikasi PRG
│   ├── create.php          # CREATE: Form tambah produk + Validasi Server-Side + PRG
│   ├── edit.php            # UPDATE: Form edit produk berdasarkan ID + Validasi + PRG
│   └── delete.php          # DELETE: Penghapusan produk + Proteksi CSRF Token + PRG
└── README.md               # Dokumentasi lengkap dan cara menjalankan proyek
```

---

##  Persyaratan Sistem (Prerequisites)

- **Web Server:** Apache (via XAMPP / Laragon / Web Server Lokal)
- **Database Server:** MySQL / MariaDB (versi 5.7+ atau 8.x)
- **PHP:** Versi 7.4 atau versi 8.x (dengan ekstensi `pdo_mysql` diaktifkan)
- **Browser:** Google Chrome, Mozilla Firefox, Microsoft Edge, atau browser modern lainnya

---

##  Panduan Cara Menjalankan Aplikasi

### Langkah 1: Menyalin Berkas Proyek
Salin seluruh folder `product-manager` ke dalam direktori publik web server Anda:
- **XAMPP:** `C:\xampp\htdocs\product-manager`
- **Laragon:** `C:\laragon\www\product-manager`

### Langkah 2: Menyalakan Web Server & Database
Buka **XAMPP Control Panel** (atau Laragon), lalu tekan tombol **Start** pada baris **Apache** dan **MySQL**.

### Langkah 3: Mengimpor Database MySQL
Pilih salah satu cara berikut untuk mengimpor schema database:

#### Cara A: Melalui phpMyAdmin (Visual)
1. Buka browser dan akses `http://localhost/phpmyadmin`.
2. Klik tab **SQL**.
3. Buka file `database/store_db.sql`, salin seluruh isinya, tempel pada kolom SQL phpMyAdmin, lalu klik **Kirim / Go**.

#### Cara B: Melalui Terminal / PowerShell (CLI)
Buka Terminal/PowerShell pada folder proyek dan jalankan perintah berikut:
```powershell
Get-Content database/store_db.sql | C:\xampp\mysql\bin\mysql.exe -u root
```

### Langkah 4: Konfigurasi Database (Jika Diperlukan)
Pengaturan koneksi secara bawaan berada pada file `config/db.php`:
- **Host:** `localhost`
- **Database Name:** `store_db`
- **Username:** `root`
- **Password:** `""` *(kosong)*

Jika pengaturan MySQL lokal Anda menggunakan password, sesuaikan variabel `$pass` pada file `config/db.php`.

### Langkah 5: Mengakses Aplikasi
Buka browser dan navigasi ke alamat berikut:
 **`http://localhost/product-manager/public/`**

---

##  Fitur Teknis & Konsep Keamanan yang Diimplementasikan

### 1. Form Validation (Server-Side)
Validasi di sisi klien (HTML) dapat dengan mudah dilewati. Oleh karena itu, seluruh input divalidasi ulang di PHP (`create.php` & `edit.php`):
- **Nama Produk:** Wajib diisi, minimal 3 karakter (`mb_strlen`), dan diperiksa keunikannya terhadap database.
- **Kategori:** Wajib diisi (default bernilai `"Umum"` jika kosong).
- **Harga Produk:** Memakai `FILTER_VALIDATE_FLOAT` dan dipastikan bernilai lebih besar dari 0 (`price > 0`).
- **Stok Produk:** Memakai `FILTER_VALIDATE_INT` dan dipastikan tidak negatif (`stock >= 0`).

### 2. Pola Post-Redirect-Get (PRG)
Untuk mencegah pengiriman ulang data (*double form submission*) ketika pengguna melakukan refresh halaman (F5) setelah menambah/mengubah/menghapus produk:
- Setelah proses mutasi data (POST) berhasil, server melakukan pengalihan halaman via header HTTP:
  ```php
  header("Location: index.php?status=created");
  exit;
  ```
- Halaman `index.php` membaca parameter `$_GET['status']` untuk menampilkan notifikasi pesan sukses secara aman.

### 3. Keamanan Database (Anti SQL Injection)
Menggunakan API **PDO** dengan *Native Prepared Statements*:
```php
$stmt = $pdo->prepare("INSERT INTO products (name, category, price, stock) VALUES (:name, :category, :price, :stock)");
$stmt->execute(["name" => $name, "category" => $category, "price" => $price, "stock" => $stock]);
```
Instruksi SQL dipisahkan secara murni dari variabel data pengguna sehingga input tidak dapat ditafsirkan sebagai sintaks query SQL.

### 4. Pencegahan Cross-Site Scripting (XSS)
Seluruh data teks dari database yang ditayangkan pada template HTML di-escape menggunakan fungsi `htmlspecialchars()`:
```php
<h3><?= htmlspecialchars($p["name"], ENT_QUOTES, "UTF-8") ?></h3>
```
Jika pengguna memasukkan input berisi skrip seperti `<b>Promo</b>` atau `<script>alert('xss')</script>`, teks tersebut akan dicetak sebagai string aman dan tidak dieksekusi sebagai markup/skrip HTML.

### 5. Proteksi CSRF (Cross-Site Request Forgery) pada Aksi Hapus
Aksi penghapusan data pada `delete.php`:
- Memeriksa bahwa request dikirim hanya melalui metode **POST**.
- Membuat token acak `$_SESSION['csrf'] = bin2hex(random_bytes(32))` dan menyisipkannya pada form hapus sebagai hidden input.
- Memvalidasi token form dengan token session menggunakan `hash_equals()` sebelum eksekusi `DELETE`.

### 6. UI Styling Responsif (Box Model & Flexbox)
- **Box Model:** Menggunakan `*, *::before, *::after { box-sizing: border-box; }` agar perhitungan `width` elemen mencakup `padding` dan `border`.
- **Flexbox Grid:** Menggunakan `display: flex; flex-wrap: wrap; gap: 20px;` pada kontainer `.products` dan `flex: 1 1 280px;` pada kartu `.card` sehingga tampilan kartu produk otomatis menyesuaikan lebar layar dari desktop hingga perangkat seluler.

---

##  Skenario Pengujian (Testing Checklist)

| No | Kasus Pengujian | Tindakan | Hasil yang Diharapkan |
|---|---|---|---|
| 1 | **Tambah Produk Valid** | Isi Nama: "Sepatu Lari", Harga: 250000, Stok: 10 $\rightarrow$ Simpan | Produk berhasil disimpan dan muncul di kartu produk `index.php`. |
| 2 | **Validasi Nama Pendek** | Isi Nama: "Ab", Harga: 10000, Stok: 5 $\rightarrow$ Simpan | Ditolak oleh PHP dengan pesan error *"Nama produk minimal 3 karakter"*. |
| 3 | **Validasi Harga / Stok Negatif** | Isi Harga: -5000 atau Stok: -2 $\rightarrow$ Simpan | Ditolak oleh PHP dengan pesan error validasi yang sesuai. |
| 4 | **Pencegahan Nama Duplikat** | Tambah produk dengan nama yang sudah ada di database | Ditolak dengan pesan *"Nama produk sudah digunakan"*. |
| 5 | **Uji Ulang Refresh (Anti PRG)** | Lakukan tambah produk, lalu tekan F5 / Refresh pada `index.php` | Tidak ada data produk yang terduplikasi. |
| 6 | **Uji Keamanan XSS** | Masukkan nama produk: `<b>Kopi Super</b>` | Teks `<b>Kopi Super</b>` tampil sebagai karakter tulisan murni, bukan teks tebal. |
| 7 | **Edit Produk** | Klik tombol Edit pada salah satu produk, ubah stok/harga $\rightarrow$ Perbarui | Data produk di database dan tampilan diperbarui. |
| 8 | **Hapus Produk (CSRF)** | Klik tombol Hapus pada kartu produk $\rightarrow$ Konfirmasi dialog | Produk terhapus dari database dengan notifikasi sukses. |

---

##  Identitas Pembuat
- **Mata Kuliah:** Pemrograman Web (Pertemuan 3)
- **Proyek:** Mini Project Final - Product Manager
