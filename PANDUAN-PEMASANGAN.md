# Panduan Pemasangan - Gudang Ledger

Aplikasi ini adalah satu file HTML (`index.html`) yang berjalan sepenuhnya di browser. Tidak perlu server backend khusus - yang dibutuhkan hanya **Firebase** (gratis) untuk login dan penyimpanan data, lalu file-nya di-hosting di mana saja (atau bahkan dibuka langsung dari komputer).

Ikuti langkah-langkah di bawah secara berurutan.

---

## 1. Yang Dibutuhkan

- Akun Google (untuk membuat project Firebase)
- Browser (Chrome/Edge/Firefox)
- File `index.html` dari proyek ini

---

## 2. Membuat Project Firebase

1. Buka [https://console.firebase.google.com](https://console.firebase.google.com)
2. Klik **Add project** / **Tambah project**
3. Beri nama project (mis. `gudang-ledger`), lanjutkan sampai selesai (Google Analytics boleh dimatikan, tidak perlu)
4. Setelah project terbuat, klik ikon **Web (`</>`)** di halaman utama project untuk mendaftarkan aplikasi web
5. Beri nama app (mis. `gudang-ledger-web`), klik **Register app**
6. Firebase akan menampilkan blok kode `firebaseConfig` seperti ini:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "nama-project.firebaseapp.com",
     projectId: "nama-project",
     appId: "1:xxxx:web:xxxx"
   };
   ```

   **Simpan/screenshot bagian ini** - akan dipakai di langkah 4.

---

## 3. Mengaktifkan Authentication & Firestore

### a. Authentication (untuk login)
1. Di menu kiri Firebase Console, buka **Build > Authentication**
2. Klik **Get started**
3. Pilih metode **Email/Password**, klik toggle **Enable**, lalu **Save**

### b. Firestore Database (untuk penyimpanan data)
1. Di menu kiri, buka **Build > Firestore Database**
2. Klik **Create database**
3. Pilih lokasi server (pilih yang terdekat, mis. `asia-southeast2` untuk Indonesia)
4. Pilih mode **Start in production mode** (aturan keamanan akan diatur manual di langkah 5)
5. Klik **Enable**

---

## 4. Memasukkan Konfigurasi Firebase ke Aplikasi

1. Buka file `index.html` dengan text editor (Notepad, VS Code, dll)
2. Cari bagian ini (dekat awal tag `<script>`):

   ```js
   const FIREBASE_CONFIG_BAWAAN = {
     apiKey: "...",
     authDomain: "...",
     projectId: "...",
     appId: "..."
   };
   ```

3. Ganti nilai-nilainya dengan `firebaseConfig` dari project Firebase kamu sendiri (langkah 2.6)
4. Simpan file

> Setelah ini, **semua orang** yang membuka file/link aplikasi akan otomatis terhubung ke project Firebase kamu - tidak perlu setup lagi di perangkat masing-masing.

---

## 5. Memasang Aturan Keamanan Firestore (Wajib)

Tanpa langkah ini, data tidak akan bisa dibaca/ditulis sama sekali (atau sebaliknya, terbuka untuk siapa saja).

1. Di Firebase Console, buka **Firestore Database > Rules**
2. Hapus isi default, ganti dengan:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       function statusUser() {
         return get(/databases/$(database)/documents/users/$(request.auth.uid)).data;
       }
       function aktif() {
         return request.auth != null && statusUser().aktif == true;
       }
       function admin() {
         return request.auth != null && statusUser().role == 'admin';
       }

       match /barang/{id} {
         allow read, write: if aktif();
       }
       match /transaksi/{id} {
         allow read, write: if aktif();
       }
       match /kerusakan/{id} {
         allow read, write: if aktif();
       }
       match /users/{id} {
         allow read: if request.auth != null;
         allow create: if request.auth != null && request.auth.uid == id;
         allow update: if admin() || (request.auth.uid == id && request.resource.data.role == resource.data.role && request.resource.data.aktif == resource.data.aktif);
       }
     }
   }
   ```

3. Klik **Publish**

**Penjelasan singkat:** hanya user yang statusnya **aktif** (`aktif: true`) yang boleh baca/tulis data barang, transaksi, dan kerusakan. Setiap user boleh baca daftar user (untuk menu Kelola User), tapi hanya **admin** yang boleh mengubah role/status aktif orang lain.

---

## 6. (Opsional) Sinkronisasi ke Google Sheets

Kalau ingin setiap transaksi otomatis tersalin ke Google Sheets:

1. Buat Google Spreadsheet baru
2. Buka menu **Extensions > Apps Script**
3. Buat fungsi `doPost(e)` yang menerima data JSON dan menuliskannya ke sheet sesuai `sheet` (`masuk`, `keluar`, `master`) yang dikirim
4. **Deploy > New deployment > Web app**, akses **Anyone**, salin URL yang berakhiran `/exec`
5. Di `index.html`, isi variabel:

   ```js
   const GOOGLE_SHEETS_URL_BAWAAN = "https://script.google.com/macros/s/xxxx/exec";
   ```

Kalau tidak butuh fitur ini, biarkan kosong (`""`) saja - aplikasi tetap berjalan normal, Firestore tetap jadi sumber data utama.

---

## 7. Meng-hosting File

Pilih salah satu cara termudah:

- **Paling simpel:** buka langsung file `index.html` dua kali klik di komputer (berjalan di `file://`), atau upload ke Google Drive lalu buka dengan ekstensi HTML viewer
- **Netlify Drop:** buka [app.netlify.com/drop](https://app.netlify.com/drop), seret file `index.html` ke sana - langsung dapat link publik
- **Firebase Hosting:** karena sudah pakai Firebase, bisa juga host di sana lewat Firebase CLI (`firebase deploy`)
- **GitHub Pages:** upload ke repository GitHub, aktifkan Pages di Settings

Yang penting: **semua orang di tim membuka link/file yang sama**, supaya semuanya tersambung ke Firestore yang sama.

---

## 8. Pendaftaran Pertama Kali

1. Buka aplikasi, klik **Daftar**
2. Isi nama, email, kata sandi (khusus untuk pendaftar **pertama**) - akun ini otomatis jadi **Admin** dan langsung **aktif**
3. Setelah itu login dengan akun tersebut
4. Untuk anggota tim lainnya: mereka daftar sendiri lewat tombol **Daftar**, tapi akunnya akan berstatus **Belum Aktif** sampai Admin mengaktifkannya di menu **Kelola User**

---

## 9. Troubleshooting Umum

| Pesan Error | Penyebab | Solusi |
|---|---|---|
| API Key tidak valid | `apiKey` salah ketik | Cek ulang `FIREBASE_CONFIG_BAWAAN` |
| Project ID tidak ditemukan | `projectId` salah | Cek ejaan di Firebase Console |
| Gagal terhubung ke Firestore / database belum dibuat | Firestore belum diaktifkan | Ulangi langkah 3b |
| Email atau kata sandi salah | Salah input, atau akun belum daftar | Klik Daftar dulu kalau belum pernah |
| Setelah daftar/login tidak bisa lihat data apa pun | Akun belum diaktifkan admin | Minta Admin buka menu Kelola User > Aktifkan |

---

Kalau ada langkah yang bikin bingung pas dipraktikkan, kirim saja screenshot error-nya - nanti dibantu diagnosa lebih lanjut.
