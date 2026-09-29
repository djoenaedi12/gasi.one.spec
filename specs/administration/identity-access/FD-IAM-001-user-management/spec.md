# Spesifikasi Fitur: Manajemen Pengguna

**FD ID**: `FD-IAM-001`

**Domain**: Administrasi

**Subdomain**: Identitas & Akses

**Pemilik**: GASI

**Sistem Terdampak**: Aplikasi GASI:One

**Referensi Sumber**: Aplikasi sekarang GPS & GPC, hasil brainstorming IAM

**Branch Fitur**: `N/A`

**Dibuat**: 2026-09-25

**Status**: Draf

**Masukan**: Mendefinisikan administrasi akun pengguna dan perilaku siklus hidupnya untuk GASI:One.

## Tujuan dan Ruang Lingkup

User Management menyediakan satu tempat yang konsisten bagi administrator
berwenang untuk membuat, mencari, meninjau, memelihara, mengaktifkan, dan
menonaktifkan akun pengguna dalam aplikasi GASI:One. Setiap akun memiliki
identitas internal yang stabil.

### Dalam Ruang Lingkup

- Membuat akun pengguna.
- Menampilkan, mencari, memfilter, dan melihat akun pengguna.
- Mengelola identitas, kontak, avatar, status, dan masa otorisasi akun pengguna.
- Mengaktifkan, menonaktifkan, dan mengaktifkan kembali akun.
- Menghasilkan audit event untuk perubahan administratif akun.
- Memastikan deaktivasi akun menghentikan session aktif dan penggunaan token autentikasi pengguna.

### Di Luar Ruang Lingkup

- Mendefinisikan role, permission, menu, atau aturan segregation of duties.
- Menghitung effective permission untuk beberapa active role.
- Mengonfigurasi login channel, AppClient, identity provider, atau masa berlaku token.
- Enrollment MFA, verifikasi OTP/TOTP, dan account recovery.
- Penyimpanan password atau credential, penghitungan kegagalan login, account lockout, dan pencatatan waktu login terakhir.
- Pengelolaan session, trusted device, refresh token, atau Personal Access Token.
- Menyediakan penyimpanan, pencarian, dan peninjauan audit log lintas domain.
- Membentuk relasi langsung antara akun pengguna dan master employee atau person.

## Skenario Pengguna & Pengujian *(wajib)*

### User Story 1 - Membuat Akun Pengguna (Prioritas: P1)

Sebagai administrator pengguna yang berwenang, saya ingin membuat akun pengguna
dengan identitas login yang unik dalam GASI:One agar identitas akun tersebut
kemudian dapat memperoleh akses terkontrol ke GASI:One.

**Alasan prioritas ini**: Seluruh kapabilitas pengelolaan pengguna lainnya bergantung
pada akun yang valid dan dapat diidentifikasi secara unik.

**Pengujian Mandiri**: Buat akun menggunakan data wajib, termasuk email,
dan status pilihan yang valid, pastikan akun tersimpan sebagai akun GASI:One dengan status yang
dipilih, dan pastikan identitas login duplikat ditolak tanpa membuat akun lain.

**Skenario Penerimaan**:

1. **AC-US1-01** — **Given** administrator berwenang, username unik, display name dan email valid, serta status `Active` atau `Inactive` telah dipilih, **When** administrator membuat akun, **Then** akun dibuat dengan identitas stabil, status pilihan, serta waktu pembuatan dan pembaruan yang ditetapkan sistem.
2. **AC-US1-02** — **Given** sebuah username sudah ada, **When** administrator mengirim username yang sama dengan perbedaan hanya pada kapitalisasi huruf atau spasi di awal/akhir, **Then** pembuatan ditolak dan tidak ada akun duplikat yang dibuat.
3. **AC-US1-03** — **Given** sebuah akun sudah menggunakan suatu display name, **When** administrator membuat akun lain dengan display name yang sama, username yang unik, dan status yang valid, **Then** akun baru dibuat sebagai identitas mandiri dengan status yang dipilih.
4. **AC-US1-04** — **Given** actor tidak memiliki kewenangan membuat pengguna, **When** actor mencoba membuat akun, **Then** operasi ditolak dan tidak ada data akun yang dibuat.
5. **AC-US1-05** — **Given** administrator berwenang dan data akun lainnya valid, **When** administrator membuat akun tanpa memilih status atau menggunakan status selain `Active` dan `Inactive`, **Then** pembuatan ditolak dan tidak ada akun yang dibuat.
6. **AC-US1-06** — **Given** administrator berwenang dan data akun lainnya valid, **When** administrator membuat akun tanpa email atau dengan format email yang tidak valid, **Then** pembuatan ditolak dan tidak ada akun yang dibuat.

---

### User Story 2 - Mencari dan Meninjau Akun Pengguna (Prioritas: P1)

Sebagai administrator atau reviewer yang berwenang, saya ingin mencari akun
pengguna dan meninjau kondisi terkininya agar dapat menjawab kebutuhan dukungan
akses dan governance tanpa mengekspos credential secret.

**Alasan prioritas ini**: Administrator harus dapat mengidentifikasi akun yang tepat
sebelum mengubahnya, terutama ketika beberapa akun memiliki display name yang
serupa.

**Pengujian Mandiri**: Cari berdasarkan atribut identitas yang didukung, filter
berdasarkan status, buka salah satu hasil, dan pastikan detail akun hanya terlihat
oleh reviewer yang berwenang.

**Skenario Penerimaan**:

1. **AC-US2-01** — **Given** terdapat beberapa akun pengguna, **When** reviewer berwenang mencari berdasarkan username, display name, email, atau phone, **Then** akun yang cocok ditampilkan dengan informasi yang cukup untuk membedakannya.
2. **AC-US2-02** — **Given** terdapat akun dengan status siklus hidup berbeda, **When** reviewer memfilter berdasarkan status akun, **Then** hanya akun dengan status yang dipilih yang ditampilkan.
3. **AC-US2-03** — **Given** reviewer berwenang melihat akun pengguna, **When** reviewer membuka akun, **Then** identitas, kontak, avatar, status, authorized until, metadata waktu, dan ringkasan relasi akses non-secret ditampilkan tanpa credential atau token secret.
4. **AC-US2-04** — **Given** actor tidak memiliki kewenangan melihat akun pengguna, **When** actor meminta daftar atau detail akun, **Then** informasi akun tidak diungkapkan.
5. **AC-US2-05** — **Given** akun memiliki beberapa assigned role, **When** reviewer berwenang membuka ringkasan relasi aksesnya, **Then** seluruh assigned role ditampilkan sebagai satu set tanpa menyiratkan bahwa akun dibatasi hanya pada satu role.

---

### User Story 3 - Mengelola Informasi Akun Pengguna (Prioritas: P2)

Sebagai administrator pengguna yang berwenang, saya ingin memperbarui username,
display name, email, phone, avatar, atau authorized until sambil mempertahankan
identitas stabil akun.

**Alasan prioritas ini**: Informasi akun dapat berubah seiring waktu, tetapi perubahan
tidak boleh menciptakan duplikasi atau merusak identitas stabil akun.

**Pengujian Mandiri**: Perbarui informasi akun yang diizinkan, lalu pastikan data
divalidasi, keunikan username diperiksa ulang, waktu pembaruan berubah, dan
identitas stabil akun tetap sama.

**Skenario Penerimaan**:

1. **AC-US3-01** — **Given** akun sudah ada dan perubahan display name, email, phone, avatar, atau authorized until valid, **When** administrator berwenang menyimpan perubahan, **Then** akun menampilkan data baru, waktu pembaruan berubah, dan identitas stabil tetap sama.
2. **AC-US3-02** — **Given** administrator mengubah username menjadi nilai lain yang unik dalam GASI:One, **When** perubahan disimpan, **Then** username baru berlaku dan identitas stabil akun tetap tidak berubah.
3. **AC-US3-03** — **Given** username yang diusulkan setelah normalisasi sudah dimiliki akun lain, **When** perubahan dikirim, **Then** perubahan ditolak dan data akun semula tetap tidak berubah.
4. **AC-US3-04** — **Given** dua administrator mengubah versi akun yang sama, **When** administrator kedua menyimpan setelah perubahan pertama berhasil, **Then** perubahan kedua ditolak sebagai stale update dan administrator diminta meninjau kondisi terbaru.
5. **AC-US3-05** — **Given** akun sudah ada, **When** administrator memperbarui email menjadi kosong atau menggunakan format yang tidak valid, **Then** perubahan ditolak dan informasi akun tetap tidak berubah.

---

### User Story 4 - Mengendalikan Siklus Hidup Akun (Prioritas: P1)

Sebagai administrator pengguna yang berwenang, saya ingin mengaktifkan,
menonaktifkan, atau mengaktifkan kembali akun agar kelayakan akses mengikuti
kebutuhan bisnis terkini tanpa menghapus data identitas historis.

**Alasan prioritas ini**: Menghentikan akses secara cepat merupakan kontrol keamanan
dan operasional yang kritis.

**Pengujian Mandiri**: Aktifkan akun `Inactive` yang valid, nonaktifkan akun
tersebut ketika masih memiliki authentication artifact aktif, dan pastikan akses
berhenti; kemudian aktifkan kembali dan pastikan session atau token lama tidak
dipulihkan secara diam-diam.

**Skenario Penerimaan**:

1. **AC-US4-01** — **Given** akun `Inactive` memiliki seluruh data wajib dan authorized until kosong atau belum terlewati, **When** administrator berwenang mengaktifkannya, **Then** status berubah menjadi `Active` dan akun menjadi eligible untuk autentikasi sesuai login policy dan access policy yang berlaku.
2. **AC-US4-02** — **Given** akun berstatus `Active`, **When** administrator berwenang menonaktifkannya, **Then** autentikasi baru diblokir dan session serta token yang masih dapat digunakan diinvalidasi sesuai session policy dan token policy terkait.
3. **AC-US4-03** — **Given** akun sebelumnya telah dinonaktifkan, **When** administrator berwenang mengaktifkannya kembali, **Then** status berubah menjadi `Active`, tetapi session dan token yang sebelumnya telah diinvalidasi tetap tidak valid.
4. **AC-US4-04** — **Given** administrator sedang menggunakan akun yang sama dengan akun yang dikelola, **When** administrator mencoba menonaktifkan akun tersebut, **Then** operasi ditolak untuk mencegah self-lockout yang tidak disengaja.
5. **AC-US4-05** — **Given** akun sudah berstatus `Inactive`, **When** administrator berwenang meminta deaktivasi kembali, **Then** akun tetap `Inactive` dan tidak ada lifecycle transition duplikat yang dicatat.
6. **AC-US4-06** — **Given** actor tidak memiliki kewenangan mengelola siklus hidup akun, **When** actor mencoba mengaktifkan, menonaktifkan, atau mengaktifkan kembali akun, **Then** operasi ditolak dan kondisi akun tetap tidak berubah.
7. **AC-US4-07** — **Given** akun telah dinonaktifkan tetapi invalidasi session atau token belum dapat dikonfirmasi, **When** pengguna mencoba mengakses GASI:One menggunakan session atau token tersebut, **Then** akses ditolak karena akun berstatus `Inactive`.
8. **AC-US4-08** — **Given** pembuatan, perubahan informasi, aktivasi, deaktivasi, atau reaktivasi akun berhasil, **When** operasi selesai, **Then** sebuah audit event dihasilkan dengan actor, waktu, tindakan, referensi akun, dan nama field yang berubah tanpa menyertakan nilai sebelum atau sesudah perubahan.
9. **AC-US4-09** — **Given** akun `Inactive` masih direferensikan oleh business record historis, **When** proses berwenang me-resolve referensi tersebut, **Then** identitas stabil akun tetap tersedia tanpa mengaktifkan kembali akses.
10. **AC-US4-10** — **Given** akun berstatus `Active` memiliki authorized until yang telah terlewati, **When** pengguna mencoba melakukan autentikasi atau melanjutkan akses, **Then** akses ditolak dan invalidasi session serta token dimulai sesuai policy terkait.

### Kasus Tepi

- Username berbeda dari username yang sudah ada hanya pada kapitalisasi huruf, spasi di awal/akhir, atau aturan normalisasi lain yang ditetapkan.
- Dua akun memiliki display name yang sama tetapi username berbeda.
- Display name akun berubah tanpa mengubah identitas stabil akun.
- Administrator bertindak atas akun yang sudah berubah setelah dimuat.
- Deaktivasi akun diminta ketika pengguna masih memiliki session, trusted device, atau token aktif.
- Reaktivasi dilakukan setelah session atau token dicabut.
- Administrator mencoba menonaktifkan akun yang sedang digunakan untuk melakukan operasi.
- Akun `Inactive` tetap direferensikan oleh approval, audit event, atau business record historis lainnya.
- Layanan session atau token downstream tidak dapat segera mengonfirmasi invalidasi; akses harus fail closed sampai kondisi akun tersinkronisasi.
- Authorized until kosong sehingga akun tidak memiliki batas waktu akses.
- Authorized until terlewati ketika akun masih berstatus `Active` dan memiliki session atau token aktif.

## Kebutuhan *(wajib)*

### Kebutuhan Fungsional

- **FR-001**: Sistem MUST merepresentasikan setiap akun pengguna sebagai identitas yang stabil dalam GASI:One.
- **FR-002**: Akun MUST memiliki identitas stabil, username, display name, email, status, waktu pembuatan, dan waktu pembaruan; akun MAY memiliki phone, avatar, dan authorized until.
- **FR-003**: Hanya actor yang memiliki kewenangan membuat pengguna yang MAY membuat akun pengguna.
- **FR-004**: Pembuatan akun MUST mewajibkan username, display name, email, dan status serta MUST memvalidasi seluruh nilai wajib maupun opsional yang diberikan sebelum membuat akun.
- **FR-005**: Username MUST unik dalam GASI:One setelah aturan normalisasi diterapkan, termasuk normalisasi kapitalisasi huruf dan spasi di awal/akhir.
- **FR-006**: Status MUST menjadi input wajib saat pembuatan akun, MUST bernilai `Active` atau `Inactive`, dan akun MUST disimpan menggunakan status yang dipilih administrator.
- **FR-007**: Sistem MUST mengelola setiap akun sebagai identitas mandiri berdasarkan identitas stabil dan username unik, termasuk ketika beberapa akun memiliki display name yang sama.
- **FR-008**: Pengguna berwenang MUST dapat menampilkan, mencari, dan memfilter akun berdasarkan username, display name, email, phone, status, dan atribut lain yang didukung.
- **FR-009**: Detail akun MUST hanya mengekspos informasi akun, siklus hidup, metadata waktu, dan ringkasan relasi akses non-secret kepada actor yang berwenang.
- **FR-010**: Administrator berwenang MUST dapat memperbarui username, display name, email, phone, avatar, dan authorized until; seluruh perubahan MUST divalidasi sebelum disimpan.
- **FR-011**: Perubahan username MUST memvalidasi ulang keunikan dalam GASI:One dan MUST mempertahankan identitas stabil akun.
- **FR-012**: Pembaruan MUST mendeteksi kondisi akun yang stale dan MUST NOT menimpa perubahan administratif yang lebih baru secara diam-diam.
- **FR-013**: Hanya actor berwenang yang MAY mengaktifkan, menonaktifkan, atau mengaktifkan kembali akun.
- **FR-014**: Aktivasi MUST mewajibkan seluruh data akun mandatory dalam kondisi valid dan authorized until belum terlewati apabila nilainya tersedia.
- **FR-015**: Deaktivasi MUST segera membuat akun tidak eligible untuk autentikasi baru dan MUST memulai invalidasi session aktif serta token yang masih dapat digunakan.
- **FR-016**: Reaktivasi MUST NOT memulihkan session, status trusted device, atau token yang diinvalidasi selama deaktivasi.
- **FR-017**: Administrator MUST NOT dapat menonaktifkan akun yang sama dengan akun yang sedang digunakan untuk melakukan tindakan tersebut.
- **FR-018**: Pengulangan lifecycle request untuk kondisi akun yang sama MUST aman dan MUST NOT membuat transition event duplikat.
- **FR-019**: Akun pengguna MUST NOT dihapus permanen melalui feature ini; akun `Inactive` dan identitas stabilnya MUST tetap tersedia untuk referensi historis yang berwenang.
- **FR-020**: Akun MUST mendukung relasi dengan beberapa assigned role; definisi role, kombinasi active role yang kompatibel, dan effective authorization dikelola oleh kapabilitas role dan permission.
- **FR-021**: Setiap pembuatan, pembaruan informasi, aktivasi, deaktivasi, dan reaktivasi akun MUST menghasilkan audit event yang memuat actor, waktu, tindakan, referensi akun, dan nama field yang berubah; event MUST NOT memuat nilai sebelum atau sesudah perubahan.
- **FR-022**: Akses ke daftar, detail, dan tindakan akun MUST dibatasi berdasarkan permission actor yang berlaku.
- **FR-023**: Operasi yang ditolak MUST membiarkan kondisi akun tidak berubah dan memberikan alasan yang actionable tanpa mengungkap credential, token, atau nilai secret lainnya.
- **FR-024**: Deaktivasi akun MUST fail closed ketika authentication artifact downstream belum dapat dikonfirmasi invalid, sehingga akun tidak dapat terus mengakses GASI:One selama sinkronisasi.
- **FR-025**: Authorized until MAY dikosongkan untuk akses tanpa batas waktu; jika diisi dan waktunya telah terlewati, akun MUST tidak eligible untuk autentikasi atau kelanjutan akses meskipun berstatus `Active`, serta invalidasi session dan token MUST dimulai.

### Ketertelusuran Kebutuhan ke Penerimaan

| Requirement | Acceptance Coverage |
|---|---|
| FR-001–FR-002 | AC-US1-01, AC-US2-03 |
| FR-003–FR-007 | AC-US1-01–AC-US1-06 |
| FR-008–FR-009 | AC-US2-01–AC-US2-04 |
| FR-010–FR-012 | AC-US3-01–AC-US3-05 |
| FR-013–FR-014 | AC-US4-01, AC-US4-06 |
| FR-015–FR-016 | AC-US4-02, AC-US4-03, AC-US4-07 |
| FR-017–FR-018 | AC-US4-04, AC-US4-05 |
| FR-019 | AC-US4-09 |
| FR-020 | AC-US2-05 |
| FR-021 | AC-US4-08 |
| FR-022 | AC-US1-04, AC-US2-04, AC-US4-06 |
| FR-023 | AC-US1-02, AC-US3-03, AC-US3-04, AC-US4-06 |
| FR-024 | AC-US4-07 |
| FR-025 | AC-US4-01, AC-US4-10 |

### Aturan Bisnis

- **BR-001**: User Management memiliki identitas, kontak, avatar, status, masa otorisasi, dan metadata waktu akun; role, credential, serta kontrol akses dikelola oleh kapabilitas terpisah.
- **BR-002**: Setiap akun pengguna dikelola secara mandiri berdasarkan identitas stabil dan username; kesamaan display name tidak menyatukan identitas ataupun siklus hidup antar akun.
- **BR-003**: Pengguna dapat memiliki beberapa assigned role, dan sebuah session dapat menggunakan beberapa compatible active role; perhitungan tersebut berada di luar FD ini.
- **BR-004**: Deaktivasi menghapus kelayakan akses, tetapi tidak menghapus identitas atau referensi historis.
- **BR-005**: Secret seperti password, authentication factor, session token, dan nilai Personal Access Token tidak pernah menjadi bagian dari informasi akun User Management.

### Entitas Utama

- **User Account**: Akun pengguna dalam GASI:One yang memiliki identitas stabil, username unik, display name, email, phone opsional, avatar opsional, status, authorized until opsional, serta waktu pembuatan dan pembaruan.
- **User Account Audit Event**: Event yang dihasilkan atas pembuatan, perubahan informasi, aktivasi, deaktivasi, atau reaktivasi akun; berisi actor, waktu, tindakan, referensi akun, dan nama field yang berubah tanpa nilai sebelum atau sesudah perubahan.
- **Role Assignment**: Referensi ke satu atau lebih role yang terhubung dengan akun; perilaku role didefinisikan dalam FD role dan permission.

### Dependensi

- **FD-IAM-002 — Roles, Permissions & Menus**: Menyediakan role assignment, effective permission, multiple active role, dan constraint segregation of duties.
- **FD-IAM-004 — App Clients & Authentication Policies**: Menentukan dari mana dan berdasarkan policy apa akun `Active` dapat melakukan autentikasi.
- **FD-IAM-005 — MFA & Account Recovery**: Mengelola verifikasi serta penggunaan email atau phone untuk authentication factor dan account recovery.
- **FD-IAM-006 — Sessions, Tokens & Trusted Devices**: Menerapkan deaktivasi terhadap authentication artifact aktif.
- **FD-IAM-007 — Personal Access Token**: Memastikan status akun dan aturan revocation membatasi penggunaan PAT.
- **Cross-domain audit capability**: Menyimpan, mencari, dan menampilkan audit event kepada reviewer berwenang.

## Kriteria Keberhasilan *(wajib)*

### Hasil Terukur

- **SC-001**: Dalam usability validation, sedikitnya 95% administrator berwenang dapat membuat akun pengguna dengan username, display name, email, dan status yang valid pada percobaan pertama dalam waktu kurang dari dua menit.
- **SC-002**: Dalam usability validation, sedikitnya 95% reviewer berwenang dapat menemukan akun yang diketahui dan mengidentifikasi status terkininya dalam waktu kurang dari 30 detik.
- **SC-003**: Seluruh username duplikat yang diuji, termasuk variasi kapitalisasi huruf dan spasi di awal/akhir, ditolak tanpa membuat atau mengubah akun.
- **SC-004**: Seluruh operasi create, view, update, dan lifecycle tanpa kewenangan yang diuji ditolak tanpa mengungkap informasi akun yang dilindungi.
- **SC-005**: Akun yang dinonaktifkan langsung tidak dapat memulai interaksi terautentikasi baru, dan authentication artifact yang sebelumnya dapat digunakan berhenti memberikan akses dalam security window yang disetujui, dengan target tidak lebih dari satu menit.
- **SC-006**: Reaktivasi akun tidak pernah memulihkan session, status trusted device, atau token yang diinvalidasi oleh deaktivasi.
- **SC-007**: Seluruh pembuatan akun, perubahan informasi, dan lifecycle transition yang diuji menghasilkan audit event berisi actor, waktu, tindakan, referensi akun, dan nama field yang berubah, tanpa nilai sebelum atau sesudah perubahan.

## Asumsi

- Username merupakan identitas login yang mudah dibaca manusia dan unik dalam GASI:One; akun juga memiliki identitas stabil terpisah yang tidak berubah ketika username berubah.
- Username dapat diubah oleh administrator berwenang ketika persyaratan keunikan terpenuhi.
- Status awal akun dipilih administrator saat pembuatan dengan nilai `Active` atau `Inactive`; perilaku invitation atau self-activation tidak termasuk dalam FD ini.
- Email wajib tersedia sebagai kontak utama untuk account recovery; proses verifikasi dan pengiriman reset password dikelola oleh FD MFA & Account Recovery.
- Phone dan avatar bersifat opsional.
- Authorized until bersifat opsional; nilai kosong berarti akun tidak memiliki batas waktu akses.
- Waktu pembuatan dan pembaruan dikelola sistem dan tidak dapat diubah langsung oleh administrator.
- FD ini tidak memodelkan relasi akun dengan employee atau person; kebutuhan integrasi tersebut harus didefinisikan sebagai capability terpisah.
- Permanent deletion tidak termasuk agar referensi business historis tetap utuh; kewajiban retention dan privacy diatur oleh data policy organisasi.
- Project constitution masih berupa scaffold sehingga belum ada governance rule tambahan yang diratifikasi untuk diterapkan pada draft ini.
