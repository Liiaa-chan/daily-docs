# Laravel Code Quality & Automation Standards

Dokumen ini berisi panduan resmi untuk menjaga konsistensi kode, arsitektur model/migrasi, dan standarisasi pengujian otomatis di lingkungan **Laravel**. Panduan ini wajib diikuti untuk mencegah kegagalan berulang (_pipeline failures_) pada GitHub Actions.

---

## Perangkat Kendali Mutu (Quality Tools)

Kami menggunakan kombinasi tiga alat utama untuk memastikan kode Laravel bersih, bebas dari bug logika, dan sinkron dengan skema database:

1.  **Laravel Pint:** Linter resmi Laravel untuk memperbaiki format kerapian kode secara otomatis (_style & formatting_).
2.  **Larastan (PHPStan):** Linter berbasis _static analysis_ yang memahami keajaiban (_magic methods_) Laravel untuk mendeteksi eror logika tanpa menjalankan kode.
3.  **Laravel IDE Helper:** Alat otomatis untuk menyinkronkan properti kolom database ke dalam dokumentasi file Model.

---

## Alur Kerja Pengujian Lokal (Local CI Workflow)

**DILARANG** melakukan `git push` ke GitHub sebelum memastikan seluruh proyek lulus pengecekan di komputer lokal Anda. Untuk menghemat waktu dan mencegah kegagalan berulang di server CI, gunakan perintah terpadu berikut:

### 1. Jalankan Sinkronisasi Model & Skema Database

Setiap kali Anda membuat atau mengubah file _migration_, jalankan perintah berikut untuk memperbarui dokumentasi Model agar linter tidak mendeteksi eror palsu (_false positives_):

```bash
# Jalankan migrasi di database lokal
php artisan migrate:fresh --seed

# Tulis ulang anotasi dokumentasi pada file Model secara otomatis
php artisan ide-helper:models --write-mixin
```

### 2. Jalankan Pemeriksaan Massal (Sekali Ketik)

Kami telah menyediakan skrip kustom untuk memeriksa **seluruh file di dalam proyek sekaligus**. Cukup ketik perintah ini di terminal lokal Anda:

```bash
composer ci-local
```

Perintah di atas secara otomatis mengeksekusi rangkaian pengecekan berikut:

1.  Memastikan seluruh berkas _migration_ dapat berjalan bersih dari nol tanpa konflik.
2.  Memeriksa kepatuhan gaya kode (_code style_) di seluruh berkas `.php` via Laravel Pint.
3.  Memindai seluruh eror logika dan tipe data di semua _Controller_, _Model_, dan _Service_ via Larastan.
4.  Menjalankan seluruh Unit dan Feature Test menggunakan PHPUnit/Pest.

_Catatan: Jika perintah `composer ci-local` menghasilkan status hijau (sukses), Anda dijamin 100% lulus CI di GitHub dalam sekali push._

---

## Konfigurasi Proyek (`composer.json`)

Pastikan bagian `"scripts"` di dalam file `composer.json` proyek Anda telah dikonfigurasi sebagai berikut:

```json
"scripts": {
    "post-autoload-dump": [
        "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
        "@php artisan package:discover --ansi"
    ],
    "post-update-cmd": [
        "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
    ],
    "post-root-package-install": [
        "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
    ],
    "post-create-project-cmd": [
        "@php artisan key:generate --ansi"
    ],
    "lint": "vendor/bin/pint --test",
    "lint:fix": "vendor/bin/pint",
    "analyse": "vendor/bin/phpstan analyse",
    "ci-local": [
        "@php artisan migrate:fresh --seed",
        "@lint",
        "@analyse",
        "@php artisan test"
    ]
}
```

---

## Resolusi Masalah Cepat (Troubleshooting)

### A. Linter Mendeteksi Ratusan Eror Kerapian Kode

Jangan panik dan jangan perbaiki manual satu per satu. Jalankan perintah _auto-fix_ global ini untuk membetulkan seluruh file dalam satu detik:

```bash
composer lint:fix
```

### B. Larastan Menganggap `$user->nama_kolom` Tidak Ada

Ini terjadi karena file Model Anda belum membaca perubahan terbaru di database.

- **Solusi:** Pastikan Anda sudah menjalankan `php artisan migrate`, lalu jalankan kembali `php artisan ide-helper:models --write-mixin`.
