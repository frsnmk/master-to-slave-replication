
# ✅ Checklist Migrasi Aplikasi Sementara ke VPS (MySQL/MariaDB Master-Slave)

---

## 🔧 1. Persiapan di Server Lama (Master)

- [ ] Buat user replikasi:
  ```sql
  CREATE USER 'repl'@'%' IDENTIFIED BY 'passwordku';
  GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
  FLUSH PRIVILEGES;
  ```

- [ ] Edit konfigurasi MySQL/MariaDB (`/etc/my.cnf`):
  ```ini
  [mysqld]
  server-id=1
  log_bin=mysql-bin
  binlog_do_db=namadb
  ```

- [ ] Restart MariaDB:
  ```bash
  sudo systemctl restart mariadb
  ```

- [ ] Lock tabel dan catat posisi binlog:
  ```sql
  FLUSH TABLES WITH READ LOCK;
  SHOW MASTER STATUS;
  ```
- [ ] Contoh output daru master status yang harus disimpan
  ```sql
  MASTER_LOG_FILE='mysql-bin.000237',
  MASTER_LOG_POS=3119983;
  ```
- [ ] Jangan tutup koneksi terminal yang terkunci!

- [ ] Dump database (terminal baru):
  ```bash
  mysqldump -u root -p --databases hsklinik-prod > dump.sql
  ```

- [ ] Unlock tabel:
  ```sql
  UNLOCK TABLES;
  ```

- [ ] Transfer dump ke VPS:
  ```bash
  scp dump.sql user@ip_vps:/home/user/
  ```

---

## 🧱 2. Setup di VPS (Slave)

- [ ] Edit konfigurasi `/etc/my.cnf`:
  ```ini
  [mysqld]
  server-id=2
  ```

- [ ] Restart MariaDB:
  ```bash
  sudo systemctl restart mariadb
  ```

- [ ] Import dump:
  ```bash
  mysql -u root -p < dump.sql
  ```

- [ ] Jalankan `CHANGE MASTER TO` (isi dari posisi binlog tadi):
  ```sql
  CHANGE MASTER TO
    MASTER_HOST='host',
    MASTER_USER='repl',
    MASTER_PASSWORD='passwordku',
    MASTER_LOG_FILE='mysql-bin.000237',
    MASTER_LOG_POS=3119983;
  ```

- [ ] Mulai replikasi:
  ```sql
  START SLAVE;
  ```

- [ ] Cek status:
  ```sql
  SHOW SLAVE STATUS\G
  ```

---
- [ ] Pastikan semua berjalan normal
