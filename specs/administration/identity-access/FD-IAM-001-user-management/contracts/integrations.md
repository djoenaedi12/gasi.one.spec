# Kontrak Integrasi

## Aturan batas

Manajemen Pengguna memiliki fakta identitas akun dan state siklus hidup. Fitur
ini tidak memiliki credential autentikasi, perhitungan permission, penyimpanan
session/token, penyimpanan PAT, atau persistensi audit. Pemanggilan lintas-plugin
menggunakan modul kontrak stabil ditambah `CapabilityRegistry`; consumer tidak
mengimpor entity, repository, implementasi service, atau JAR implementasi milik
plugin lain.

## Capability kelayakan pengguna (milik FD-IAM-001)

Tujuan: memungkinkan provider Authentication, Session/Token, dan PAT mengambil
keputusan fail-closed dari state akun yang otoritatif.

Kontrak konseptual:

```text
capability id: iam.user-eligibility
input: userAccountId stabil yang terenkode
output:
  userAccountId
  status: ACTIVE | INACTIVE
  authorizedUntil: instant | null
  eligible: boolean
  evaluatedAt: instant
  reason: ACTIVE | INACTIVE | AUTHORIZATION_EXPIRED | ACCOUNT_NOT_FOUND
```

Aturan:

- Akun yang tidak dikenal tidak layak.
- INACTIVE tidak layak apa pun nilai authorized-until.
- ACTIVE hanya layak ketika authorized-until null atau secara ketat setelah waktu evaluasi.
- Capability tidak mengekspos password, faktor autentikasi, token, role, atau rahasia.
- Provider sebaiknya mendukung evaluasi batch ketika throughput validasi session membuat lookup tunggal tidak efisien; ini adalah evolusi antarmuka, bukan alasan untuk menyimpan cache kelayakan melampaui kebijakan keamanan.

Kerangka modul kontrak dapat dibuat dengan generator kontrak CLI yang tersedia.
Definisi interface/record ditulis manual karena dokumentasi CLI secara eksplisit
menetapkan generate kontrak hanya sebagai scaffolding.

## Identitas aktor saat ini (dependensi: FD-IAM-004)

Perlindungan nonaktifkan diri sendiri memerlukan ID akun stabil pada
principal/context terautentikasi. Username saja tidak mencukupi karena mutable.

Perilaku wajib:

- mengembalikan ID Akun Pengguna stabil dan terenkode untuk aktor terautentikasi;
- tidak mengembalikan aktor untuk pemanggilan anonim/sistem, dengan kebijakan sistem eksplisit;
- menjaga credential dan token di luar object identitas yang dikembalikan;
- endpoint siklus hidup membandingkan ID aktor dengan ID target sebelum perubahan state apa pun.

Sampai kontrak ini tersedia, acceptance test nonaktifkan diri sendiri tidak
dapat lulus dan endpoint siklus hidup harus fail-closed, bukan fallback ke
username yang mutable.

## Kelayakan autentikasi (dependensi: FD-IAM-004)

Authentication harus memanggil `iam.user-eligibility` sebelum menerbitkan
session atau token baru. ACTIVE saja tidak cukup; authorized-until juga harus
valid.

Manajemen Pengguna tidak mengimplementasikan:

- verifikasi atau penyimpanan password;
- penghitungan failed-login atau account lockout;
- pembaruan last-login;
- MFA/OTP/recovery;
- kebijakan AppClient/channel login.

## Pencabutan artefak (dependensi: FD-IAM-006 dan FD-IAM-007)

Kontrak command konseptual:

```text
capability id: auth.revoke-user-artifacts
input:
  userAccountId
  reason: ACCOUNT_DEACTIVATED | AUTHORIZATION_EXPIRED
  requestedAt
  idempotencyKey
output:
  accepted: boolean
  requestId
  confirmation: CONFIRMED | PENDING
```

Perilaku wajib:

- mencabut/menolak session aktif, refresh token, state trusted-device, bearer token dalam cakupan FD-IAM-006, dan PAT dalam cakupan FD-IAM-007;
- request idempoten berdasarkan identitas transisi/event akun;
- setelah diinvalidasi, artefak tidak pernah valid kembali saat reaktivasi;
- provider memiliki retry durable dan konfirmasi;
- pemeriksaan akses secara independen memanggil kelayakan pengguna sehingga pencabutan PENDING/tidak tersedia tidak pernah membuat akun INACTIVE/kedaluwarsa dapat terus mengakses.

Urutan:

1. Validasi aktor, target, version, dan transisi.
2. Simpan INACTIVE dan commit.
3. Pancarkan identitas audit/event siklus hidup.
4. Setelah commit, kirim pencabutan dengan idempotency key tersebut.
5. Jika tidak tersedia, akun tetap INACTIVE; provider melakukan retry dan monitoring mengekspos kondisi pending.

Untuk kedaluwarsa berbasis waktu, provider Authentication/Session/PAT mendeteksi
hasil tidak layak saat login/akses berlanjut dan memulai kontrak pencabutan yang
sama dengan `AUTHORIZATION_EXPIRED`.

## Ringkasan role yang ditetapkan (dependensi: FD-IAM-002)

UI detail User memerlukan daftar read-only dan non-rahasia untuk semua role
yang ditetapkan. Kontrak dependensi mengembalikan nol atau lebih ringkasan role
untuk ID Akun Pengguna stabil. Kontrak tersebut tidak boleh menghitung atau
menyiratkan satu role terpilih/efektif di dalam FD-IAM-001.

Field ringkasan minimum:

```text
roleId, code, displayName, assignmentStatus
```

Definisi role, pemilihan beberapa role aktif yang kompatibel, permission
efektif, dan segregation-of-duties tetap sepenuhnya berada di FD-IAM-002.

## Event audit (dependensi: capability audit lintas-domain)

Event wajib:

```text
actorId: ID akun stabil yang terenkode
occurredAt: instant
action: CREATE | UPDATE | ACTIVATE | DEACTIVATE | REACTIVATE
resourceType: UserAccount
resourceId: ID akun target yang terenkode
changedFields: set<string>
```

Batasan keamanan:

- hanya sertakan nama field;
- jangan pernah menyertakan nilai before/after;
- jangan pernah menyertakan password, credential, token, rahasia MFA, atau data recovery;
- operasi yang ditolak dan no-op siklus hidup idempoten tidak membuat event transisi berhasil;
- tepat satu event disimpan untuk setiap operasi yang berhasil.

Status framework saat ini:

- `@AuditResource` dapat menandai service CRUD hasil generate;
- `@AuditAction` dapat menandai method siklus hidup;
- `AuditLogExtension` dapat menyesuaikan description;
- `AuditLogEntry` saat ini tidak dapat membawa `changedFields`.

Karena itu, pemilik audit harus mengembangkan kontrak/interceptor stabilnya
sebelum FR-021 dianggap selesai. Manajemen Pengguna menyediakan nama field yang
benar-benar berubah dengan membandingkan state akun kanonis sebelum/sesudah;
audit memiliki penyimpanan durable dan API pencarian/review.

## Kebijakan kegagalan dan startup

| Dependensi | Perilaku wajib saat tidak ada/tidak tersedia |
|---|---|
| Aktor saat ini/permission autentikasi | Tolak operasi terlindungi; jangan ungkapkan data akun |
| Provider kelayakan pengguna | Auth/session/PAT fail-closed |
| Provider pencabutan | Akun tetap tidak layak; request pending/di-retry; akses tetap ditolak |
| Writer/interceptor audit | Gate kesiapan produksi gagal; jangan diam-diam mengklaim operasi telah diaudit |
| Ringkasan role | Detail akun inti tetap dapat digunakan dengan state unavailable/empty eksplisit sesuai kebijakan rollout FD-IAM-002 |

Nama artefak Maven konkret dan ID plugin PF4J diterbitkan oleh FD pemilik.
Setelah diterbitkan, `plugin.json` mencatat dependensi kontrak dan metadata urutan
load; class implementasi tidak pernah dirujuk.
