# Continuous Integration (CI) Pipeline Standards

Dokumen ini menjelaskan arsitektur, standarisasi, dan alur kerja sistem **Continuous Integration (CI)** yang diterapkan dalam repositori ini menggunakan **GitHub Actions**. Setiap kontributor wajib memahami mekanisme pengujian otomatis ini untuk memastikan kualitas kode tetap terjaga.

---

## Arsitektur Pipeline CI (Multi-Stage)

Sistem CI kita dirancang menggunakan konsep **Fast-Fail Pipeline**. Pemeriksaan dibagi menjadi 4 tahap (_stages_) berurutan. Jika salah satu tahap gagal, proses akan langsung dihentikan secara otomatis untuk menghemat waktu dan sumber daya _cloud_.

```text
 [ Developer Pull Request ]
           │
           ▼
┌─────────────────────────┐
│     Stage 1: LINT       │ ❌ Gagal -> Stop (Format kode berantakan)
└──────────┬──────────────┘
           │  Lolos
           ▼
┌─────────────────────────┐
│    Stage 2: SECURITY    │ ❌ Gagal -> Stop (Ada celah keamanan/bocor API key)
└──────────┬──────────────┘
           │  Lolos
           ▼
┌─────────────────────────┐
│     Stage 3: BUILD      │ ❌ Gagal -> Stop (Gagal kompilasi / syntax error)
└──────────┬──────────────┘
           │  Lolos
           ▼
┌─────────────────────────┐
│     Stage 4: TEST       │ ❌ Gagal -> Stop (Unit test gagal / Coverage < 80%)
└──────────┬──────────────┘
           │  Lolos
           ▼
 [ 🔥 PR Approved & Ready to Merge ]
```

---

## Detail Pemeriksaan di Setiap Stage

### Stage 1: Code Style & Formatting (Linting)

- **Tujuan:** Memastikan seluruh tim menulis kode dengan gaya yang seragam dan rapi.
- **Apa yang dicek:** Spasi/tab, variabel yang tidak terpakai (_unused variables_), dan struktur penulisan sesuai _style guide_ industri.
- **Kriteria Lolos:** 0 Error dari Linter.

### Stage 2: Security Scanning (SAST)

- **Tujuan:** Mencegah celah keamanan masuk ke server produksi.
- **Apa yang dicek:**
    - Pemindaian rahasia (_Secret Scanning_) untuk mendeteksi jika ada token atau API Key yang tertulis langsung di kode (_hardcoded_).
    - Pemindaian pustaka pihak ketiga (_Dependency Scan_) untuk mencari _package_ usang yang memiliki celah keamanan aktif.

### Stage 3: Build Validation

- **Tujuan:** Memastikan aplikasi aman untuk dikemas.
- **Apa yang dicek:** Kode berhasil di-kompilasi (_compile_) tanpa eror sintaksis, dan pengecekan tipe data static (misal: TypeScript) tidak mendeteksi _mismatch_.

### Stage 4: Automated Testing & Coverage

- **Tujuan:** Memastikan logika bisnis berjalan 100% benar dan tidak merusak fitur lama (_regression_).
- **Aturan Pengujian yang Baik di CI:**
    1.  **Isolasi Total:** Pengujian unit tidak boleh menembak database asli atau API eksternal. Gunakan _Mocking_ / _Stubbing_.
    2.  **Deterministik:** Hasil tes harus selalu sama. Tes tidak boleh lulus secara acak (_flaky tests_).
    3.  **Minimum Code Coverage (80%):** Sistem CI akan menolak PR jika persentase baris kode yang dilindungi oleh pengujian berada di bawah **80%**.

---

## File Konfigurasi Pipeline (`.github/workflows/ci.yml`)

Berikut adalah berkas manifes otomatisasi GitHub Actions standar industri maju yang digunakan dalam proyek ini:

```yaml
name: Continuous Integration (CI)

on:
    pull_request:
        branches: [main, develop]

jobs:
    # STAGE 1 & 2: Lint & Security
    lint-and-security:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout Code
              uses: actions/checkout@v4

            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: 20
                  cache: "npm"

            - name: Install Dependencies
              run: npm ci

            - name: Run Linter (Code Style)
              run: npm run lint

            - name: Security Scan (Audit Dependencies)
              run: npm audit --audit-level=high

    # STAGE 3: Build Validation
    build-validation:
        needs: lint-and-security # Hanya jalan jika stage 1 & 2 lolos
        runs-on: ubuntu-latest
        steps:
            - name: Checkout Code
              uses: actions/checkout@v4

            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: 20
                  cache: "npm"

            - name: Install Dependencies
              run: npm ci

            - name: Compile Application
              run: npm run build

    # STAGE 4: Automated Testing
    automated-testing:
        needs: build-validation # Hanya jalan jika stage 3 lolos
        runs-on: ubuntu-latest
        steps:
            - name: Checkout Code
              uses: actions/checkout@v4

            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: 20
                  cache: "npm"

            - name: Install Dependencies
              run: npm ci

            - name: Execute Unit & Integration Tests
              run: npm run test:coverage # Menjalankan tes sekaligus menghasilkan laporan coverage

            - name: Check Coverage Threshold
              # Skrip internal untuk memastikan coverage minimal 80%
              run: npm run check-coverage 80
```

---

## Pro-Tips untuk Developer

- **Jalankan secara Lokal Terlebih Dahulu:** Sebelum melakukan `git push`, biasakan menjalankan perintah `npm run lint` dan `npm test` di terminal komputer Anda sendiri agar tidak mengotori riwayat _commit_ akibat eror sepele di CI.
- **Gunakan Git Pre-commit Hooks:** Anda bisa memasang _tools_ seperti `husky` untuk mencegah _commit_ dibuat secara lokal jika linter atau unit tes Anda terdeteksi gagal.
