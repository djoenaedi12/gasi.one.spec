# Kontrak Web/UI

## Route dan kepemilikan

Base route hasil generate: `/user-accounts`.

| Route | Permukaan | Kepemilikan |
|---|---|---|
| `/user-accounts` | Daftar/pencarian/filter berhalaman di server | Hasil generate ditambah slot ResourceCustom |
| `/user-accounts/new` | Form create | Hasil generate |
| `/user-accounts/:id` | Detail akun | Data/konten hasil generate yang disusun pengganti ResourceCustom |
| `/user-accounts/:id/edit` | Form edit profil | Hasil generate |

Modul khusus didaftarkan dari startup plugin melalui import bootstrap yang
dideklarasikan dalam JSON dan direncanakan. File TS/TSX hasil generate tidak
diubah.

## Kontrak daftar

Kolom yang terlihat secara default:

- username;
- nama tampilan;
- email;
- status;
- masa otorisasi.

Telepon tetap dapat dicari dan tersedia melalui visibilitas kolom, tetapi tidak
harus terlihat secara default. Avatar dikeluarkan dari DTO ringkasan/kolom daftar.

Global search:

- satu nilai teks dengan debounce;
- filter OR/LIKE pada username, nama tampilan, email, dan telepon;
- dikomposisikan dengan filter status exact khusus menggunakan AND;
- ukuran page server default 20;
- sort default berdasarkan username ascending.

Filter status:

- Semua, Aktif, Nonaktif;
- Aktif dipetakan ke `status EQUALS ACTIVE`;
- Nonaktif dipetakan ke `status EQUALS INACTIVE`;
- menghapus filter tidak mengubah global search.

Aksi per row:

- Lihat dan Edit menggunakan perilaku navigasi hasil generate.
- Hapus dihilangkan.
- Aktifkan muncul untuk row INACTIVE ketika aktor boleh memperbarui.
- Nonaktifkan muncul untuk row ACTIVE ketika aktor boleh memperbarui dan bukan akun saat ini.
- Aksi siklus hidup memerlukan detail/version terbaru; jika proyeksi ringkasan tidak memuat version, aksi memuat detail sebelum konfirmasi.

## Kontrak create

Bagian:

1. Identitas: username, nama tampilan, email, telepon.
2. Kelayakan akses: status wajib dan masa otorisasi opsional.
3. Tampilan: URI/referensi avatar opsional.

Perilaku:

- error username dan email bersifat spesifik pada field dan dapat ditindaklanjuti;
- status hanya memiliki pilihan Aktif/Nonaktif;
- metadata dibuat/diperbarui tidak pernah dapat diedit;
- tidak ada kontrol password, invitation, MFA, lockout, atau role;
- akun ACTIVE dengan authorized-until yang sudah kedaluwarsa ditolak API meskipun clock browser berbeda.

## Kontrak edit

Form edit hasil generate mendukung username, nama tampilan, email, telepon,
avatar, dan authorized-until. Status dikeluarkan dari `UpdateRequest`/form hasil
generate dan hanya dapat berubah melalui aksi siklus hidup. Field `version`
tersembunyi hasil generate dikirim untuk optimistic locking.

Saat update basi, UI:

- tidak menimpa data yang lebih baru;
- menampilkan message konflik yang dapat ditindaklanjuti;
- menawarkan muat ulang/tinjau alih-alih mengirim ulang secara diam-diam.

## Kontrak detail

Permukaan detail menampilkan:

- referensi akun stabil yang terenkode;
- username dan nama tampilan;
- email dan telepon opsional;
- preview avatar dengan fallback aman ketika tidak ada/tidak valid;
- status dan authorized-until, termasuk indikator turunan Kedaluwarsa;
- metadata created-at, updated-at, created-by, dan updated-by yang dikembalikan response detail dasar API;
- ringkasan role yang ditetapkan dari FD-IAM-002;
- Edit ditambah aksi siklus hidup yang valid;
- tanpa aksi hapus dan tanpa rahasia credential/token.

Type web hasil generate saat ini mengekspos `id` dan `version`, tetapi tidak
menyertakan field timestamp dan aktor dasar meskipun API mengembalikannya.
Komponen detail khusus menggunakan extended response type lokal sampai generator
memiliki typing metadata dasar umum; komponen tidak mengubah file type hasil
generate.

## Interaksi siklus hidup

Aktivasi/deaktivasi selalu menggunakan dialog konfirmasi dan version terbaru.
Keberhasilan menginvalidasi kueri daftar/detail Akun Pengguna. Response basi
memuat ulang state terbaru. Response idempoten ditampilkan sebagai sudah berada
dalam state yang diminta, bukan sebagai transisi kedua.

Teks deaktivasi menjelaskan bahwa session/token saat ini akan berhenti bekerja
dan reaktivasi tidak akan memulihkannya. Deaktivasi diri sendiri dinonaktifkan
di UI, tetapi juga ditolak API.

## Permission dan pengungkapan data

- Route hasil generate mendeklarasikan metadata resource/action dan backend tetap menjadi otoritas penegakan.
- Aktor tanpa READ tidak menerima konten list/detail.
- Aktor tanpa CREATE/UPDATE tidak melihat aksi terkait, tetapi UI tersembunyi tidak pernah dianggap sebagai otorisasi.
- Message error memuat code/field dan tidak pernah memuat detail credential atau token.

## State dependensi

- Provider role tersedia: render semua role yang ditetapkan sebagai satu set.
- Provider role tidak tersedia selama rollout bertahap: tampilkan state unavailable secara eksplisit; jangan pernah menyiratkan akun tidak memiliki role kecuali provider mengembalikan set kosong yang otoritatif.
- Pencabutan pending: akun tetap menampilkan INACTIVE dan akses tetap ditolak; status operasional dimiliki provider Session/Token, bukan detail UI rahasia.

## Aksesibilitas dan lokalisasi

- Status disampaikan melalui teks serta warna.
- Semua aksi dapat dijangkau dengan keyboard dan dialog konfirmasi mengelola focus.
- Avatar memiliki alt text bermakna atau semantik dekoratif sebagaimana mestinya.
- i18n Inggris/Indonesia hasil generate tetap dikendalikan definisi; message siklus hidup khusus dan state dependensi berada dalam namespace locale modul khusus non-generated.
