# HAGA_STORE — Docker configuration

Ringkasan: konfigurasi di repositori ini menyediakan stack Docker untuk menjalankan aplikasi web berbasis PHP dengan Nginx, MySQL, dan phpMyAdmin. File utama: `docker-compose.yml`, `Dockerfile`, dan konfigurasi Nginx di `nginx/default.conf`.

Services yang dibuat
- **nginx**: reverse/proxy dan web server yang melayani file dari `src/`.
- **php-fpm (php)**: menjalankan PHP (meng-handle request PHP pada Nginx melalui socket atau TCP).
- **mysql**: database layanan untuk menyimpan data aplikasi.
- **phpmyadmin**: GUI web untuk mengelola database MySQL.

Port (default yang biasa dipakai — periksa `docker-compose.yml` Anda)
- `80` -> Nginx (website)
- `3306` -> MySQL (internal, biasanya tidak diexpose publik kecuali diperlukan)
- `8080` atau `8081` -> phpMyAdmin (jika dikonfigurasi pada docker-compose)

Volume dan file penting
- `src/` di-mount ke container web; kode aplikasi PHP berada di `src/`.
- Direktori `src/HAGA_STORE/` berisi konfigurasi lokal, uploads, dan database SQL: folder ini *diabaikan* dari git (`.gitignore`) karena berisi data lokal/privat.
- Data MySQL disimpan di volume Docker (lihat `docker-compose.yml`).

Environment / rahasia
- Gunakan variabel lingkungan di `.env` atau `docker-compose.yml` untuk `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`. Jangan commit credential sensitif.

Cara menjalankan (singkat)
1. Pastikan Docker dan Docker Compose terpasang.
2. (Opsional) buat file `.env` dengan credential MySQL.
3. Jalankan:

```bash
docker compose up -d
```

4. Lihat logs:

```bash
docker compose logs -f nginx
```

5. Matikan/bersihkan:

```bash
docker compose down
```

Catatan penting
- `src/HAGA_STORE/` di-ignore oleh git; jika Anda sudah pernah men-commit folder tersebut, jalankan di lokal:

```bash
git rm -r --cached src/HAGA_STORE
git add .gitignore
git commit -m "Ignore src/HAGA_STORE (local uploads/config)"
```

- Periksa `nginx/default.conf` untuk menyesuaikan server_name, root, dan aturan rewrite.
- Jika ingin mempublikasikan repo ini ke `https://github.com/Rakhasptro/docker-config.git`, ubah remote lokal sebelum push:

```bash
git remote set-url origin https://github.com/Rakhasptro/docker-config.git
git push -u origin main
```

Kontak / next steps
- Jika Anda mau, saya bisa:
  - commit perubahan `.gitignore` untuk Anda (saya sudah menambahkan file `.gitignore`),
  - menjalankan perintah `git rm --cached` untuk meng-untrack `src/HAGA_STORE` (butuh konfirmasi), atau
  - menyesuaikan README dengan detail port/variabel yang ada di `docker-compose.yml` Anda.
