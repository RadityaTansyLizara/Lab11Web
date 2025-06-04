# Praktikum ke 4 - 6

## Profil

|                 |                        |
| --------------- | ---------------------- |
| **Nama**        |Raditya Tansy Lizara |
| **Kelas**       | TI.23.A.5              |
| **Mata Kuliah** | Pemrograman Web 2      |

## Langkah-langkah Praktikum

### **Praktikum 4: Membuat Sistem Login**

#### 1. Lakukan Persiapan Basis Data

Gunakan SQL berikut untuk mendefinisikan tabel `user`.

```sql
CREATE TABLE user (
  id INT(11) auto_increment,
  username VARCHAR(200) NOT NULL,
  useremail VARCHAR(200),
  userpassword VARCHAR(200),
  PRIMARY KEY(id)
);
```

![database (1)](https://github.com/user-attachments/assets/376e1a78-e80e-4c6d-892b-491859c8cf02)

#### 2. Membuat Model User

Letakkan file model `UserModel.php` ke dalam direktori `app/Models`.

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

#### 3. Membuat Controller User

Rancang file `User.php` sebagai controller dengan fungsi `index()` dan `login()` untuk kebutuhan manajemen user dan login.

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

#### 4. Membuat View Login

Rancang view `login.php` sebagai antarmuka form login.

```html
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

#### 5. Membuat Database Seeder

Database seeder berfungsi untuk menghasilkan data dummy yang diperlukan dalam pengujian.
Dalam rangka menguji modul login, diperlukan penyisipan data pengguna beserta kata sandinya ke dalam basis data. Oleh karena itu, buatlah sebuah database seeder untuk tabel `user`.

Langkah selanjutnya, buka Command Line Interface (CLI) dan jalankan perintah berikut:

```php
php spark make:seeder UserSeeder
```

Langkah berikutnya, buka file `UserSeeder.php` yang terletak di direktori `app/Database/Seeds/`.
Kemudian, lengkapi file tersebut dengan kode berikut:

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

Setelah itu, buka kembali Command Line Interface (CLI) dan jalankan perintah berikut:

```php
php spark db:seed UserSeeder
```

#### Uji Coba Login

![auth (1)](https://github.com/user-attachments/assets/b8ad7dbe-b4bf-4d18-9bda-78127b01cbeb)


#### 6. Membuat Filter Auth

Siapkan filter `Auth.php` sebagai pengaman untuk membatasi akses ke halaman admin.

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

Lanjutkan dengan membuka file `app/Config/Filters.php` dan tambahkan kode berikut ini:

```php
'auth' => App\Filters\Auth::class
```

![Auth Filters (1)](https://github.com/user-attachments/assets/4ac31538-8c74-4a55-8cea-2a29cdaa03d6)

Setelah itu, buka file `app/Config/Routes.php` dan lakukan penyesuaian kode.
![Routes (1)](https://github.com/user-attachments/assets/d8133e09-3882-419b-8abc-dccd66b5c81c)


#### 7. Percobaan Akses Menu Admin

Buka URL dengan alamat http://localhost:8080/admin/artikel.
Saat alamat tersebut diakses, halaman login akan otomatis ditampilkan.

![alt text](img/auth.png)

#### 8. Fungsi Logout

Sisipkan fungsi `logout` ke dalam Controller User sesuai kode berikut:

```php
public function logout()
    {
        session()->destroy();
        return redirect()->to('/user/login');
    }
```

---

### **Praktikum 5: Pagination dan Pencarian**

#### 1. Membuat Pagination

Sesuaikan controller `Artikel` agar mendukung pagination.

```php
public function admin_index()
{
    $title = 'Daftar Artikel';
    $model = new ArtikelModel();
    $data = [
        'title' => $title,
        'artikel' => $model->paginate(10), #data dibatasi 10 record per halaman
        'pager' => $model->pager,
    ];
    return view('artikel/admin_index', $data);
}
```
Lanjutkan dengan membuka file `views/artikel/admin_index.php` lalu masukkan kode berikut di bawah bagian deklarasi tabel data.
```php
<?= $pager->links(); ?>
```
Setelah itu, buka kembali menu daftar artikel dan masukkan data baru untuk memastikan hasilnya.

![alt text](img/pagination.png)

#### 2. Membuat Pencarian

Perbarui controller dengan menambahkan fungsi pencarian data.

```php
public function admin_index()
    {
        $title = 'Daftar Artikel';
        $q = $this->request->getVar('q') ?? '';
        $model = new ArtikelModel();
        $data = [
            'title' => $title,
            'q' => $q,
            'artikel' => $model->like('judul', $q)->paginate(10), # data dibatasi 10 record per halaman
            'pager' => $model->pager,
        ];
        return view('artikel/admin_index', $data);
    }
```
Kemudian, akses file `views/artikel/admin_index.php` dan sisipkan form pencarian di atas deklarasi tabel dengan kode berikut:

```html
<form method="get" class="form-search">
    <input type="text" name="q" value="<?= $q; ?>" placeholder="Cari data">
    <input type="submit" value="Cari" class="btn btn-primary">
</form>
```
Perbarui link pager agar mengikuti format berikut.

```php
<?= $pager->only(['q'])->links(); ?>
```

#### 3. Uji Coba Pagination dan Pencarian

Lanjutkan dengan membuka halaman admin artikel, lalu ketikkan kata kunci pada form pencarian untuk menguji fungsinya.
![alt text](img/search.png)

---

### **Praktikum 6: Upload File Gambar**

#### 1. Modifikasi Controller Artikel

Selanjutnya, buka Controller `Artikel` dari proyek sebelumnya dan sesuaikan implementasi method `add` seperti yang dijelaskan berikut:

```php
public function add()
    {
        // validasi data.
        $validation = \Config\Services::validation();
        $validation->setRules(['judul' => 'required']);
        $isDataValid = $validation->withRequest($this->request)->run();
        if ($isDataValid) {
            $file = $this->request->getFile('gambar');
            $file->move(ROOTPATH . 'public/gambar');
            $artikel = new ArtikelModel();
            $artikel->insert([
                'judul' => $this->request->getPost('judul'),
                'isi' => $this->request->getPost('isi'),
                'slug' => url_title($this->request->getPost('judul')),
                'gambar' => $file->getName(),
            ]);
            return redirect('admin/artikel');
        }
        $title = "Tambah Artikel";
        return view('artikel/form_add', compact('title'));
    }
```

#### 2. Modifikasi View Artikel

Lengkapi form artikel dengan penambahan field input untuk file.

```html
<p>
    <input type="file" name="gambar">
</p>
```
Ubah tag form dengan menyisipkan atribut enctype sesuai contoh berikut.

```html
<form action="" method="post" enctype="multipart/form-data">
```

#### 3. Uji Coba Upload Gambar

Masuk ke menu tambah artikel dan uji fungsi unggah gambar.

![alt text](<img/add file.png>)
---

