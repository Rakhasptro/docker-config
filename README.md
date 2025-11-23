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
# docker-config — Dockerized PHP + Nginx stack (HAGA_STORE)

This repository contains a Docker configuration that runs a simple PHP web application using the following stack:

- Nginx (web server / reverse proxy)
- PHP-FPM (PHP 8.2)
- MySQL 5.7 (database)
- phpMyAdmin (database administration web UI)

Main files:
- `docker-compose.yml` — declares the services, ports, volumes, and environment variables
- `Dockerfile` — builds the `php` service image with required PHP extensions
- `nginx/default.conf` — Nginx server configuration
- `src/` — application code mounted into containers

---

Services (as defined in `docker-compose.yml`)

- `php` (built from `Dockerfile`)
	- Purpose: runs PHP-FPM (php:8.2-fpm) and handles PHP execution
	- Build: uses the repository `Dockerfile` (installs `mysqli`, `pdo`, and `pdo_mysql` extensions)
	- Mounts: `./src` → `/var/www/html`
	- Depends on: `db`

- `nginx` (official `nginx` image)
	- Purpose: serves HTTP traffic and forwards PHP requests to `php` via FastCGI
	- Ports: `8080:80` (host:container)
	- Mounts:
		- `./src` → `/var/www/html` (application code)
		- `./nginx/default.conf` → `/etc/nginx/conf.d/default.conf` (site configuration)
	- Depends on: `php`

- `db` (MySQL 5.7)
	- Purpose: relational database for the application
	- Ports: `3306:3306` (exposes MySQL on host; change if you don't want it exposed)
	- Environment (defaults present in `docker-compose.yml`):
		- `MYSQL_ROOT_PASSWORD: root`
		- `MYSQL_DATABASE: test_db`

- `phpmyadmin` (phpMyAdmin)
	- Purpose: web UI to manage the `db` service
	- Ports: `8081:80` (open phpMyAdmin at `http://localhost:8081`)
	- Environment:
		- `PMA_HOST: db` (points phpMyAdmin to the `db` service)

---

Ports summary (default mapping in this repo)

- Website (Nginx): http://localhost:8080 → container port 80
- phpMyAdmin: http://localhost:8081 → container port 80
- MySQL: host port 3306 → container port 3306 (change if you prefer not to expose it)

Volumes and persistence

- Application code: `./src` is mounted into `php` and `nginx` at `/var/www/html` so code changes are reflected immediately.
- MySQL data: unless explicitly configured with a host-mounted volume in `docker-compose.yml`, MySQL data will be stored in a Docker-managed volume. Check `docker-compose.yml` to ensure persistent volumes if required.
- Note: the `src/HAGA_STORE/` directory was previously used for uploads/local config and is intentionally ignored from git (see `.gitignore`).

Dockerfile details

- Base image: `php:8.2-fpm`
- Installed extensions: `mysqli`, `pdo`, `pdo_mysql`

Nginx config

- `nginx/default.conf` sets `root /var/www/html;` and forwards PHP requests to `php:9000` (fastcgi_pass). This matches the `php` (php-fpm) service.

Environment and credentials

- The compose file currently contains default values:
	- `MYSQL_ROOT_PASSWORD: root`
	- `MYSQL_DATABASE: test_db`

	These defaults are fine for local development but should be changed before production. Use a `.env` file or override the environment variables in `docker-compose.yml`.

---

How to run (development)

1. Make sure Docker Desktop / Docker Engine and Docker Compose are installed.
2. (Optional) Create a `.env` file in the project root to override credentials, e.g.:

```env
MYSQL_ROOT_PASSWORD=supersecret
MYSQL_DATABASE=haga_store
MYSQL_USER=haga
MYSQL_PASSWORD=yourpassword
```

3. Start the stack:

```bash
docker compose up -d
```

4. Check logs for a service (example: nginx):

```bash
docker compose logs -f nginx
```

5. Access the app and DB UI:

- Application: http://localhost:8080
- phpMyAdmin: http://localhost:8081 (use MySQL root or the created user)

6. Stop and remove containers:

```bash
docker compose down
```

If you change environment variables that affect the database, you may need to recreate the `db` volume or the container:

```bash
docker compose down -v
docker compose up -d
```

Common tasks

- Run a shell in the `php` container:

```bash
docker compose exec php bash
```

- Run migrations / import SQL into `db`:

```bash
docker compose exec db mysql -u root -p${MYSQL_ROOT_PASSWORD} ${MYSQL_DATABASE} < /path/to/dump.sql
```

Troubleshooting

- Port conflicts: if `8080`, `8081`, or `3306` are already taken, update `docker-compose.yml` port mappings.
- Permission errors for file uploads: ensure host `src/` permissions allow container to write to `/var/www/html/uploads` (adjust ownership or mount options).
- php-fpm connection errors: confirm `nginx/default.conf` uses `fastcgi_pass php:9000;` and the `php` service is healthy.

Security notes

- Do not commit production credentials to git. Use `.env` and add it to `.gitignore`.
- Do not expose MySQL on public networks; remove the `3306:3306` mapping when deploying to servers where the DB should remain internal.

Contributing / next steps

- If you want, I can:
	- update README with exact environment variable names from your `.env` (if you create one),
	- add a persistent named volume for MySQL in `docker-compose.yml`, or
	- help set up a simple entrypoint to initialize the database from `src/HAGA_STORE/thread_db.sql`.

---

If you'd like me to commit this README and push to `https://github.com/Rakhasptro/docker-config.git`, confirm and I will run the git commands.
