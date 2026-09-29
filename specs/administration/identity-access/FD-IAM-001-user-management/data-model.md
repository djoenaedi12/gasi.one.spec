# Model Data: Manajemen Pengguna

## UserAccount

Catatan identitas administratif milik FD-IAM-001. Entitas ini tidak memuat data
credential, telemetri autentikasi, session, token, penetapan role, employee,
atau person.

### Field

| Field | Persistensi | Kontrak publik | Wajib | Aturan/catatan |
|---|---|---|---|---|
| `id` | Primary key `BIGINT` | `string` terenkode | Sistem | Identitas stabil berbasis TSID; tidak pernah berubah ketika username berubah |
| `username` | `VARCHAR(100) UNIQUE` | `string` | Ya | Hapus whitespace Unicode luar, lowercase dengan `Locale.ROOT`, lalu validasi dan simpan; pola `[a-z0-9][a-z0-9._-]{2,99}` |
| `displayName` | `VARCHAR(150)` | `string` | Ya | Nilai tampilan non-blank yang di-trim; duplikat diperbolehkan |
| `email` | `VARCHAR(254)` | email `string` | Ya | Di-trim; sintaks email valid; tidak unik |
| `phone` | `VARCHAR(32)` nullable | `string | null` | Tidak | Nilai kontak yang di-trim; blank dinormalisasi menjadi null; dapat dicari |
| `avatar` | `VARCHAR(2048)` nullable | `string | null` | Tidak | Hanya URI atau referensi penyimpanan opaque; unggah/penyimpanan biner merupakan capability terpisah |
| `status` | `VARCHAR(50)` | `ACTIVE | INACTIVE` | Ya | Dipilih saat create; hanya diubah melalui aksi siklus hidup |
| `authorizedUntil` | `TIMESTAMP(6)` nullable | Instant RFC 3339 atau null | Tidak | Null berarti tidak terbatas; nilai kedaluwarsa membuat akun tidak layak sekalipun ACTIVE |
| `createdAt` | `TIMESTAMP(6)` | Instant RFC 3339 | Sistem | Metadata hasil generate yang diwarisi, immutable |
| `updatedAt` | `TIMESTAMP(6)` | Instant RFC 3339 | Sistem | Metadata hasil generate yang diwarisi |
| `createdBy` | `VARCHAR(50)` nullable | `string | null` | Sistem | Aktor disediakan melalui auditing Spring Security/JPA |
| `updatedBy` | `VARCHAR(50)` nullable | `string | null` | Sistem | Aktor disediakan melalui auditing Spring Security/JPA |
| `version` | `INT NOT NULL DEFAULT 0` | `integer` | Mutasi | Version optimistic lock yang diwarisi; wajib dalam request update/siklus hidup |
| `lifecycleStatus` | `TINYINT UNSIGNED` | Bukan bagian API akun | Framework | Field teknis yang diwarisi; berbeda dari `status` bisnis |

### Urutan kanonisasi

1. Ubah phone/avatar opsional yang blank menjadi null.
2. Hapus whitespace awal/akhir pada username, nama tampilan, email, dan telepon.
3. Ubah username menjadi lowercase dengan aturan independen locale.
4. Terapkan validasi generator/Jakarta/Zod pada request kanonis.
5. Jalankan validasi keunikan hasil generate.
6. Andalkan constraint unik database untuk perlindungan race konkuren.

Kapitalisasi email tidak digunakan sebagai identitas dan tidak dinormalisasi
untuk keunikan.

### Invarian

- ID bersifat stabil dan tidak dapat diberikan atau diubah administrator.
- Username unik secara global dalam bentuk kanonis.
- Nama tampilan dapat sama antar-akun.
- Status selalu ACTIVE atau INACTIVE.
- PUT profil standar tidak dapat mengubah status.
- Akun ACTIVE hanya layak selama authorized-until null atau berada di masa depan.
- Akun INACTIVE tidak pernah layak, apa pun nilai authorized-until.
- Resource tidak dapat dihapus permanen melalui FD-IAM-001.
- Setiap operasi mutasi yang berhasil menggunakan optimistic locking.

## State dan kelayakan

```text
                 activate/reactivate
        ┌──────────────────────────────────┐
        │                                  ▼
    INACTIVE                         ACTIVE + belum kedaluwarsa
        ▲                                  │
        └──────────── deactivate ──────────┘
                                           │
                                           │ waktu melewati authorizedUntil
                                           ▼
                              ACTIVE + kedaluwarsa (tidak layak)
```

`ACTIVE + kedaluwarsa` bukan status tersimpan ketiga, melainkan kondisi tidak
layak yang diturunkan. Reaktivasi mewajibkan authorized-until bernilai null atau
diubah ke masa depan melalui pembaruan profil sebelum aktivasi.

### Aturan transisi

| Command | State saat ini | Prasyarat | Hasil | Efek samping |
|---|---|---|---|---|
| Activate | INACTIVE | Data wajib valid; authorized-until null/masa depan; version terbaru | ACTIVE, version bertambah | Satu event REACTIVATE; tidak memulihkan artefak lama |
| Activate | ACTIVE | Tidak ada selain otorisasi | Tidak ada perubahan state/version | Tidak ada event transisi |
| Deactivate | ACTIVE | ID stabil aktor berbeda dari target; version terbaru | INACTIVE, version bertambah | Satu event DEACTIVATE; request pencabutan setelah commit |
| Deactivate | INACTIVE | Tidak ada selain otorisasi | Tidak ada perubahan state/version | Tidak ada event atau pengulangan request pencabutan |
| Pembaruan profil | Keduanya | Version terbaru; validasi hasil generate/khusus lulus | Field profil mutable diperbarui | Satu event UPDATE yang memuat nama field yang benar-benar berubah |
| Delete | Keduanya | Tidak pernah diizinkan | Ditolak, tidak berubah | Tidak ada event delete/transisi |

Pembuatan dapat memilih ACTIVE atau INACTIVE. Pembuatan ACTIVE dengan
authorized-until yang sudah kedaluwarsa ditolak karena tidak valid pada waktu
pembuatan.

## Desain migrasi

Generator resource API membuat migrasi pertama di:

```text
plugins/identity-access-plugin/src/main/resources/db/migration/iam/
V<timestamp>__create_iam_user_accounts.sql
```

Bentuk logis yang diharapkan (generator mengendalikan format dan nama constraint
secara exact):

```sql
CREATE TABLE iam_user_accounts (
    id BIGINT NOT NULL PRIMARY KEY,
    created_by VARCHAR(50),
    created_at TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP(6),
    updated_by VARCHAR(50),
    updated_at TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
    version INT NOT NULL DEFAULT 0,
    lifecycle_status TINYINT UNSIGNED,
    username VARCHAR(100) NOT NULL UNIQUE,
    display_name VARCHAR(150) NOT NULL,
    email VARCHAR(254) NOT NULL,
    phone VARCHAR(32),
    avatar VARCHAR(2048),
    status VARCHAR(50) NOT NULL,
    authorized_until TIMESTAMP(6)
);
```

Gate peninjauan sebelum menerapkan SQL hasil generate:

- enum status disimpan sebagai string, bukan ordinal;
- kolom username dan constraint unik tersedia;
- tidak ada kolom password/credential/login-state;
- tidak ada cascade hard-delete atau foreign key Employee/Person;
- timestamp dan SQL enum kompatibel dengan MariaDB;
- migrasi immutable setelah deployment.

Perubahan definisi berikutnya dihasilkan sebagai migrasi alter baru dari
`.gasi-one/manifest.json`. Perubahan keunikan, relasi, dan tipe harus mendapat
peninjauan SQL manual karena CLI memperingatkan kasus tersebut mungkin
memerlukan logika drop/recreate constraint secara eksplisit.

## UserAccountAuditEvent (entitas dependensi logis)

Ini adalah payload kontrak, bukan tabel milik Manajemen Pengguna.

| Field | Tipe | Aturan |
|---|---|---|
| `actorId` | string terenkode | Referensi stabil aktor saat ini dari Authentication |
| `occurredAt` | instant | Clock subsistem audit |
| `action` | enum/string | CREATE, UPDATE, ACTIVATE, DEACTIVATE, REACTIVATE |
| `userAccountId` | string terenkode | Referensi akun target yang stabil |
| `changedFields` | set string | Hanya nama field; kosong hanya jika kontrak aksi mengizinkan |
| `beforeValues` | tidak ada | Tidak boleh pernah dipancarkan |
| `afterValues` | tidak ada | Tidak boleh pernah dipancarkan |

Penyimpanan, indexing, pencarian, dan UI reviewer audit dimiliki capability
audit lintas-domain.

## Relasi eksternal

| Relasi | Kardinalitas | Pemilik | Representasi di sini |
|---|---|---|---|
| Role yang ditetapkan | User 1 → 0..* assignment | FD-IAM-002 | Ringkasan eksternal read-only; tanpa FK/koleksi tabel User |
| Kebijakan/klien Authentication | Banyak kebijakan/klien dapat mengevaluasi user | FD-IAM-004 | Capability kelayakan berdasarkan ID user stabil |
| Session/token/trusted device | User 1 → 0..* artefak | FD-IAM-006 | Pemanggilan capability pencabutan; tanpa tabel artefak lokal |
| Personal access token | User 1 → 0..* PAT | FD-IAM-007 | Pemeriksaan status/kedaluwarsa dan pencabutan dimiliki provider |
| Event audit | User 1 → 0..* event | Capability Audit | Hanya kontrak event; tanpa tabel audit lokal |

Credential password, penghitung failed-login, timestamp/state lockout, dan
timestamp last login sengaja tidak ada dari setiap entitas dan relasi.
