# Praktikum 4-6

## Profil
|  |  |
| -------- | --- |
| **Nama** |Raditya Tansy Lizara |
| **Kelas** | TI.23.A.5 |
| **Mata Kuliah** | Pemrograman Web 2 |

## Langkah - Langkah Praktikum

## Praktikum 4: Membuat Sistem Login
### 1. Menyiapkan Basis Data
   
Buatlah tabel user pada basis data dengan menggunakan perintah SQL berikut:

```php
CREATE TABLE user (
  id INT(11) auto_increment,
  username VARCHAR(200) NOT NULL,
  useremail VARCHAR(200),
  userpassword VARCHAR(200),
  PRIMARY KEY(id)
);
```

![database](https://github.com/user-attachments/assets/885fbd67-be7f-4191-accc-33562233b33a)

### 2. Membuat Model User

Buatlah file model UserModel.php di dalam direktori app/Models:

```php
<?php
namespace App\Models;
use CodeIgniter\Model;

class UserModel extends Model
{
    protected $table = 'user';
    protected $primaryKey = 'id';
    protected $useAutoIncrement = true;
    protected $allowedFields = ['username', 'useremail', 'userpassword'];
}
```

### 3. Membuat Controller User

Buatlah file controller `User.php` dengan metode `index()` dan `login()` yang berfungsi untuk mengelola data pengguna serta proses login:

```php
<?php

namespace App\Controllers;

use App\Models\UserModel;

class User extends BaseController
{
    public function index()
    {
        $title = 'Daftar User';
        $model = new UserModel();
        $users = $model->findAll();
        return view('user/index', compact('users', 'title'));
    }
    public function login()
    {
        helper(['form']);
        $email = $this->request->getPost('email');
        $password = $this->request->getPost('password');
        if (!$email)
        {
        return view('user/login');
        }

        $session = session();
        $model = new UserModel();
        $login = $model->where('useremail', $email)->first();
        if ($login)
        {
            $pass = $login['userpassword'];
            if (password_verify($password, $pass))
            {
                $login_data = [
                'user_id' => $login['id'],
                'user_name' => $login['username'],
                'user_email' => $login['useremail'],
                'logged_in' => TRUE,
                ];

                $session->set($login_data);
                return redirect('admin/artikel');
            }
            else
            {
                $session->setFlashdata("flash_msg", "Password salah.");
                return redirect()->to('/user/login');
            }
        }
        else
        {
            $session->setFlashdata("flash_msg", "email tidak terdaftar.");
            return redirect()->to('/user/login');
        }
    }
}
```

### 4. Membuat View Login

Buatlah file view `login.php` yang berisi form untuk proses login:

```php
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<title>Login</title>
		<link rel="stylesheet" href="<?= base_url('/style.css'); ?>" />
	</head>

	<body>
		<div id="login-wrapper">
			<h1>Sign In</h1>
			<?php if (session()->getFlashdata('flash_msg')): ?>
			<div class="alert alert-danger">
				<?= session()->getFlashdata('flash_msg') ?>
			</div>
			<?php endif; ?>
			<form action="" method="post">
				<div class="mb-3">
					<label for="InputForEmail" class="form-label">Email address</label>
					<input
						type="email"
						name="email"
						class="form-control"
						id="InputForEmail"
						value="<?= set_value('email') ?>"
					/>
				</div>
				<div class="mb-3">
					<label for="InputForPassword" class="form-label">Password</label>

					<input
						type="password"
						name="password"
						class="form-control"
						id="InputForPassword"
					/>
				</div>
				<button type="submit" class="btn btn-primary">Login</button>
			</form>
		</div>
	</body>
</html>
```

### 5. Membuat Databse Seeder

Database seeder digunakan untuk menghasilkan data dummy. Dalam rangka pengujian modul login, diperlukan data pengguna dan kata sandi yang dimasukkan ke dalam basis data. Oleh karena itu, buatlah *database seeder* untuk tabel `user`. Buka Command Line Interface (CLI), lalu jalankan perintah berikut:

```php
php spark make:seeder UserSeeder
```

Selanjutnya, buka file `UserSeeder.php` yang terletak di direktori `app/Database/Seeds/UserSeeder.php`, kemudian isi file tersebut dengan kode berikut:

```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run()
    {
        $model = model('UserModel');
        $model->insert([
            'username' => 'admin',
            'useremail' => 'admin@email.com',
            'userpassword' => password_hash('admin123', PASSWORD_DEFAULT),
        ]);
    }
}
```

Langkah berikutnya, buka kembali Command Line Interface (CLI) dan jalankan perintah berikut:

![auth](https://github.com/user-attachments/assets/1dd2d03e-25ad-4b31-82df-4e6b7c960d5c)

### 6. Membuat Filter Auth

Buat file filter `Auth.php` untuk membatasi akses ke halaman admin hanya bagi pengguna yang sudah login.

```php
<?php namespace App\Filters;

use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use CodeIgniter\Filters\FilterInterface;

class Auth implements FilterInterface
{
    public function before(RequestInterface $request, $arguments = null)
    {
        // jika user belum login
        if (! session()->get('logged_in')) {
            // maka redirct ke halaman login
            return redirect()->to('/user/login');
        }
    }
    public function after(RequestInterface $request, ResponseInterface $response, $arguments = null)
    {
        // Do something here
    }
}
```
Setelah itu, akses file `app/Config/Filters.php` dan tambahkan kode berikut:

```php
'auth' => App\Filters\Auth::class
```

![Auth Filters](https://github.com/user-attachments/assets/b509e76f-45eb-4fcb-b34d-50d811a82109)

Kemudian, buka file `app/Config/Routes.php` dan lakukan penyesuaian pada kodenya.

![Routes](https://github.com/user-attachments/assets/78efd2de-9ca4-484e-b899-d934b5585b0f)

### 7. Percobaan Menu Akses Admin

Buka URL [http://localhost:8080/admin/artikel](http://localhost:8080/admin/artikel); saat alamat tersebut diakses, halaman login akan ditampilkan.

![auth (1)](https://github.com/user-attachments/assets/37f27345-35c1-4756-8d79-f2f62889a155)

### 8. Fungsi Logout

Tambahkan metode `logout` pada Controller `User` dengan kode sebagai berikut:

```php
public function logout()
    {
        session()->destroy();
        return redirect()->to('/user/login');
    }
```

## Praktikum 5: Pagination dan Pencarian

### 1. Membuat Pagination

Lakukan modifikasi pada controller `Artikel` untuk menambahkan fitur paginasi:









