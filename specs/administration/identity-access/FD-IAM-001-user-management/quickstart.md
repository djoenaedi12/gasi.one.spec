# Panduan Validasi Cepat: Manajemen Pengguna

Panduan ini memvalidasi implementasi mendatang secara end-to-end. Panduan ini
tidak menggantikan [model data](./data-model.md),
[kontrak API](./contracts/api.openapi.yaml), atau
[kontrak integrasi](./contracts/integrations.md), serta tidak memuat body
implementasi atau test suite.

## Prasyarat

- Java 25 dan Maven.
- Node.js 20.19+ atau 22.12+ dan npm (juga memenuhi Node CLI >=18).
- MariaDB yang dapat dijangkau platform API.
- Repository sibling yang telah di-checkout: `gasi.one.cli`, `gasi.one.api`, dan `gasi.one.web`.
- Provider Authentication/aktor saat ini, pencabutan Session/Token/PAT, ringkasan Role, dan Audit tersedia untuk validasi integrasi/keamanan penuh.
- Satu aktor pengujian dengan permission UserAccount READ, CREATE, dan UPDATE serta satu aktor tanpa otorisasi.

Fase implementasi terlebih dahulu harus menambahkan file definisi yang
dijelaskan dalam [crud-definition.md](./contracts/crud-definition.md). Fase
perencanaan ini tidak membuatnya.

## 1. Validasi definisi sumber

Dari `gasi.one.cli`:

```bash
node ./bin/gasi-one.js plugin validate -f definitions/identity-access/plugin.json
node ./bin/gasi-one.js resource validate -f definitions/identity-access/resources.json
node ./bin/gasi-one.js contract validate -f definitions/identity-access/iam-user-contract.json
```

Hasil yang diharapkan:

- setiap command berakhir dengan exit code nol dan melaporkan schema valid;
- jumlah resource adalah satu;
- tidak ada field password, credential, failed-login, lockout, atau last-login;
- pengujian regresi CLI membuktikan `ui.customizationModule` dinormalisasi, divalidasi, dan dipancarkan, bukan diabaikan tanpa peringatan.

## 2. Tinjau plan generate sebelum menulis

```bash
node ./bin/gasi-one.js plugin plan -f definitions/identity-access/plugin.json -o ../gasi.one.api/plugins --target api
node ./bin/gasi-one.js plugin plan -f definitions/identity-access/plugin.json -o ../gasi.one.web/plugins --target web
node ./bin/gasi-one.js resource plan -f definitions/identity-access/resources.json -o ../gasi.one.api/plugins/identity-access-plugin --target api
node ./bin/gasi-one.js resource plan -f definitions/identity-access/resources.json -o ../gasi.one.web/plugins/identity-access-plugin --target web
node ./bin/gasi-one.js contract plan -f definitions/identity-access/iam-user-contract.json -o ../gasi.one.api/contracts --target api
```

Tinjau output yang diharapkan:

- kerangka plugin API, layer CRUD UserAccount, enum AccountStatus, migrasi, i18n, dan manifest hasil generate;
- kerangka plugin web, types/schema/service/hooks/routes/pages/i18n UserAccount, registry route, dan manifest hasil generate;
- tidak ada penyimpanan password/authentication/session/token/role-assignment hasil generate;
- tidak ada file dalam package backend non-generated `useraccountcustom` atau `src/custom` web yang dijadwalkan untuk ditimpa;
- route web menyertakan side-effect import kustomisasi yang dideklarasikan setelah peningkatan CLI selesai.

Hanya setelah peninjauan ini, jalankan command yang sama dengan mengganti
`plan` menjadi `sync`. API dan web harus tetap dijalankan secara terpisah.

## 3. Periksa migrasi dan manifest

Konfirmasikan migrasi create hasil generate dan `.gasi-one/manifest.json`
terhadap [data-model.md](./data-model.md).

Kondisi lulus:

- ID BIGINT non-auto-increment;
- constraint unik username dan kolom akun wajib;
- status akun disimpan sebagai string;
- authorized-until nullable;
- field timestamp/aktor/version standar;
- tanpa FK Employee/Person dan tanpa kolom milik authentication;
- migrasi ditandai create-once dan file khusus tidak ada dalam manifest.

## 4. Jalankan pemeriksaan repository

CLI:

```bash
cd ../gasi.one.cli
npm test
```

Plugin API:

```bash
cd ../gasi.one.api/plugins/identity-access-plugin
mvn test
mvn clean verify
```

Workspace web:

```bash
cd ../gasi.one.web
npm run build -w plugins/identity-access-plugin
npm run build
npm run lint
```

Plugin web hasil generate saat ini tidak memiliki script `test` atau `lint`
lokal plugin. Jika implementasi menambahkan salah satunya, dokumentasikan
sebelum menggunakannya sebagai gate wajib.

## 5. Jalankan platform terintegrasi

Build/package/install plugin menggunakan alur plugin yang didokumentasikan oleh
repository, lalu jalankan API dengan datasource MariaDB terkonfigurasi:

```bash
cd ../gasi.one.api
APP_PLUGINS_PATH=runtime-plugins mvn -pl platform-app spring-boot:run
```

Jalankan host web di terminal lain:

```bash
cd ../gasi.one.web
npm run dev
```

Proxy web mengirim traffic `/platform-app` ke API pada port 8080 secara default.
Gunakan database non-produksi dan credential pengujian.

## 6. Validasi pembuatan dan normalisasi

Dengan aktor yang diotorisasi:

1. Buat akun ACTIVE dengan username unik, nama tampilan, email, dan tanpa authorized-until.
2. Pastikan response memuat ID terenkode, username kanonis, status, metadata dibuat/diperbarui, dan version; response tidak memuat rahasia.
3. Coba username yang sama dengan perbedaan kapitalisasi dan whitespace luar.
4. Pastikan HTTP 422/409 sebagaimana mestinya, error khusus username, dan tidak ada row kedua.
5. Buat akun lain dengan nama tampilan sama dan username berbeda; pastikan berhasil.
6. Coba email yang tidak ada/tidak valid serta status yang tidak ada/tidak valid; pastikan field error dan tidak ada row.

Ini memvalidasi AC-US1-01 hingga AC-US1-06 dan SC-003.

## 7. Validasi pencarian, filter, detail, dan pengungkapan data

1. Buat record dengan username, nama tampilan, email, dan telepon yang berbeda.
2. Cari setiap atribut dari daftar hasil generate dan pastikan hasil yang diharapkan melalui server paging.
3. Terapkan filter Aktif dan Nonaktif serta pastikan keduanya dapat dikomposisikan dengan pencarian.
4. Buka detail dan pastikan identitas/kontak/avatar/status/authorized-until, timestamp/aktor, serta ringkasan role eksternal lengkap.
5. Pastikan kontrol hapus, credential, session token, nilai PAT, failed-login, lockout, atau last-login tidak ditampilkan.
6. Ulangi pemanggilan list/detail sebagai aktor tanpa READ; pastikan 401/403 dan tidak ada payload akun terlindungi.

Ini memvalidasi AC-US2-01 hingga AC-US2-05 dan SC-002/SC-004.

## 8. Validasi pembaruan profil dan optimistic locking

1. Muat satu response detail dan simpan ID/version-nya.
2. Perbarui username, nama tampilan, email, telepon, avatar, dan authorized-until.
3. Pastikan ID tidak berubah, username kanonis, updated-at/version berubah, dan status tidak berubah.
4. Coba username kanonis duplikat; pastikan ditolak dan data asli tetap ada.
5. Kirim dua update dengan version awal yang sama; pastikan yang kedua menerima HTTP 409 dan tidak menimpa yang pertama.
6. Kirim email kosong/tidak valid; pastikan HTTP 422 dan tidak ada perubahan data.

Ini memvalidasi AC-US3-01 hingga AC-US3-05.

## 9. Validasi siklus hidup dan pencabutan

1. Aktifkan akun INACTIVE yang valid; pastikan ACTIVE dan satu event REACTIVATE.
2. Ulangi activate; pastikan berhasil/no-op, version tidak berubah, dan tanpa event transisi duplikat.
3. Nonaktifkan akun ACTIVE yang memiliki artefak autentikasi aktif.
4. Pastikan INACTIVE telah di-commit, autentikasi baru langsung ditolak, dan artefak lama berhenti memberi otorisasi dalam satu menit.
5. Simulasikan ketidaktersediaan provider pencabutan; pastikan akun tetap INACTIVE dan setiap upaya akses berlanjut ditolak berdasarkan kelayakan.
6. Aktifkan kembali; pastikan artefak yang sebelumnya diinvalidasi tetap tidak valid.
7. Coba menonaktifkan diri sendiri; pastikan HTTP 422, state tidak berubah, dan tanpa event.
8. Coba aktivasi dengan authorized-until kedaluwarsa; pastikan ditolak.
9. Biarkan authorized-until berlalu pada akun ACTIVE; pastikan login/akses berlanjut ditolak dan pencabutan dimulai sementara status tersimpan tetap ACTIVE.
10. Coba siklus hidup sebagai aktor tanpa otorisasi; pastikan 401/403 dan tidak ada perubahan state.

Ini memvalidasi AC-US4-01 hingga AC-US4-07 dan AC-US4-10, ditambah SC-005/SC-006.

## 10. Validasi event audit

Untuk create, pembaruan profil, activate, deactivate, dan reactivate, pastikan
tepat satu event dengan:

- ID aktor stabil;
- waktu event;
- action yang benar;
- referensi akun target terenkode;
- nama field yang berubah secara exact;
- tanpa nilai before/after dan tanpa rahasia.

Pastikan operasi yang ditolak dan no-op siklus hidup tidak membuat event
transisi berhasil. Ini memvalidasi AC-US4-08 dan SC-007.

## 11. Validasi identitas historis dan proteksi penghapusan

1. Simpan referensi historis terotorisasi eksternal ke suatu akun.
2. Nonaktifkan akun tersebut.
3. Pastikan referensi tetap mengarah ke identitas stabil tanpa memberi akses.
4. Panggil endpoint DELETE warisan sebagai aktor berizin DELETE.
5. Pastikan HTTP 422 `PERMANENT_DELETE_NOT_ALLOWED`, row tetap ada, dan tidak ada audit sukses DELETE yang dicatat.

Ini memvalidasi AC-US4-09 dan FR-019.

## Kriteria selesai

- Semua skenario penerimaan terpetakan ke pemeriksaan otomatis atau integrasi yang lulus.
- Plan generator ditinjau dan kode hasil generate tetap tidak diubah.
- Build CLI/API/web lulus tanpa asumsi command yang tidak terdokumentasi.
- Kontrak dependensi keamanan dan audit wajib tersedia serta diuji.
- Tidak ada warning generator yang belum diselesaikan, overwrite pembaruan basi, pengungkapan rahasia, event siklus hidup duplikat, atau akses setelah tidak layak.
