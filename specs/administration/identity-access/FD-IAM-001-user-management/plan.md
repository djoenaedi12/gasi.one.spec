# Rencana Implementasi: Manajemen Pengguna

**Branch**: `FD-IAM-001-user-management` | **Tanggal**: 2026-09-29 | **Spesifikasi**: [spec.md](./spec.md)

**Masukan**: Spesifikasi fitur dari `specs/administration/identity-access/FD-IAM-001-user-management/spec.md`

## Ringkasan

Implementasikan Manajemen Pengguna sebagai plugin GASI:One `identity-access`.
Permukaan CRUD Akun Pengguna standar, DTO, adapter persistensi, migrasi Flyway,
form/halaman web, kueri daftar, optimistic locking, validasi, dan pemeriksaan
izin CRUD dihasilkan dari satu definisi resource JSON di `gasi.one.cli`. File
hasil generate tidak pernah diubah secara langsung.

Perilaku khusus resource yang sesuai dengan extension point yang tersedia
ditempatkan di luar path hasil generate: normalisasi username kanonis,
penolakan penghapusan permanen, publikasi kelayakan akun, dan kustomisasi
resource UI. Transisi siklus hidup menggunakan service/controller aksi khusus
yang kecil karena hook CRUD saat ini tidak dapat menghentikan transisi idempoten,
mengekspos endpoint aksi, atau menerapkan semantik audit khusus transisi.
Pencabutan session/token, penegakan autentikasi, ringkasan role, dan persistensi
audit tetap menjadi dependensi FD/plugin pemiliknya.

## Konteks Teknis

**Bahasa/Versi**: Java 25 untuk kode API/plugin; TypeScript 5.9 dan React 19 untuk kode plugin web; Node.js 20.19+ atau 22.12+ untuk workspace web (CLI mendeklarasikan Node.js >=18)

**Dependensi Utama**: Spring Boot 4.1.1, Spring Data JPA, otorisasi method Spring Security, Jakarta Validation, MapStruct 1.6.3, PF4J 3.15.0, Flyway, driver MariaDB, React Router 7, TanStack Query/Table, React Hook Form, Zod 4, Vite 7, serta GASI:One `core-api`/`core-starter`/`core-ui`

**Baseline Framework**: [Baseline GASI:One](../../../../references/gasi-one/baseline.md), diverifikasi terhadap commit API `88417135daeeb2b10c2df3fb8c23d345b734849c`, web `cce0c54cdc1f7e8a0fd2e4b42123a034720bf6b8`, dan CLI `fcc740631b838aed241d66619ebc9a69e853e263`

**Penyimpanan**: Skema MariaDB yang dikelola oleh migrasi Flyway dalam lingkup plugin; primary key `BIGINT` berbasis TSID dan ID string terenkode pada antarmuka publik

**Pengujian**: Maven/JUnit untuk plugin API dan pengujian integrasi; test runner bawaan Node untuk CLI; build TypeScript/Vite serta pengujian React terfokus untuk kustomisasi web; pengujian kontrak/keamanan dengan provider autentikasi, session/token, role, dan audit

**Platform Target**: Aplikasi web modular GASI:One; host API Spring Boot pada runtime kompatibel Linux dan klien browser React/Vite

**Jenis Proyek**: Aplikasi web modular multi-repository dengan plugin backend/frontend hasil generate dan modul kontrak bersama

**Target Performa**: Pencarian administratif berhalaman menggunakan kontrak page framework (default 20 untuk resource ini, maksimum API 100); pencarian akun yang diketahui dan tampilan status mendukung target hasil pengguna <30 detik dalam spesifikasi; akun tidak layak ditolak pada pemeriksaan terautentikasi berikutnya dan pencabutan artefak memenuhi batas keamanan <=1 menit

**Batasan**: Definisi CRUD JSON adalah source of truth; file Java/React hasil generate menggunakan strategi overwrite dan tidak boleh diubah; penghapusan permanen dilarang; keunikan username tidak peka huruf besar/kecil setelah trim; pembaruan basi memerlukan `version`; kegagalan pencabutan downstream harus fail-closed; tidak ada field credential, failed-login, lockout, atau last-login yang diperkenalkan

**Skala/Ruang Lingkup**: Satu agregat `UserAccount` beserta permukaan administratifnya; ukuran direktori tidak ditentukan sehingga semua path daftar UI/API tetap berhalaman di server dan berbasis proyeksi, bukan memuat seluruh populasi pengguna

## Pemeriksaan Konstitusi

*GATE: Harus lulus sebelum riset Fase 0. Diperiksa kembali setelah desain Fase 1.*

File konstitusi masih berupa kerangka yang belum diratifikasi dan berisi
placeholder, sehingga belum menyediakan prinsip proyek atau gate numerik yang
dapat diberlakukan. Karena itu, rencana ini menggunakan batasan fitur eksplisit
sebagai gate operasional:

| Gate | Hasil pra-desain | Hasil pasca-desain |
|---|---|---|
| CRUD standar mengutamakan generator | LULUS — command serta template resource/plugin/contract telah diperiksa dan diuji dengan dry-run | LULUS — CRUD standar tetap dihasilkan dari JSON |
| JSON tetap menjadi source of truth CRUD hasil generate | LULUS | LULUS — seluruh pilihan field/layout hasil generate ditentukan dalam kontrak definisi |
| Tidak mengubah kode hasil generate secara langsung | LULUS | LULUS — file khusus berada di luar path milik manifest; celah bootstrap CLI ditangani pada generator, bukan dengan menambal output |
| Hook/ekstensi sebelum implementasi khusus | LULUS | LULUS — hook dan `registerResourceCustom` mencakup perilaku lokal resource; kode siklus hidup khusus dibatasi pada semantik aksi yang belum didukung |
| Implementasi manual memiliki alasan terdokumentasi | LULUS | LULUS — hanya orkestrasi aksi siklus hidup, antarmuka capability stabil, dan satu peningkatan bootstrap ekstensi CLI yang manual |
| Data milik Authentication tetap di luar FD-IAM-001 | LULUS | LULUS — credential, failed login, lockout, dan last login hanya dependensi |

Tidak ada pelanggaran konstitusi yang memerlukan entri Pelacakan Kompleksitas.

## Klasifikasi Implementasi

| Area Manajemen Pengguna | Klasifikasi | Pendekatan yang direncanakan |
|---|---|---|
| Kerangka plugin dan CRUD Akun Pengguna standar | 1. Konfigurasi generator | `plugin.json` ditambah `resources.json`; generate API dan web secara terpisah |
| ID stabil, timestamp, metadata aktor, dan `version` optimistis | 1. Konfigurasi generator | Perilaku model/entity/DTO dasar GASI yang diwarisi |
| Field username/nama tampilan/email/telepon/avatar/status/masa otorisasi | 1. Konfigurasi generator | Metadata field resource, DTO, proyeksi, validasi, enum, layout UI, dan i18n |
| Email wajib dan validasi format | 1. Konfigurasi generator | `required: true`, `validation.email: true`, panjang maksimum selaras dengan migrasi |
| Kanonisasi username | 2. Hook/ekstensi framework | Service hook terurut melakukan trim dan lowercase dengan `Locale.ROOT` sebelum validasi unik hasil generate |
| Keunikan username dan perlindungan race condition | 1 + 2 | Validasi `unique: true`/constraint DB hasil generate yang bekerja pada nilai hasil kanonisasi hook |
| Pencarian berdasarkan username/nama tampilan/email/telepon dan filter status | 1 + 2 | Global search fields hasil generate; `registerResourceCustom.useListFilters` menyediakan filter status enum exact |
| Buat/baca/perbarui profil dan penanganan pembaruan basi | 1. Konfigurasi generator | Endpoint hasil generate dan DTO update wajib `version`; status tidak disertakan dalam DTO update standar |
| Aktifkan/nonaktifkan/aktifkan kembali | 3. Implementasi khusus | Controller/service aksi siklus hidup yang sempit; diperlukan karena hook CRUD tidak dapat mengekspos endpoint aksi atau mengembalikan no-op idempoten tanpa menyimpan |
| Proteksi nonaktifkan diri sendiri dan validitas aktivasi | 3 + 4 | Service siklus hidup memvalidasi ID stabil aktor dan data akun; identitas aktor berasal dari kontrak FD Authentication |
| Larangan penghapusan permanen | 2. Hook/ekstensi framework | Service hook menolak DELETE hasil generate; kustomisasi web menghapus aksi hapus |
| Tombol siklus hidup web, penghilangan hapus, tampilan metadata/avatar | 2. Hook/ekstensi framework | `registerResourceCustom`; gunakan kembali hook/konten hasil generate jika memungkinkan |
| Bootstrap kustomisasi web yang aman | 3. Implementasi khusus di CLI | Tambahkan import modul kustomisasi deklaratif yang tervalidasi karena seluruh file entry/route plugin yang dapat dijangkau saat ini dimiliki generator |
| Produksi event audit | 1 + 2 + 4 | `@AuditResource` hasil generate untuk CRUD, `@AuditAction` untuk siklus hidup, ekstensi audit untuk metadata aman; plugin audit memiliki intersepsi/penyimpanan dan harus menambahkan dukungan field yang berubah |
| Invalidasi session/token | 2 + 4 | Integrasi siklus hidup melalui capability bertipe setelah state akun di-commit; provider/retry dimiliki FD-IAM-006/007 |
| Penolakan akses untuk akun Nonaktif atau kedaluwarsa | 2 + 4 | Plugin User mengekspos capability kelayakan; dependensi Authentication/Session/PAT harus memanggilnya saat login dan akses berlanjut |
| Ringkasan role yang ditetapkan | 4. Dependensi ke FD lain | FD-IAM-002 menyediakan kueri penetapan role; Manajemen Pengguna hanya merender ringkasan non-rahasia jika tersedia |
| Password, failed login, lockout, last login | 4. Dependensi ke FD lain | Dimiliki Authentication; tidak ada field, migrasi, DTO, atau kontrol UI dalam fitur ini |

## Struktur Proyek

### Dokumentasi (fitur ini)

```text
specs/administration/identity-access/FD-IAM-001-user-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── api.openapi.yaml
│   ├── crud-definition.md
│   ├── integrations.md
│   └── web-ui.md
└── tasks.md                 # tidak dibuat oleh command ini
```

### Repository generator CRUD (`../gasi.one.cli`)

```text
definitions/identity-access/
├── plugin.json                         # descriptor plugin/masukan sumber
├── resources.json                      # source of truth CRUD hasil generate
└── iam-user-contract.json              # descriptor kerangka kontrak, jika kontrak milik fitur disetujui

src/schema/resource/
├── normalize.js                        # dukungan metadata modul kustomisasi yang direncanakan
└── validate.js                         # validasi path modul relatif yang aman
src/targets/resource/web/
└── context.js                          # menghasilkan import bootstrap ke modul routes hasil generate
templates/resource/web/
└── routes.ts.hbs                       # target import hasil generate
test/
└── resource-example.test.js            # cakupan regresi generator
README.md                               # dokumentasikan metadata baru setelah tersedia
```

File source/doc/test CLI di atas terdampak hanya oleh celah bootstrap
kustomisasi yang telah diverifikasi. Tidak ada command baru yang direka:
command `plugin|resource|contract validate|plan|sync` yang tersedia tetap
menjadi antarmuka.

### Repository backend (`../gasi.one.api`)

```text
contracts/iam-user-contract/             # kerangka hasil generate, lalu hanya antarmuka/value object stabil

plugins/identity-access-plugin/
├── pom.xml                              # dihasilkan dari plugin.json
├── src/main/java/gasi/one/plugins/iam/
│   ├── extension/                       # ekstensi PF4J/Flyway/i18n hasil generate
│   └── useraccount/
│       ├── application/dto/             # DTO CRUD hasil generate
│       ├── application/mapper/          # mapper CRUD hasil generate
│       ├── application/service/         # service CRUD hasil generate
│       ├── domain/model/                # model hasil generate + enum AccountStatus
│       ├── domain/port/                 # port CRUD hasil generate
│       ├── infrastructure/              # entity/repository/mapper hasil generate
│       └── presentation/controller/     # controller CRUD hasil generate
├── src/main/java/gasi/one/plugins/iam/useraccountcustom/
│   ├── UserAccountPolicyHook.java       # normalisasi, validasi pembuatan aktif, penolakan hapus
│   ├── UserAccountLifecycleService.java # orkestrasi transisi non-generated
│   ├── UserAccountLifecycleController.java
│   ├── UserEligibilityProvider.java     # ekstensi CapabilityProvider
│   ├── UserAccountAuditDescriptionExtension.java # hanya SPI deskripsi saat ini
│   └── UserAccountAuditEventAdapter.java # adapter field berubah setelah kontrak audit berkembang
├── src/main/resources/db/migration/iam/
│   └── V...__create_iam_user_accounts.sql # dihasilkan sekali; ditinjau sebelum diterapkan
├── src/main/resources/i18n/iam/          # message hasil generate/merge
└── src/test/java/gasi/one/plugins/iam/   # pengujian hook, siklus hidup, API, integrasi
```

Package lifecycle/custom sengaja berada di luar tree resource
`...useraccount...` hasil generate yang tercatat di `.gasi-one/manifest.json`.

Modul milik dependensi tidak diimplementasikan oleh FD ini. Saat tersedia,
descriptor plugin mendeklarasikan kontrak stabilnya dengan scope `provided` dan
metadata PF4J `dependsOn`; descriptor tidak pernah mengimpor implementasi plugin
lain.

### Repository frontend (`../gasi.one.web`)

```text
plugins/identity-access-plugin/
├── package.json                         # kerangka plugin hasil generate
├── src/index.ts                         # registrasi plugin hasil generate
├── src/routes.ts                        # registry route resource hasil generate
├── src/features/user-accounts/          # types/schema/service/hooks/pages/i18n CRUD hasil generate
└── src/custom/user-account.custom.tsx   # registrasi ResourceCustom non-generated
```

`user-account.custom.tsx` menghapus aksi hapus, menambahkan filter status dan
aksi siklus hidup, menampilkan metadata audit/avatar secara aman, serta menyusun
dependensi ringkasan role. File ini dimuat melalui import deklaratif JSON yang
direncanakan; file halaman hasil generate tidak diubah.

### Repository spesifikasi (`gasi.one.spec`)

Hanya artefak perencanaan yang tercantum di atas yang dibuat pada fase ini.
Tidak ada file implementasi atau `tasks.md` yang dibuat.

**Keputusan Struktur**: Gunakan satu plugin `identity-access` hasil generate di
setiap repository runtime, didukung definisi yang disimpan di repository CLI.
Kontrak lintas-plugin yang stabil berada di `contracts/*` API; perilaku lokal
resource berada dalam file hook/ekstensi di luar path hasil generate;
implementasi dependensi tetap berada di plugin pemiliknya.

## Desain Definisi CRUD

Bentuk terperinci yang selaras dengan validasi tersedia di
[contracts/crud-definition.md](./contracts/crud-definition.md). Resource yang
direncanakan memiliki `name: UserAccount`, `pluginName: iam`, package
`gasi.one.plugins.iam.useraccount`, tabel `iam_user_accounts`, endpoint
`/user-accounts`, mode `crud`, serta layout list/create/edit/detail eksplisit.

Pilihan utama:

- `username`: string, wajib, unik, diproyeksikan, maksimum 100, dinormalisasi oleh hook.
- `displayName`: string, wajib, diproyeksikan, maksimum 150; tidak unik.
- `email`: string, wajib, diproyeksikan, maksimum 254, validasi email; tidak unik.
- `phone`: string opsional, dapat dicari, maksimum 32.
- `avatar`: referensi/URI string opsional, berorientasi detail, maksimum 2048.
- `status`: enum wajib yang disimpan sebagai string `ACTIVE|INACTIVE`, diproyeksikan; hanya ada pada DTO mutasi create hasil generate.
- `authorizedUntil`: `instant` opsional; null berarti tanpa batas waktu.
- Field dasar hasil generate menyediakan ID, timestamp/aktor audit, dan version.
- `ui.list.searchFields` tepat berisi username, displayName, email, dan phone.
- Filter status exact dan aksi siklus hidup adalah ekstensi UI, bukan filter JSON rekaan.

## Model Data dan Migrasi

Lihat [data-model.md](./data-model.md). Migrasi hasil generate membuat
`iam_user_accounts` dengan primary key `BIGINT` non-auto-increment, kolom
audit/version/lifecycle standar GASI, tujuh kolom data Akun Pengguna,
constraint unik database pada `username` kanonis, dan tanpa cascade hard-delete.

Migrasi create hasil generate dibuat satu kali. Perubahan field JSON berikutnya
menggunakan snapshot manifest untuk menghasilkan migrasi alter; setiap perubahan
keunikan/tipe ditinjau karena dokumentasi CLI secara eksplisit memperingatkan
bahwa SQL alter tersebut mungkin memerlukan penanganan constraint secara manual.
Migrasi tidak pernah diubah setelah deployment.

## Kontrak API

Kontrak lengkap tersedia di
[contracts/api.openapi.yaml](./contracts/api.openapi.yaml).

- Endpoint CRUD/kueri hasil generate: `POST /api/v1/user-accounts`, `GET /api/v1/user-accounts/{id}`, `PUT /api/v1/user-accounts/{id}`, serta endpoint POST kueri list/page/one.
- DELETE hasil generate tetap dapat dirutekan pada level framework tetapi selalu ditolak dengan `PERMANENT_DELETE_NOT_ALLOWED`; endpoint tersebut tidak hadir dalam perilaku UI normal.
- Aksi khusus: `POST /api/v1/user-accounts/{id}/activate` dan `/deactivate`, masing-masing dengan `version` wajib dan semantik no-op idempoten.
- ID publik berupa string terenkode; `version` melindungi mutasi profil dan siklus hidup dari pembaruan basi.
- Read/create/update menggunakan `UserAccount:READ|CREATE|UPDATE`; izin DELETE tidak dapat mengesampingkan aturan tanpa penghapusan fitur ini.
- Error API menggunakan bentuk framework `ApiResponse.errors[]` dengan code, field opsional, dan message yang dapat ditindaklanjuti.

## Kontrak Web/UI

Lihat [contracts/web-ui.md](./contracts/web-ui.md).

Layar hasil generate menyediakan daftar berhalaman di server, global search,
create, edit, dan detail. Create menyertakan status wajib; edit tidak menyertakan
status dan menggunakan aksi siklus hidup eksplisit. Filter status menggabungkan
filter `EQUALS` dengan global search. Kontrol hapus dihilangkan. Detail
menampilkan identitas/kontak/avatar, status, masa otorisasi, metadata
dibuat/diperbarui, aksi berbasis version, dan ringkasan role yang ditetapkan dari
FD-IAM-002 tanpa mengekspos rahasia.

## Validasi dan Siklus Hidup

- Username dikanonisasi dengan penghapusan whitespace yang aman untuk Unicode dan lowercase `Locale.ROOT` sebelum pemeriksaan keunikan hasil generate dan persistensi.
- Hook kebijakan request yang sama melakukan trim pada string kontak/tampilan non-identitas, mengubah nilai opsional kosong menjadi null, dan menolak pembuatan sebagai ACTIVE jika `authorizedUntil` telah kedaluwarsa.
- Aturan Bean Validation/Zod hasil generator menegakkan field wajib, panjang, pola username, dan format email pada klien yang didukung serta API.
- Aktivasi membutuhkan data wajib yang valid dan `authorizedUntil > now` jika nilainya ada.
- Deaktivasi menolak ID akun stabil milik aktor saat ini, mempersistensikan `INACTIVE`, lalu meminta pencabutan artefak melalui capability dependensi. Kegagalan mengonfirmasi pencabutan tidak mengaktifkan kembali akun.
- Pengulangan aksi yang sudah tercermin pada akun mengembalikan detail saat ini tanpa menyimpan, menaikkan version, mencabut lagi, atau menghasilkan event audit transisi lain.
- Reaktivasi tidak pernah memulihkan artefak. Aksi ini hanya mengubah kelayakan autentikasi baru sesuai kebijakan Authentication.
- `authorizedUntil` yang kedaluwarsa membuat akun ACTIVE tidak layak. Pemilik Authentication/Session/PAT harus memeriksa capability kelayakan saat login dan akses berlanjut serta memulai alur pencabutan mereka sendiri.

## Audit dan Integrasi Lintas-FD

Lihat [contracts/integrations.md](./contracts/integrations.md).

Kontrak audit saat ini dapat menandai operasi CRUD/aksi dan menyesuaikan
deskripsi, tetapi tidak dapat membawa himpunan nama field yang berubah sesuai
kebutuhan. Karena itu, rencana ini tidak mengklaim kepatuhan audit penuh hanya
dari `@AuditResource`. Capability audit lintas-domain harus memperluas kontrak
event agar menerima aktor, occurred-at, aksi, referensi akun terenkode, dan nama
field, tanpa nilai sebelum/sesudah. Manajemen Pengguna menyediakan nama-nama
tersebut; plugin audit memiliki intersepsi, penyimpanan durable, dan UI kueri.

Demikian pula, FD-IAM-006/007 memiliki eksekusi pencabutan dan retry,
FD-IAM-004 memiliki principal autentikasi/aktor saat ini serta kelayakan login,
dan FD-IAM-002 memiliki penetapan role. Provider wajib yang belum tersedia
menjadi gate startup/integrasi, bukan alasan menyalin logikanya ke Manajemen
Pengguna.

## Strategi Pengujian

1. **Pengujian kontrak generator**: validasi normalisasi/validasi untuk JSON CRUD dan modul kustomisasi yang direncanakan; snapshot/periksa plan API dan web; pastikan file dan route hasil generate tanpa menyentuh output repository.
2. **Pengujian API hasil generate**: create/read/query/update, validasi field, ID terenkode, proyeksi, izin, dan konflik `version` basi.
3. **Pengujian hook**: trim/lowercase username sebelum pemeriksaan unik, penolakan create dan rename duplikat, perlindungan duplikat konkuren, serta penolakan DELETE.
4. **Pengujian siklus hidup**: prasyarat aktivasi, deaktivasi diri sendiri, pengulangan idempoten, transisi basi, commit state sebelum permintaan pencabutan, kegagalan downstream tetap fail-closed, dan tidak ada pemulihan artefak.
5. **Pengujian kontrak audit**: tepat satu event berhasil per create/pembaruan profil/transisi nyata; himpunan nama field tepat; tanpa nilai atau rahasia; operasi ditolak dan no-op tidak membuat event transisi.
6. **Pengujian web**: perilaku schema/form/list hasil generate, komposisi filter pencarian/status, ketiadaan aksi hapus, izin/state aksi siklus hidup, tampilan metadata/avatar, dan state dependensi ringkasan role.
7. **Pengujian integrasi/keamanan lintas-plugin**: CRUD/siklus hidup tanpa otorisasi, penolakan akun nonaktif/kedaluwarsa saat login dan akses berlanjut, pencabutan dalam satu menit, serta reaktivasi tidak menghidupkan artefak lama.
8. **Pemeriksaan regresi di luar ruang lingkup**: tidak ada field password, credential, failed-login, lockout, atau last-login dalam JSON, migrasi, DTO, API, maupun UI.

## Command Generator dan Validasi yang Terverifikasi

Rangkaian command berikut tersedia di `gasi.one.cli` dan telah diverifikasi
dengan menjalankan varian `validate`/`plan` terhadap contoh repository:

```bash
cd ../gasi.one.cli

node ./bin/gasi-one.js plugin validate -f definitions/identity-access/plugin.json
node ./bin/gasi-one.js resource validate -f definitions/identity-access/resources.json
node ./bin/gasi-one.js contract validate -f definitions/identity-access/iam-user-contract.json

node ./bin/gasi-one.js plugin plan -f definitions/identity-access/plugin.json -o ../gasi.one.api/plugins --target api
node ./bin/gasi-one.js plugin plan -f definitions/identity-access/plugin.json -o ../gasi.one.web/plugins --target web
node ./bin/gasi-one.js resource plan -f definitions/identity-access/resources.json -o ../gasi.one.api/plugins/identity-access-plugin --target api
node ./bin/gasi-one.js resource plan -f definitions/identity-access/resources.json -o ../gasi.one.web/plugins/identity-access-plugin --target web
node ./bin/gasi-one.js contract plan -f definitions/identity-access/iam-user-contract.json -o ../gasi.one.api/contracts --target api
```

Setelah meninjau setiap plan, ganti hanya aksi `plan` dengan `sync`. Tidak ada
target gabungan `api,web`; API dan web harus dijalankan secara terpisah. `clean`
tersedia tetapi destruktif dan bukan bagian dari alur implementasi normal.

Verifikasi repository setelah generate/implementasi khusus menggunakan command
yang memang tersedia:

```bash
cd ../gasi.one.cli && npm test
cd ../gasi.one.api/plugins/identity-access-plugin && mvn test
cd ../gasi.one.api/plugins/identity-access-plugin && mvn clean verify
cd ../gasi.one.web && npm run build -w plugins/identity-access-plugin
cd ../gasi.one.web && npm run build
cd ../gasi.one.web && npm run lint
```

Plugin web hasil generate saat ini mendefinisikan `build`, tetapi tidak memiliki
script `test` atau `lint` lokal plugin. Karena itu, rencana tidak boleh
mengklaim command tersebut tersedia sebelum ditambahkan.

## Pelacakan Kompleksitas

Tidak ada pelanggaran konstitusi. Komponen khusus diperlukan karena celah
capability, bukan sebagai implementasi CRUD alternatif:

| Komponen khusus | Mengapa hook/konfigurasi tidak mencukupi | Dijaga minimal dengan |
|---|---|---|
| Service/controller aksi siklus hidup | Hook CRUD tidak dapat menambahkan route aksi, menggunakan no-op idempoten khusus aksi, atau menghindari save/kenaikan version | Menggunakan kembali model, port repository, konvensi mapping DTO, izin, ID, dan response envelope hasil generate |
| Metadata bootstrap kustomisasi CLI | Registry ResourceCustom yang tersedia dapat digunakan, tetapi seluruh file bootstrap/route plugin yang dapat dijangkau adalah hasil generate dan akan ditimpa | Menambahkan satu field import deklaratif dan mempertahankan seluruh perilaku UI dalam registry yang tersedia |
| Antarmuka capability stabil | Import implementasi lintas-plugin dilarang dan generator kontrak sengaja hanya membuat kerangka | Hanya antarmuka/value object; implementasi tetap di plugin pemilik |
