# cara-membuat-virtualhost-di-fedora-untuk-laravel-12

note: catatan di bawah ditulis menggunakan chatgpt agar lebih rapi/terstruktur.

---

# 🚀 Setup VirtualHost Apache di Fedora untuk Laravel 12

Tutorial ini menjelaskan langkah-langkah **konfigurasi VirtualHost Apache di Fedora** untuk menjalankan **Laravel 12** menggunakan **domain lokal** secara aman dan terstruktur.

---

## 📌 Prasyarat

* Fedora Linux
* Akses `sudo`
* Apache (httpd)
* PHP 8.2 atau lebih baru
* Composer
* Project Laravel

---

## 1️⃣ Install Apache (httpd)

```bash
sudo dnf install httpd -y
```

Aktifkan dan jalankan Apache:

```bash
sudo systemctl enable httpd
sudo systemctl start httpd
```

Verifikasi melalui browser:

```
http://localhost
```

---

## 2️⃣ Install PHP (Minimal PHP 8.2)

```bash
sudo dnf install php php-cli php-common php-mbstring php-xml php-curl php-mysqlnd php-zip -y
```

Cek versi PHP:

```bash
php -v
```

---

## 3️⃣ Install Composer

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer --version
```

---

## 4️⃣ Buat Project Laravel

```bash
mkdir -p ~/projects
cd ~/projects
composer create-project laravel/laravel laravel-app
```

Struktur penting:

```text
laravel-app/
└── public   ← DocumentRoot
```

---

## 5️⃣ Atur Permission Folder

Apache memerlukan akses ke direktori project:

```bash
sudo chown -R apache:apache /home/USER/projects/laravel-app
sudo chmod -R 755 /home/USER/projects
sudo chmod -R 775 /home/USER/projects/laravel-app/storage
sudo chmod -R 775 /home/USER/projects/laravel-app/bootstrap/cache
```

> Ganti `USER` dengan username Linux masing-masing.

---

## 6️⃣ Konfigurasi SELinux (WAJIB di Fedora)

Install utilitas SELinux:

```bash
sudo dnf install policycoreutils-python-utils -y
```

Atur context SELinux:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/home/USER/projects/laravel-app(/.*)?"
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/home/USER/projects/laravel-app/storage(/.*)?"
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/home/USER/projects/laravel-app/bootstrap/cache(/.*)?"

sudo restorecon -Rv /home/USER/projects/laravel-app
```

---

## 7️⃣ Pastikan `mod_rewrite` Aktif

```bash
sudo httpd -M | grep rewrite
```

Jika belum aktif, edit:

```bash
sudo nano /etc/httpd/conf.modules.d/00-base.conf
```

Pastikan baris berikut tidak dikomentari:

```apache
LoadModule rewrite_module modules/mod_rewrite.so
```

---

## 8️⃣ Buat VirtualHost Apache

Buat file konfigurasi:

```bash
sudo nano /etc/httpd/conf.d/laravel-app.conf
```

Isi konfigurasi berikut:

```apache
<VirtualHost *:80>
    ServerName laravel.test
    ServerAlias www.laravel.test
    DocumentRoot /home/USER/projects/laravel-app/public

    <Directory /home/USER/projects/laravel-app/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog logs/laravel-app_error.log
    CustomLog logs/laravel-app_access.log combined
</VirtualHost>
```

---

## 9️⃣ Tambahkan Domain ke `/etc/hosts`

```bash
sudo nano /etc/hosts
```

Tambahkan baris berikut:

```text
127.0.0.1   laravel.test www.laravel.test
```

---

## 🔟 (Opsional) Hilangkan Warning Apache

Edit konfigurasi global Apache:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Tambahkan di bagian bawah:

```apache
ServerName localhost
```

Reload Apache:

```bash
sudo systemctl reload httpd
```

---

## 1️⃣1️⃣ Validasi & Reload Apache

```bash
sudo apachectl configtest
sudo systemctl reload httpd
```

Pastikan output:

```text
Syntax OK
```

---

## 1️⃣2️⃣ Generate Application Key Laravel

```bash
cd ~/projects/laravel-app
php artisan key:generate
```

---

## 1️⃣3️⃣ Akses Aplikasi

Buka browser dan akses:

```
http://laravel.test
```

Jika halaman Laravel tampil, maka konfigurasi berhasil.

---

## 🧪 Troubleshooting

### Cek VirtualHost

```bash
sudo apachectl -S
```

### Cek Log Apache

```bash
sudo tail -f /var/log/httpd/error_log
sudo tail -f /var/log/httpd/laravel-app_error.log
```

---

## ✅ Kesimpulan

* VirtualHost Apache berhasil dikonfigurasi di Fedora
* Laravel berjalan menggunakan domain lokal
* Tutorial ini aman untuk dokumentasi publik

