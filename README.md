# API Dokumentasi: Single Sign-On (SSO) Gebyok Kudus

Dokumentasi ini menjelaskan cara mengakses API Single Sign-On (SSO) Gebyok Kudus yang menggunakan protokol OAuth 2.0 untuk autentikasi dan otorisasi terpusat. API ini menyediakan langkah-langkah untuk mendapatkan authorization code, menukarnya dengan access token, serta menggunakan token tersebut untuk mengambil data profil pengguna.

## Daftar Endpoint
1. Mendapatkan Authorization Code
2. Mendapatkan Access Token
3. Mengakses Data Pengguna

---

## Endpoint 1: Mendapatkan Authorization Code

- **URL**: `https://gebyokkudus.kuduskab.go.id/oauth/authorize`
- **Method**: `GET`
- **Deskripsi**: Endpoint ini digunakan untuk meminta kode otorisasi yang akan ditukar dengan access token. Kode ini hanya dapat digunakan sekali dan memiliki masa berlaku terbatas.

### Parameter

| Parameter     | Wajib | Deskripsi                                                                                       |
|---------------|-------|-------------------------------------------------------------------------------------------------|
| `client_id`   | Ya    | ID unik untuk aplikasi client yang didaftarkan di SSO server.                                   |
| `redirect_uri`| Ya    | URL callback yang akan menerima kode otorisasi setelah pengguna memberikan izin.                 |
| `response_type` | Ya  | Jenis respons yang diminta. Untuk authorization code, gunakan nilai `code`.                    |
| `scope`       | Ya    | Izin yang diminta oleh aplikasi client, misalnya `view-user`.                                    |
| `state`       | Ya    | Nilai unik untuk mencegah serangan CSRF, dihasilkan secara acak oleh client.                    |

### Contoh Permintaan

```bash
GET https://gebyokkudus.kuduskab.go.id/oauth/authorize?client_id=ISI_DENGAN_CLIENT_ID_GEBYOK_KUDUS&redirect_uri=ISI_DENGAN_REDIRECT_URI_CLIENT&response_type=code&scope=view-user&state=ISI_DENGAN_STATE_UNIK
```

---

## Endpoint 2: Mendapatkan Access Token

- **URL**: `https://gebyokkudus.kuduskab.go.id/oauth/token`
- **Method**: `POST`
- **Deskripsi**: Endpoint ini digunakan untuk menukar authorization code dengan access token, yang kemudian digunakan untuk mengakses API SSO lainnya.

### Parameter

| Parameter      | Wajib | Deskripsi                                                                                |
|----------------|-------|------------------------------------------------------------------------------------------|
| `grant_type`   | Ya    | Jenis grant. Untuk authorization code, gunakan `authorization_code`.                     |
| `client_id`    | Ya    | ID unik untuk aplikasi client yang terdaftar.                                            |
| `client_secret`| Ya    | Kunci rahasia yang didapat saat mendaftarkan aplikasi client.                            |
| `redirect_uri` | Ya    | URL callback yang sama dengan yang digunakan pada permintaan kode.                       |
| `code`         | Ya    | Kode otorisasi yang didapatkan dari endpoint authorize.                                  |

### Headers

| Header            | Wajib | Deskripsi                                  |
|-------------------|-------|--------------------------------------------|
| `Content-Type`    | Ya    | `application/x-www-form-urlencoded`        |

### Contoh Permintaan

```bash
curl -X POST "https://gebyokkudus.kuduskab.go.id/oauth/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=authorization_code" \
-d "client_id=ISI_DENGAN_CLIENT_ID_GEBYOK_KUDUS" \
-d "client_secret=ISI_DENGAN_CLIENT_SECRET_GEBYOK_KUDUS" \
-d "redirect_uri=ISI_DENGAN_REDIRECT_URI_CLIENT" \
-d "code=ISI_DENGAN_AUTHORIZATION_CODE"
```

---

## Endpoint 3: Mengakses Data Pengguna

- **URL**: `https://gebyokkudus.kuduskab.go.id/api/user-sso`
- **Method**: `GET`
- **Deskripsi**: Setelah mendapatkan access token, gunakan endpoint ini untuk mendapatkan informasi profil pengguna yang terautentikasi.

### Headers

| Header          | Wajib | Deskripsi                                |
|-----------------|-------|------------------------------------------|
| `Authorization` | Ya    | Bearer diikuti access token yang valid.  |

### Contoh Respons

```json
{
    "id": 41,
    "name": "Nama Bendahara",
    "email": "test@gmail.com",
    "nip": "-",
    "email_pp": [
        "ppk1@gmail.com"
    ],
    "no_hp": "087705572167",
    "kd_satker": "75642",
    "nama_satker": "DINAS KEPENDUDUKAN DAN PENCATATAN SIPIL",
    "email_verified_at": null,
    "created_at": "2024-08-22T05:48:01.000000Z",
    "updated_at": "2024-08-22T05:48:01.000000Z",
    "role": "bendahara"
}
```

### Penjelasan Struktur Respons

| Field               | Tipe Data       | Deskripsi                                                 |
|---------------------|-----------------|-----------------------------------------------------------|
| `id`                | Integer         | ID unik pengguna dalam sistem.                            |
| `name`              | String          | Nama lengkap pengguna.                                    |
| `email`             | String          | Alamat email utama pengguna.                              |
| `nip`               | String          | Nomor Induk Pegawai pengguna, atau tanda `"-"` jika tidak tersedia. |
| `email_pp`          | Array of String | Daftar alamat email tambahan yang terkait dengan pengguna.|
| `no_hp`             | String          | Nomor telepon pengguna.                                   |
| `kd_satker`         | String          | Kode satuan kerja pengguna.                               |
| `nama_satker`       | String          | Nama satuan kerja pengguna.                               |
| `email_verified_at` | String/Null     | Tanggal dan waktu verifikasi email, `null` jika belum diverifikasi. |
| `created_at`        | String          | Tanggal dan waktu pembuatan data pengguna dalam format ISO 8601. |
| `updated_at`        | String          | Tanggal dan waktu terakhir data pengguna diperbarui dalam format ISO 8601. |
| `role`              | String          | Peran atau jabatan pengguna dalam sistem.                 |

---

## Panduan Penggunaan

1. **Dapatkan Authorization Code** melalui endpoint `/oauth/authorize` dengan parameter yang sesuai.
2. **Tukar Authorization Code** dengan Access Token melalui endpoint `/oauth/token`.
3. **Gunakan Access Token** untuk mengakses data pengguna melalui endpoint `/api/user-sso`.

Dengan API ini, aplikasi dapat menggunakan autentikasi SSO yang aman untuk mengelola akses pengguna.
