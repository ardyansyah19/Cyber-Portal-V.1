# Sistem Login (PHP + MySQL)

Sistem login demo dengan berbagai lapisan keamanan (defense-in-depth).
Ditujukan untuk pembelajaran/pengembangan lokal — sesuaikan lagi sebelum dipakai produksi.

## Fitur Keamanan

| Ancaman              | Mitigasi yang diterapkan |
|----------------------|---------------------------|
| SQL Injection        | PDO **prepared statements** di semua query, `PDO::ATTR_EMULATE_PREPARES => false` |
| Password bocor       | `password_hash()` (Argon2id/bcrypt) + **pepper** tambahan via `hash_hmac` sebelum hashing |
| Brute force          | Rate limiting per-IP + penguncian akun otomatis setelah `MAX_LOGIN_ATTEMPTS` gagal |
| Timing attack        | `hash_equals()` untuk cek CSRF token, `password_verify()` dummy saat user tidak ditemukan |
| User enumeration     | Pesan error login digeneralisasi ("username atau password salah") |
| CSRF                 | Token acak per-session, wajib dicocokkan di setiap form POST |
| Session hijacking    | Cookie `HttpOnly`, `Secure` (saat HTTPS), `SameSite=Strict`, regenerasi session ID berkala |
| Session fixation     | `session_regenerate_id(true)` setiap kali login berhasil |
| XSS                  | `htmlspecialchars()` di semua output, Content-Security-Policy header |
| Clickjacking         | Header `X-Frame-Options: DENY` |
| Idle session         | Auto logout setelah 30 menit tidak aktif |
| Audit trail          | Tabel `activity_log` mencatat login/logout/registrasi |

## Cara Menjalankan

1. **Buat database & tabel**
   ```bash
   mysql -u root -p < database.sql
   ```

2. **Set environment variable** (jangan hardcode kredensial):
   ```bash
   export DB_HOST=127.0.0.1
   export DB_NAME=secure_login_db
   export DB_USER=db_user
   export DB_PASS="password_kuat_anda"
   export APP_PEPPER="string_rahasia_panjang_dan_acak_ganti_ini"
   ```

3. **Isi data dummy dengan hash yang valid**:
   ```bash
   php seed.php
   ```
   Ini akan menampilkan 3 akun dummy beserta passwordnya:
   - `admin_demo` / `P@ssw0rdKuat!123`
   - `budi_santoso` / `Budi#Aman2026`
   - `siti_rahayu` / `Siti$Secure99`

4. **Jalankan server PHP** (untuk testing lokal):
   ```bash
   php -S localhost:8000
   ```
   Buka `http://localhost:8000/login.php`

## Struktur File

```
secure-login/
├── database.sql      # skema tabel: users, login_attempts, activity_log
├── seed.php          # generate password hash valid untuk dummy user
├── config.php        # koneksi DB + hardening session + security headers
├── functions.php     # CSRF, rate limiting, hashing, validasi, audit log
├── login.php         # form + proses login
├── register.php      # form + proses registrasi
├── dashboard.php     # halaman terproteksi (butuh login)
├── logout.php        # hancurkan session dengan aman
└── .htaccess         # blokir akses langsung ke file sensitif
```

## Catatan Sebelum Produksi

- Ganti `APP_PEPPER` dengan string acak panjang (≥32 karakter) dan simpan di secrets manager, bukan di repo.
- Aktifkan HTTPS dan uncomment baris pemaksaan HTTPS di `.htaccess`.
- Pertimbangkan menambahkan **2FA (TOTP)** — kolom `two_factor_secret` sudah disediakan di skema.
- Pertimbangkan CAPTCHA (misal Cloudflare Turnstile) setelah beberapa kali gagal login.
- Batasi hak akses user MySQL (`db_user`) hanya ke database ini, jangan pakai akun root.
- Lakukan `password_needs_rehash()` check saat login untuk migrasi otomatis jika parameter hashing di-upgrade nanti.
