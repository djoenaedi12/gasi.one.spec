# Kontrak Definisi CRUD

## Kepemilikan dan lokasi

Source of truth CRUD hasil generate direncanakan berada di:

```text
../gasi.one.cli/definitions/identity-access/resources.json
```

Descriptor plugin direncanakan berada di sebelahnya sebagai `plugin.json`.
`resources.json` digunakan tanpa perubahan untuk generate API dan web. File
Java, SQL, TypeScript, dan TSX yang dihasilkan pada runtime merupakan artefak
turunan dan tidak boleh diubah.

## Descriptor plugin yang direncanakan

```json
{
  "name": "identity-access",
  "code": "iam",
  "displayName": "Identitas & Akses",
  "version": "1.0.0",
  "description": "Plugin administrasi identitas dan akses",
  "startupPolicy": "mandatory",
  "requiresPlatform": ">=1.0.0 & <2.0.0",
  "dependsOn": [],
  "contractDependencies": []
}
```

Koordinat dependensi dan ID PF4J hanya ditambahkan setelah pemilik
Authentication, Role, Session/Token, PAT, dan Audit menerbitkan kontraknya.
Nilai tersebut tidak boleh ditebak atau diganti dengan dependensi implementasi.
Provider keamanan dan audit yang wajib merupakan gate penerimaan deployment,
meskipun descriptor dijaga tetap valid secara mandiri selama generate awal.

## Bentuk resource UserAccount yang direncanakan

Ini adalah kontrak desain, bukan file implementasi yang dibuat oleh fase
perencanaan. Setiap key berikut didukung CLI saat ini kecuali
`ui.customizationModule`, yaitu peningkatan untuk celah CLI yang secara
eksplisit direncanakan setelah contoh.

```json
{
  "resources": [
    {
      "name": "UserAccount",
      "pluginName": "iam",
      "package": "gasi.one.plugins.iam.useraccount",
      "mode": "crud",
      "table": "iam_user_accounts",
      "endpoint": "/user-accounts",
      "lookup": true,
      "reference": {
        "expose": true,
        "labelFields": ["username", "displayName"]
      },
      "ui": {
        "resource": "standard",
        "customizationModule": "./custom/user-account.custom",
        "breadcrumb": {
          "labelFields": ["username", "displayName"]
        },
        "confirm": {
          "labelFields": ["username", "displayName"]
        },
        "list": {
          "pageSize": 20,
          "searchFields": ["username", "displayName", "email", "phone"],
          "defaultSort": [
            { "field": "username", "direction": "ASC" }
          ]
        },
        "lookup": {
          "pageSize": 10,
          "displayFields": ["username", "displayName", "email", "status"],
          "searchFields": ["username", "displayName", "email"],
          "labelFields": ["username", "displayName"],
          "defaultSort": [
            { "field": "username", "direction": "ASC" }
          ]
        },
        "create": {
          "sections": [
            {
              "id": "identity",
              "title": "Identitas",
              "rows": [
                { "columns": 2, "fields": ["username", "displayName"] },
                { "columns": 2, "fields": ["email", "phone"] }
              ]
            },
            {
              "id": "access",
              "title": "Kelayakan akses",
              "rows": [
                { "columns": 2, "fields": ["status", "authorizedUntil"] }
              ]
            },
            {
              "id": "presentation",
              "title": "Tampilan",
              "rows": [
                { "columns": 1, "fields": ["avatar"] }
              ]
            }
          ]
        },
        "edit": {
          "sections": [
            {
              "id": "identity",
              "title": "Identitas",
              "rows": [
                { "columns": 2, "fields": ["username", "displayName"] },
                { "columns": 2, "fields": ["email", "phone"] }
              ]
            },
            {
              "id": "access",
              "title": "Kelayakan akses",
              "rows": [
                { "columns": 1, "fields": ["authorizedUntil"] }
              ]
            },
            {
              "id": "presentation",
              "title": "Tampilan",
              "rows": [
                { "columns": 1, "fields": ["avatar"] }
              ]
            }
          ]
        },
        "detail": {
          "sections": [
            {
              "id": "identity",
              "title": "Identitas",
              "rows": [
                { "columns": 2, "fields": ["username", "displayName"] },
                { "columns": 2, "fields": ["email", "phone"] }
              ]
            },
            {
              "id": "access",
              "title": "Kelayakan akses",
              "rows": [
                { "columns": 2, "fields": ["status", "authorizedUntil"] }
              ]
            },
            {
              "id": "presentation",
              "title": "Tampilan",
              "rows": [
                { "columns": 1, "fields": ["avatar"] }
              ]
            }
          ]
        }
      },
      "fields": [
        {
          "name": "username",
          "type": "string",
          "length": 100,
          "required": true,
          "unique": true,
          "projection": true,
          "validation": {
            "minLength": 3,
            "maxLength": 100,
            "pattern": "^[a-z0-9][a-z0-9._-]{2,99}$"
          }
        },
        {
          "name": "displayName",
          "type": "string",
          "length": 150,
          "required": true,
          "projection": true,
          "validation": {
            "minLength": 1,
            "maxLength": 150
          }
        },
        {
          "name": "email",
          "type": "string",
          "length": 254,
          "required": true,
          "projection": true,
          "validation": {
            "email": true,
            "maxLength": 254
          }
        },
        {
          "name": "phone",
          "type": "string",
          "length": 32,
          "projection": false,
          "validation": {
            "maxLength": 32,
            "pattern": "^\\+?[0-9 ()-]{7,32}$"
          }
        },
        {
          "name": "avatar",
          "type": "string",
          "length": 2048,
          "dto": { "summary": false },
          "validation": { "maxLength": 2048 }
        },
        {
          "name": "status",
          "type": "enum",
          "required": true,
          "projection": true,
          "dto": { "update": false },
          "enum": {
            "name": "AccountStatus",
            "type": "string",
            "values": ["ACTIVE", "INACTIVE"]
          }
        },
        {
          "name": "authorizedUntil",
          "type": "instant",
          "projection": true
        }
      ],
      "i18n": {
        "id": {
          "field": {
            "username": "Username",
            "displayName": "Nama tampilan",
            "email": "Email",
            "phone": "Telepon",
            "avatar": "Avatar",
            "status": "Status",
            "authorizedUntil": "Diotorisasi sampai"
          }
        }
      }
    }
  ]
}
```

## Dukungan CLI saat ini dibanding celah yang direncanakan

| Kebutuhan definisi | Dukungan repository saat ini |
|---|---|
| `mode: crud`, nama/package/tabel/endpoint | Didukung |
| string/instant/enum yang disimpan sebagai string | Didukung |
| required, unique, email, length, pattern | Didukung |
| Flag DTO create/update/summary/detail | Didukung |
| projection, sortable, search fields, default sort | Didukung |
| Section dan row create/edit/detail | Didukung |
| Provider lookup/reference | Didukung |
| Generate service hook untuk validasi unik | Didukung |
| Menonaktifkan DELETE per operasi | Belum didukung; gunakan ekstensi backend/UI |
| Bootstrap modul khusus deklaratif | Belum didukung; peningkatan CLI direncanakan |
| Endpoint aksi siklus hidup | Bukan bagian definisi CRUD; implementasi aksi khusus yang sempit |

Peningkatan CLI harus menormalisasi dan memvalidasi
`ui.customizationModule` sebagai path modul relatif terhadap root plugin yang
aman, lalu menghasilkan import di `src/routes.ts`. Key yang tidak dikenal saat
ini dibuang oleh normalisasi, sehingga properti ini tidak boleh dianggap
operasional sampai pengujian regresi CLI membuktikan import telah dihasilkan.

## Kepemilikan file hasil generate

- `.gasi-one/manifest.json` mencatat path API/web hasil generate.
- File Java dan React hasil generate ditimpa ketika kontennya berbeda.
- Migrasi create dipertahankan; diff schema berikutnya menghasilkan file alter.
- Properti i18n di-merge berdasarkan key.
- `src/custom/user-account.custom.tsx` dan backend `useraccountcustom/*` sengaja tidak ada dalam manifest generator.

## Urutan command wajib

Dari `gasi.one.cli`:

```bash
node ./bin/gasi-one.js plugin validate -f definitions/identity-access/plugin.json
node ./bin/gasi-one.js resource validate -f definitions/identity-access/resources.json

node ./bin/gasi-one.js plugin plan -f definitions/identity-access/plugin.json -o ../gasi.one.api/plugins --target api
node ./bin/gasi-one.js plugin plan -f definitions/identity-access/plugin.json -o ../gasi.one.web/plugins --target web
node ./bin/gasi-one.js resource plan -f definitions/identity-access/resources.json -o ../gasi.one.api/plugins/identity-access-plugin --target api
node ./bin/gasi-one.js resource plan -f definitions/identity-access/resources.json -o ../gasi.one.web/plugins/identity-access-plugin --target web
```

Command yang sama baru dijalankan dengan `sync` setelah plan ditinjau.
