# Panduan Merubah APG versi HTML Menjadi PHP

>**Daftar isi**
>- [Panduan Merubah APG versi HTML Menjadi PHP](#panduan-merubah-apg-versi-html-menjadi-php)
>- [Membuat database](#membuat-database)
>- [Membuat koneksi ke database](#membuat-koneksi-ke-database)
>- [Membuat file login dan logout](#membuat-file-login-dan-logout)
>- [Membuat halaman dashboard](#membuat-halaman-dashboard)
>- [Membuat halaman jabatan](#membuat-halaman-jabatan)
>- [Membuat halaman pegawai](#membuat-halaman-pegawai)

## Membuat database

Sebelum memulai, baiknya kita siapkan database, struktur tabel beserta sample datanya.

```sql
-- Create and select database
CREATE DATABASE apg;
USE apg;

-- Dumping structure for table apg.jabatan
CREATE TABLE IF NOT EXISTS `jabatan` (
  `id_jabatan` int(5) NOT NULL AUTO_INCREMENT,
  `nama_jabatan` varchar(20) NOT NULL,
  PRIMARY KEY (`id_jabatan`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=latin1;

-- Dumping data for table apg.jabatan: ~4 rows (approximately)
INSERT INTO `jabatan` (`id_jabatan`, `nama_jabatan`) VALUES
	(1, 'Direktur'),
	(2, 'Manajer'),
	(3, 'Marketing'),
	(4, 'Sekretaris');

-- Dumping structure for table apg.pegawai
CREATE TABLE IF NOT EXISTS `pegawai` (
  `id_pegawai` int(5) NOT NULL AUTO_INCREMENT,
  `nama_pegawai` varchar(30) NOT NULL,
  `jenis_kelamin` varchar(1) DEFAULT NULL,
  `tgl_lahir` date DEFAULT NULL,
  `foto` varchar(50) DEFAULT NULL,
  `keterangan` text,
  `id_jabatan` int(5) DEFAULT NULL,
  PRIMARY KEY (`id_pegawai`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=latin1;

-- Dumping data for table apg.pegawai: ~1 rows (approximately)
INSERT INTO `pegawai` (`id_pegawai`, `nama_pegawai`, `jenis_kelamin`, `tgl_lahir`, `foto`, `keterangan`, `id_jabatan`) VALUES
	(1, 'Ipnu Masyaid', 'L', '1994-11-08', 'user1.png', '', 1);

-- Dumping structure for table apg.user
CREATE TABLE IF NOT EXISTS `user` (
  `id_user` int(5) NOT NULL AUTO_INCREMENT,
  `nama_lengkap` varchar(50) DEFAULT NULL,
  `username` varchar(20) DEFAULT NULL,
  `password` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id_user`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=latin1;

-- Dumping data for table apg.user: ~1 rows (approximately)
INSERT INTO `user` (`id_user`, `nama_lengkap`, `username`, `password`) VALUES
	(1, 'Ipnu Masyaid', 'admin', '21232f297a57a5a743894a0e4a801fc3');
```

## Membuat koneksi ke database

Kita buat folder `library` terlebih dahulu, lalu buat file `config.php` di dalamnya.

```php
<?php
$host = "localhost";
$user = "root";
$pass = "root";
$db = "apg";

$con = mysqli_connect($host, $user, $pass, $db);

if (mysqli_connect_errno()) {
    echo "Koneksi gagal! : " . mysqli_connect_error();
}
?>
```
>**Petunjuk**
>- Sekarang coba jalankan server php dengan cara masuk ke dalam `root` folder project dan menggunakan terminal, ketikkan `php -S localhost:80` dan <kbd>ENTER</kbd>
>- Buka browser dan ketikkan `localhost/library/config.php`
>- Jika tidak ada salah ketik dan server berjalan normal, harusnya tapil halaman kosong.

## Membuat file `login` dan `logout`

Ubah file `login.html` menjadi `login.php` tanpa mengubah skrip di dalamnya.

Berikut kodenya

```php
<!DOCTYPE html>
<html lang="en">
<head>
	<title>Login Aplikasi</title>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="stylesheet" href="css/style.css">
</head>
<body>
	<div class="container">
		<section class="login-box">
			<h2>Login Aplikasi</h2>
			<form action="ceklogin.php" method="post">
				<input type="text" placeholder="Username" id="username" name="username">
				<input type="password" placeholder="Password" id="password" name="password">
				<input type="submit" value="Login">
			</form>
		</section>
	</div>
</body>
</html>
```

Jika diperhatikan ada baris kode
```php
<form action="ceklogin.php" method="post">
```
itu artinya jika tombol `login` diklik, maka data username dan password akan dikirim dan diproses ke file `ceklogin.php`, untuk dilakukan pencocokan antara username dan password yang ada di dalam database.

>**Perhatian**
>Jangan lupa copy folder `css` beserta isinya ke dalam root folder project agar tampilannya lebih bagus.

Sekarang kita buat file `ceklogin.php`, berikut kodenya.

```php
<?php
    session_start();
    include "library/config.php";

    $username = $_POST['username'];
    $password = md5($_POST['password']);

    $query = mysqli_query($con,
    "SELECT * FROM user WHERE username = '$username' AND password = '$password' ");
    $data = mysqli_fetch_array($query);
    $jml = mysqli_num_rows($query);

    if ($jml > 0) {
        $_SESSION['username'] = $data['username'];
        $_SESSION['password'] = $data['password'];

        header('location: index.php');
    } else {
        echo "<p align='center'>Login Gagal!</p>";
        echo "<meta http-equiv='refresh' content='2; url=login.php'>";
    }
?>
```
>**Penjelasan**
>- Skrip di atas akan mengecek, apakah ada data pada tabel user dengan username dan password sesuai yang dimasukkan.
>- Password dienskripsi menggunakan fungsi `md5()`.
>- Jika data ditemukan, yang ditandai dengan jumlah data lebih dari 0, meka akan membuat `session` username dan password untuk digunakan sebagai penanda pada halaman lain bahwa user sudah login.
>- Function `header()` akan mengarahkan ke halama `index.php`

Selanjutnya kita buat file `logout.php`

```php
<?php
    session_start();
    session_destroy();

    echo "<p align='center'>Anda telah logout!</p>";
    echo "<meta http-equiv='refresh' content='2; url=login.php'>";
?>
```

>**Penjelasan**
>Fungsi `destroy()` akan menghapus data `session` username dan password yang dibuat pada file `ceklogin.php`


## Membuat halaman `dashboard`

Untuk membuat halaman dashboard dibutuhkan 3 file, yaitu:

- `index.php` yang berisi template yang digunakan oleh semua halaman,
- `konten.php` yang berisi skrip untuk mengatur halaman mana yagn akan ditampilkan, dan
- `dashboard.php` yang akan ditampilkan pada bagian konten.

Mulai dari mengubah file `dashboard.html` menadi `index.php` dan rubah skripnya menjadi berikut.

```php
<?php
    session_start();
    ob_start();

    include "library/config.php";

    if (empty($_SESSION['username']) OR empty($_SESSION['password'])) {
        echo "<p align='center'>Anda harus login terlebih dahulu!</p>";
        echo "<meta http-equiv='refresh' content='2; url=login.php'>";
    } else {
        define('INDEX', true);
?>

<!DOCTYPE html>
<html lang="en">
<head>
	<title>Dashboard</title>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="stylesheet" href="css/style.css">
</head>
<body>
	<header>
		Aplikasi Manajemen Pegawai
	</header>
	<div class="container">
		<aside>
			<ul class="menu">
				<li><a href="?hal=dashboard">Dashboard</a></li>
				<li><a href="?hal=pegawai">Data Pegawai</a></li>
				<li><a href="?hal=jabatan">Data Jabatan</a></li>
				<li><a href="logout.php">Keluar</a></li>
			</ul>
		</aside>
		<section class="main">
			<?php include "konten.php" ?>
		</section>
	</div>
	<footer>
		Copyright &copy; <b>Masipnu</b> Official
	</footer>
</body>
</html>

<?php
}
?>
```


File `konten.php`

```php
<?php
    if (!defined('INDEX')) die();

    $halaman = [
        "dashboard",
        "pegawai",
        "pegawai_tambah",
        "pegawai_insert",
        "pegawai_edit",
        "pegawai_update",
        "pegawai_hapus",
        "jabatan",
        "jabatan_tambah",
        "jabatan_insert",
        "jabatan_edit",
        "jabatan_update",
        "jabatan_hapus"
    ];

    if (isset($_GET['hal'])) {
        $hal = $_GET['hal'];
    } else {
        $hal = 'dashboard';
    }

    foreach($halaman as $h){
        if($hal == $h){
            include "content/$h.php";
            break;
        }
    }
?>
```

Dan berikut skrip file `dashboard`, diletakkan di dalam folder `content`.

```php
<?php
    if(!defined('INDEX')) die("");
?>

<h1>Selamat Datang di Aplikasi Manajemen Pegawai</h1>
<h3>Anda login sebagai <b>Administrator</b></h3>
```
>**Perhatian**
>- Sekarang coba buka browser kembali, ketikkan `localhost` lalu <kbd>ENTER</kbd>, tentu ada akan diarahkan ke halaman `login`.
>- Coba masukkan `admin` untuk username dan passwordnya, lalu <kbd>ENTER</kbd>.
>- <kbd>Selamat 😎</kbd>


## Membuat halaman `jabatan`

Buat file-file berikut di dalam folder `content`.

- `jabatan.php`
- `jabatan_tambah.php`
- `jabatan_insert.php`
- `jabatan_edit.php`
- `jabatan_update.php`
- `jabatan_hapus.php`

Berikut kode untuk masing-masing filenya.

**jabatan.php**

Silahkan modifikasi dari file `tabel.html`.

```php
<?php
if(!defined('INDEX')) die("");
?>

<h2 class="judul">Data Jabatan</h2>
<a href="?hal=jabatan_tambah" class="tombol">Tambah</a>

<table class="table">
    <thead>
        <tr>
            <th>No</th>
            <th>Nama Jabatan</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
        <?php
        $query = "SELECT * FROM jabatan ORDER BY id_jabatan DESC";
        $result = mysqli_query($con,$query);
        $no = 0;
        while($data = mysqli_fetch_array($result)){
            $no++;
        ?>
            <tr>
                <td><?= $no ?></td>
                <td><?= $data['nama_jabatan'] ?></td>
                <td>
                    <a href="?hal=jabatan_edit&id=<?= $data['id_jabatan'] ?>" class="tombol edit"> Edit</a>
                    <a href="?hal=jabatan_hapus&id=<?= $data['id_jabatan'] ?>" class="tombol hapus"> Hapus</a>
                </td>
            </tr>
        <?php    
        }
        ?>
    </tbody>
</table>
```

**jabatan_tambah.php**

Silahkan modifikasi dari file `form.html`

```php
<?php
if(!defined('INDEX')) die("");
?>

<h2 class="judul">Tambah Jabatan</h2>
<form action="?hal=jabatan_insert" method="post">
    
    <div class="form-group">
        <label for="nama">Nama</label>
        <div class="input">
            <input type="text" name="nama" id="nama">
        </div>
    </div>

    <div class="form-group">
        <input type="reset" value="Reset" class="tombol reset">
        <input type="submit" value="Simpan" class="tombol simpan">
    </div>
</form>
```

Skrip di atas berupa tampilan form, untuk bisa memproses datanya kita butuh file `jabatan_insert.php`, berikut skripnya.

**jabatan_insert.php**

```php
<?php
if(!defined('INDEX')) die("");

$nama_jabatan = $_POST['nama'];
$query = "INSERT INTO jabatan SET nama_jabatan = '$nama_jabatan'";
$result = mysqli_query($con,$query);

if ($result) {
    echo "Jabatan <b>$nama_jabatan</b> berhasil disimpan!";
    echo "<meta http-equiv='refresh' content='2; url=?hal=jabatan'>";
} else {
    echo "Tidak dapat menyimpan data!<br>";
    echo mysqli_error();
}
?>
```

## Membuat halaman `pegawai`