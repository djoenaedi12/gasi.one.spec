# Riset Fase 0: Manajemen Pengguna

## Baseline Repository

Riset menggunakan repository yang tersedia di workspace, bukan asumsi tentang perilaku framework:

- `gasi.one.api`: Java 25, Spring Boot 4.0.3, PF4J, JPA, MariaDB, Flyway, MapStruct, controller/service CRUD dasar, resource hook terurut, registry capability, anotasi/kontrak audit, ID TSID, dan optimistic locking.
- `gasi.one.web`: React 19, TypeScript 5.9, Vite 7, React Router 7, TanStack Query/Table, React Hook Form, Zod, registry kustomisasi resource, dan halaman resource hasil generate.
- `gasi.one.cli`: generator resource/plugin/contract berbasis JSON. Command `validate`, `plan`, dan `sync` dikonfirmasi dari source serta eksekusi contoh yang berhasil. API dan web merupakan target terpisah.

Direktori `plugins` API/web saat ini belum berisi implementasi plugin Manajemen
Pengguna, Authentication, Session/Token, Role, atau Audit. Karena itu,
perencanaan membedakan kontrak framework yang tersedia dari perilaku dependensi
yang belum tersedia.

## Keputusan 1: Batas plugin dan generate

**Keputusan**: Generate plugin `identity-access` (`code: iam`) secara terpisah
ke repository API dan web, lalu generate resource `UserAccount` ke root plugin
tersebut. Simpan definisi di `gasi.one.cli/definitions/identity-access/`.

**Alasan**: Framework secara eksplisit menempatkan fitur bisnis di plugin,
migrasi plugin ditemukan oleh ekstensi Flyway hasil generate, dan CLI
menggunakan `pluginName` untuk mengelompokkan migrasi/i18n. Generator resource
tidak membaca JSON plugin, sehingga resource harus mendeklarasikan
`pluginName: iam` secara eksplisit.

**Alternatif yang dipertimbangkan**:

- Menempatkan Manajemen Pengguna di `platform-app`: ditolak karena dokumentasi repository menetapkan autentikasi, audit, dan fitur bisnis sebagai milik plugin.
- Membuat kerangka plugin API/web secara manual: ditolak karena `plugin sync` sudah membuat kerangka proyek yang didukung.
- Menghasilkan kedua target dalam satu command: ditolak karena CLI hanya menerima `--target api` atau `--target web` per eksekusi.

## Keputusan 2: Definisi CRUD bersifat otoritatif

**Keputusan**: Representasikan field, validasi hasil generate, enum, penyertaan
DTO, proyeksi, search fields, sort, layout, dan i18n dalam satu `resources.json`.
Selalu jalankan `validate`, periksa `plan`, lalu jalankan `sync` secara terpisah
untuk setiap target.

**Alasan**: Kode writer dan manifest CLI menunjukkan output Java/React memakai
strategi `overwrite`, migrasi memakai perilaku create-once/alter-on-change, dan
i18n memakai merge-properties. Perubahan pada file hasil generate akan hilang
saat sync dan membuat JSON berhenti menjadi source of truth.

**Alternatif yang dipertimbangkan**:

- Mengubah file controller/service/page hasil generate: ditolak oleh batasan pengguna dan perilaku overwrite generator.
- Memelihara JSON resource API dan web terpisah: ditolak karena kontrak field dan validasinya dapat menyimpang.

## Keputusan 3: Model field Akun Pengguna

**Keputusan**: Generate satu resource `UserAccount` dengan username, nama
tampilan, email, telepon, avatar, status akun, dan masa otorisasi. Gunakan tipe
dasar GASI untuk ID, timestamp, metadata aktor, dan version.

**Alasan**: Semua tipe skalar yang dibutuhkan didukung schema resource:
`string`, `enum` yang disimpan sebagai string, dan `instant`. Class dasar
entity/model/DTO telah menyediakan metadata teknis dan version optimistic lock.

**Alternatif yang dipertimbangkan**:

- Memodelkan credential, state failed-login, lockout, atau last-login pada tabel pengguna: ditolak karena secara eksplisit merupakan lingkup Authentication.
- Menggunakan kembali `lifecycle_status` warisan sebagai status akun: ditolak karena generator tidak mengekspos/mengonfigurasikannya dalam kontrak DTO/UI resource dan semantik teknisnya bukan state bisnis ACTIVE/INACTIVE fitur.
- Menghubungkan Akun Pengguna langsung ke Employee/Person: ditolak karena di luar ruang lingkup.

## Keputusan 4: Normalisasi dan keunikan username

**Keputusan**: Simpan username dalam bentuk kanonis: hapus whitespace Unicode
di awal/akhir dan ubah menjadi lowercase menggunakan `Locale.ROOT`. Service hook
khusus terurut berjalan sebelum hook unique-field dari generator. Pertahankan
`unique: true` agar validasi aplikasi hasil generate dan constraint unik database
sama-sama berlaku.

**Alasan**: Validasi unik hasil generate membandingkan nilai field sebagaimana
diberikan; validasi tersebut tidak melakukan trim atau case folding. Kanonisasi
sebelum hook itu memenuhi persyaratan case/trim, sedangkan constraint database
menangani race antar-request konkuren. Penyimpanan hanya username kanonis
menghindari kolom normalisasi tersembunyi kedua dan ambiguitas nilai tampilan
dibanding nilai login.

**Alternatif yang dipertimbangkan**:

- Hanya mengandalkan collation MariaDB: ditolak karena perilaku trim tidak dijamin dan konfigurasi collation dapat berbeda.
- Mempertahankan huruf asli ditambah `normalized_username`: ditolak sebagai state tambahan yang tidak perlu karena spesifikasi tidak mewajibkan preservasi kapitalisasi username.
- Mengubah service hook hasil generate: ditolak karena akan ditimpa.

## Keputusan 5: Validasi email

**Keputusan**: Konfigurasikan email sebagai wajib, maksimum 254 karakter, dan
`validation.email: true`. Email tidak unik karena spesifikasi tidak mewajibkan
keunikan.

**Alasan**: CLI menghasilkan Jakarta Validation, validasi Zod, panjang kolom,
dan message yang selaras untuk validasi field yang didukung. Penambahan keunikan
akan memperkenalkan batasan bisnis yang tidak ada dalam spesifikasi.

**Alternatif yang dipertimbangkan**:

- Validator email khusus: ditolak karena dukungan generator sudah mencukupi.
- Email unik: ditolak karena beberapa akun dengan alamat kontak yang sama tidak dilarang.

## Keputusan 6: Pembaruan profil dibanding aksi siklus hidup

**Keputusan**: Keluarkan status dari `UpdateRequest` hasil generate. Gunakan PUT
hasil generate untuk field profil dan dua endpoint aksi sempit untuk
activate/deactivate, masing-masing mewajibkan `version`. Reaktivasi adalah
aktivasi dari inactive ke active.

**Alasan**: PUT dasar hasil generate menangani pembaruan standar dan write basi
dengan benar. Resource service hook dapat memvalidasi sebelum/sesudah CRUD,
tetapi tidak dapat menambahkan route aksi, menghentikan transisi berulang tanpa
save/kenaikan version, atau membedakan secara bersih pembaruan profil dari
tujuan siklus hidup. Service/controller siklus hidup khusus yang sempit karena
itu dapat dibenarkan; komponen tersebut bukan pengganti manual untuk CRUD hasil
generate.

**Alternatif yang dipertimbangkan**:

- Mengubah status melalui PUT standar: ditolak karena request berulang tetap melakukan save/version dan semantik otorisasi/audit khusus transisi menjadi ambigu.
- Mengodekan aksi dalam query string dan memeriksa request context: ditolak sebagai konvensi transport rapuh yang tetap tidak dapat menghentikan save dasar.
- Mengganti service/controller CRUD hasil generate: ditolak karena hanya dua aksi bisnis yang memerlukan orkestrasi khusus.

## Keputusan 7: Penghapusan permanen

**Keputusan**: Pertahankan mode CRUD hasil generate, tolak setiap delete dalam
service hook terurut yang terpisah, dan hilangkan kontrol hapus melalui
kustomisasi resource web.

**Alasan**: CLI memiliki mode `crud`, `read`, dan `embed`, tetapi tidak memiliki
switch delete per operasi. `read` juga akan menghapus create/update yang
didukung. Service hook memiliki `beforeDeleteRequest`/`beforeDelete`, yaitu
ekstensi backend paling sempit. Penolakan backend tetap otoritatif meskipun
klien memanggil route DELETE hasil generate secara langsung.

**Alternatif yang dipertimbangkan**:

- Membiarkan DELETE melakukan soft delete: ditolak karena siklus hidup akun direpresentasikan field status eksplisit dan DELETE tidak boleh menyamar sebagai deactivate.
- Mengubah `BaseController`: ditolak karena perilaku ini lokal untuk resource.

## Keputusan 8: Masa otorisasi dan akses fail-closed

**Keputusan**: `authorizedUntil` null berarti tidak terbatas. Kelayakan adalah
`status == ACTIVE && (authorizedUntil == null || authorizedUntil > now)`.
Plugin User mengeksposnya melalui capability bertipe. Pemilik Authentication,
session, refresh-token, trusted-device, dan PAT harus mengevaluasinya saat login
dan akses berlanjut.

**Alasan**: Manajemen Pengguna memiliki fakta akun, sedangkan FD dependensi
memiliki artefak autentikasi dan penegakan kebijakan. Hal ini mempertahankan
batas plugin dan membuat akun tidak layak secara lokal sekalipun konfirmasi
pencabutan downstream tidak tersedia.

**Alternatif yang dipertimbangkan**:

- Secara otomatis mengubah ACTIVE menjadi INACTIVE saat waktu berlalu: ditolak karena kedaluwarsa adalah kondisi kelayakan, belum tentu transisi status administratif, dan akan membutuhkan scheduler yang tidak diminta.
- Hanya memercayai pencabutan: ditolak karena FR-024 secara eksplisit mewajibkan fail-closed ketika konfirmasi tidak tersedia.

## Keputusan 9: Urutan deaktivasi/pencabutan

**Keputusan**: Commit INACTIVE terlebih dahulu, lalu minta pencabutan idempoten
melalui capability FD-IAM-006/007 setelah transaction commit. Kegagalan
pencabutan dilaporkan/dimasukkan antrean oleh provider pemilik, tetapi tidak
pernah mengembalikan akun menjadi ACTIVE. Pengulangan deaktivasi tidak melakukan
save, event, atau request kedua.

**Alasan**: Pemanggilan pencabutan di dalam transaksi akun dapat menghasilkan
rollback yang tidak aman (artefak tetap dapat digunakan saat akun kembali
ACTIVE) atau kegagalan parsial lintas-sistem. Commit lebih dahulu mempertahankan
source of truth lokal yang fail-closed. Path autentikasi harus memeriksa
kelayakan secara independen.

**Alternatif yang dipertimbangkan**:

- Mencabut sebelum pembaruan database: ditolak karena kegagalan database berikutnya mencabut artefak untuk akun yang tetap ACTIVE.
- Melempar error dan rollback saat pencabutan tidak tersedia: ditolak karena melanggar state akun fail-closed.
- Mengimplementasikan penyimpanan session/token dalam plugin ini: ditolak sebagai lingkup FD-IAM-006/007.

## Keputusan 10: Kepemilikan audit dan celah kontrak

**Keputusan**: Pertahankan `@AuditResource` hasil generate untuk CRUD dan beri
anotasi `@AuditAction` pada aksi siklus hidup. Wajibkan kontrak dependensi audit
mendukung aktor, occurred-at, aksi, referensi akun terenkode, serta himpunan nama
field yang berubah tanpa nilainya. Jangan implementasikan penyimpanan audit di
sini.

**Alasan**: `AuditLogEntry` saat ini hanya memuat action/module/resource/id dan
description; `AuditLogExtension` hanya dapat menyesuaikan description. Keduanya
tidak dapat membawa nama field yang berubah secara exact. Mengklaim anotasi
yang tersedia telah memenuhi FR-021 secara penuh adalah keliru. Plugin audit
didokumentasikan secara eksplisit sebagai pemilik perilaku intersepsi,
persistensi, dan kueri.

**Alternatif yang dipertimbangkan**:

- Menaruh nilai yang berubah dalam description: ditolak karena nilai dilarang dan teks tidak terstruktur bukan kontrak yang diminta.
- Menambahkan tabel audit-event ke Manajemen Pengguna: ditolak karena audit lintas-domain memiliki penyimpanannya.
- Mencatat event UPDATE otomatis dan transisi manual sekaligus: ditolak karena satu operasi tidak boleh membuat duplikat; integrasi audit harus menekan atau mengklasifikasikan event generik untuk method siklus hidup khusus.

## Keputusan 11: Role yang ditetapkan

**Keputusan**: Jangan menambahkan relasi/tabel role ke resource Akun Pengguna.
Render ringkasan role read-only yang disediakan FD-IAM-002 saat kontrak dan API
tersedia.

**Alasan**: Penetapan role, beberapa role aktif, perhitungan permission, dan
segregation-of-duties secara eksplisit didelegasikan. Relasi lokal akan
menggandeng persistensi plugin dan menyatakan kepemilikan secara keliru.

**Alternatif yang dipertimbangkan**:

- Menghasilkan field role many-to-one: ditolak karena seorang pengguna mendukung beberapa role yang ditetapkan.
- Menghasilkan koleksi role embedded: ditolak karena siklus hidup role bukan milik fitur ini.

## Keputusan 12: Celah bootstrap kustomisasi web

**Keputusan**: Gunakan `registerResourceCustom` yang tersedia untuk
filter/aksi/komposisi halaman, lalu tambahkan satu properti metadata resource
deklaratif di CLI yang menghasilkan side-effect import ke `src/routes.ts` hasil
generate. Modul kustom yang dirujuk bersifat non-generated.

**Alasan**: Registry mendukung filter daftar, pemfilteran row action, dan
penggantian halaman. Namun, `src/index.ts` plugin, index resource, dan
`src/routes.ts` resource hasil generate semuanya ditimpa; saat ini belum ada
bootstrap non-generated yang aman dan dapat dijangkau. Penyisipan import secara
manual akan melanggar keamanan regenerasi. Peningkatan kecil generator membuat
extension point dapat digunakan sambil mempertahankan otoritas JSON.

**Alternatif yang dipertimbangkan**:

- Mengubah `src/routes.ts` hasil generate setelah setiap sync: ditolak.
- Melakukan fork halaman hasil generate dan berhenti menyinkronkannya: ditolak karena meninggalkan kepemilikan generator.
- Menempatkan UI Manajemen Pengguna di platform-app: ditolak karena fitur dimiliki plugin.

## Keputusan 13: Kerangka kontrak dan arah dependensi

**Keputusan**: Gunakan `contract sync` hanya untuk kerangka kontrak kelayakan
Akun Pengguna jika kontrak disetujui sebagai milik Manajemen Pengguna, lalu
tambahkan interface/record secara manual sesuai maksud dokumentasi CLI. Konsumsi
kontrak session, authentication, role, dan audit dengan scope Maven `provided`
serta metadata dependensi PF4J; jangan pernah bergantung pada JAR implementasi.

**Alasan**: Generator kontrak CLI sengaja hanya membuat kerangka Maven karena
capability bisnis memerlukan desain antarmuka yang disengaja. Framework
`CapabilityRegistry` adalah jembatan runtime yang didukung.

**Alternatif yang dipertimbangkan**:

- Menempatkan antarmuka lintas-plugin di `core-api`: ditolak karena kontrak khusus domain seharusnya berada di `contracts/*`.
- Mengimpor service/entity plugin lain: ditolak oleh aturan dependensi repository.

## Keputusan 14: Command pengujian dan validasi

**Keputusan**: Gunakan hanya command yang tersedia di repository. Pemeriksaan
JSON CLI menggunakan `node ./bin/gasi-one.js ... validate`, inspeksi generate
menggunakan `... plan`, dan penulisan menggunakan `... sync`. Pengujian CLI
menggunakan `npm test`; pemeriksaan plugin API menggunakan Maven; pemeriksaan
plugin/build web menggunakan script build/lint workspace yang tersedia.

**Alasan**: Probe yang berhasil mengonfirmasi command validate dan plan untuk
plugin/resource/contract. Plugin web hasil generate hanya memiliki `build`,
sehingga command test atau lint lokal plugin tidak boleh dinyatakan tersedia
sebelum ditambahkan secara eksplisit.

**Alternatif yang dipertimbangkan**:

- Menggunakan `generate` atau target gabungan yang tidak terdokumentasi: ditolak karena command tersebut tidak ada.
- Menggunakan `clean` dalam alur normal: ditolak karena menghapus artefak hasil generate dan tidak diperlukan untuk rencana ini.
