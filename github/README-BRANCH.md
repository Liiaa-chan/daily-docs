# Git Branching & Workflow Standards

Dokumen ini berisi panduan resmi dan standarisasi manajemen _branch_ Git yang digunakan dalam proyek ini. Seluruh kontributor dan pengembang **wajib** mengikuti aturan ini demi menjaga kerapian repositori, stabilitas kode produksi, dan kelancaran proses CI/CD.

---

## Standar Penamaan Branch (Naming Conventions)

Semua nama _branch_ baru harus ditulis menggunakan huruf kecil (_lowercase_) dan mengikuti format terstruktur berikut:

```text
kategori/nomor-tiket-atau-deskripsi-singkat
```

### Kategori Branch yang Diizinkan:

- **`feature/`**
    - **Kegunaan:** Pengembangan fitur atau fungsionalitas baru.
    - _Contoh:_ `feature/login-page` atau `feature/jira-102-oauth`
- **`bugfix/`**
    - **Kegunaan:** Perbaikan masalah atau _bug_ yang ditemukan pada lingkungan pengembangan/staging (belum rilis ke publik).
    - _Contoh:_ `bugfix/navbar-overlap` atau `bugfix/issue-45-broken-link`
- **`hotfix/`**
    - **Kegunaan:** Perbaikan darurat (_critical bug_) langsung di lingkungan produksi (_production/live_) yang harus segera diselesaikan.
    - _Contoh:_ `hotfix/payment-gateway-crash`
- **`docs/`**
    - **Kegunaan:** Pembaruan atau pembuatan dokumentasi (file README, API docs, atau komentar kode).
    - _Contoh:_ `docs/update-installation-guide`
- **`chore/`** atau **`refactor/`**
    - **Kegunaan:** Pemeliharaan repositori, pembaruan _library_ (_dependencies_), konfigurasi build, atau restrukturisasi kode tanpa mengubah fungsi aplikasi.
    - _Contoh:_ `chore/update-dependencies` atau `refactor/clean-database-query`

---

## Alur Kerja Git (Git Workflow)

Proyek ini mengadopsi standarisasi **GitHub Flow** yang disesuaikan untuk kebutuhan industri:

### 1. Cabang Utama (Protected Branches)

- **`main` (atau `master`)**
    - Merupakan sumber kebenaran utama (_Source of Truth_) dari aplikasi.
    - Kode di dalam _branch_ ini harus selalu stabil, bersih, dan **siap dirilis kapan saja ke server produksi**.
    - **DILARANG** melakukan `push` langsung ke _branch_ ini. Semua perubahan wajib melalui proses _Pull Request_ (PR).

### 2. Siklus Kerja Developer (Step-by-Step)

Setiap kali Anda ingin membuat fitur baru atau memperbaiki bug, ikuti langkah-langkah profesional berikut:

```bash
# 1. Pastikan branch utama lokal Anda adalah yang paling update
git checkout main
git pull origin main

# 2. Buat branch baru sesuai standar penamaan (contoh untuk fitur baru)
git checkout -b feature/nama-fitur-anda

# 3. Lakukan coding dan simpan perubahan secara berkala (Commit)
git add .
git commit -m "feat(scope): deskripsi singkat perubahan Anda"

# 4. Push branch Anda ke GitHub remote
git push origin feature/nama-fitur-anda
```

---

## Gerbang Kualitas & Aturan Pull Request (PR)

Sebelum sebuah _branch_ fitur dapat digabungkan (_merged_) ke dalam `main`, kode Anda harus melewati gerbang otomatis dan manual berikut:

1.  **Continuous Integration (CI) Check**
    Sistem CI di GitHub Actions akan otomatis memeriksa:
    - **Linting:** Kerapian dan standarisasi penulisan kode.
    - **Build Validation:** Memastikan kode sukses di-_compile_ tanpa eror.
    - **Automated Testing:** Semua Unit Test wajib lulus 100% dengan batas minimum _Code Coverage_ sebesar **80%**.
2.  **Code Review & Approval**
    - Setiap PR wajib ditinjau dan disetujui (_approved_) oleh minimal **1-2 pengembang lain (Peer Reviewers)** atau _Code Owner_.
3.  **Kebersihan Repositori (Post-Merge)**
    - Segera setelah Pull Request Anda sukses di-_merge_, **branch fitur wajib dihapus** dari GitHub remote dan mesin lokal Anda untuk menghindari penumpukan _branch_ usang.

```bash
# Cara menghapus branch lokal yang sudah selesai di-merge
git checkout main
git pull origin main
git branch -d feature/nama-fitur-anda
```
