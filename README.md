### Dokumentasi API Single Sign-On (SSO) Gebyok Kudus

API Single Sign-On (SSO) Gebyok Kudus memfasilitasi proses autentikasi dan otorisasi terpusat melalui protokol OAuth 2.0. API ini memungkinkan aplikasi client untuk mengakses data profil pengguna setelah pengguna memberikan izin melalui mekanisme OAuth. Berikut ini adalah panduan lengkap untuk mengakses dan menggunakan API SSO Gebyok Kudus.

---

### Daftar Endpoint

1. **Mendapatkan Authorization Code**
2. **Mendapatkan Access Token**
3. **Mengakses Data Pengguna**

---

### Endpoint 1: Mendapatkan Authorization Code

**URL:** `https://gebyokkudus.kuduskab.go.id/oauth/authorize`  
**Method:** `GET`  
**Deskripsi:** Endpoint ini digunakan untuk meminta authorization code yang akan ditukarkan dengan access token. Authorization code ini hanya berlaku sekali dan memiliki masa berlaku terbatas.

**Parameter:**

| Nama          | Wajib | Deskripsi                                                                 |
|---------------|-------|---------------------------------------------------------------------------|
| `client_id`   | Ya    | ID unik untuk aplikasi client yang telah didaftarkan di server SSO.      |
| `redirect_uri`| Ya    | URL callback yang akan menerima kode otorisasi setelah pengguna login.   |
| `response_type` | Ya | Jenis respons yang diminta. Untuk authorization code, gunakan `code`.     |
| `scope`       | Ya    | Izin yang diminta oleh aplikasi client, seperti `view-user`.            |
| `state`       | Ya    | Nilai unik yang dihasilkan oleh client untuk mencegah serangan CSRF.    |

**Contoh Permintaan:**

```http
GET https://gebyokkudus.kuduskab.go.id/oauth/authorize?client_id=ISI_DENGAN_CLIENT_ID_GEBYOK_KUDUS&redirect_uri=ISI_DENGAN_REDIRECT_URI_CLIENT&response_type=code&scope=view-user&state=ISI_DENGAN_STATE_UNIK
```

**Penjelasan Proses:**
1. Aplikasi client mengarahkan pengguna ke URL `authorize`.
2. Pengguna diminta untuk login dan memberikan izin kepada aplikasi client.
3. Setelah izin diberikan, server SSO mengarahkan pengguna kembali ke `redirect_uri` dengan `authorization code` yang dapat digunakan untuk mendapatkan `access token`.

---

### Endpoint 2: Mendapatkan Access Token

**URL:** `https://gebyokkudus.kuduskab.go.id/oauth/token`  
**Method:** `POST`  
**Deskripsi:** Endpoint ini digunakan untuk menukar `authorization code` dengan `access token`. Token ini diperlukan untuk mengakses endpoint lainnya.

**Parameter:**

| Nama           | Wajib | Deskripsi                                                                  |
|----------------|-------|---------------------------------------------------------------------------|
| `grant_type`   | Ya    | Jenis grant. Untuk authorization code, gunakan `authorization_code`.      |
| `client_id`    | Ya    | ID unik untuk aplikasi client yang didaftarkan di SSO server.             |
| `client_secret`| Ya    | Kunci rahasia yang diperoleh saat mendaftarkan aplikasi client.           |
| `redirect_uri` | Ya    | URL callback yang sama dengan yang digunakan saat mendapatkan kode.       |
| `code`         | Ya    | Kode otorisasi yang diperoleh dari endpoint authorize.                    |

**Headers:**

| Header            | Wajib | Deskripsi                               |
|-------------------|-------|-----------------------------------------|
| `Content-Type`    | Ya    | `application/x-www-form-urlencoded`    |

**Contoh Permintaan (Menggunakan `curl`):**

```bash
curl -X POST "https://gebyokkudus.kuduskab.go.id/oauth/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=authorization_code" \
-d "client_id=ISI_DENGAN_CLIENT_ID_GEBYOK_KUDUS" \
-d "client_secret=ISI_DENGAN_CLIENT_SECRET_GEBYOK_KUDUS" \
-d "redirect_uri=ISI_DENGAN_REDIRECT_URI_CLIENT" \
-d "code=ISI_DENGAN_AUTHORIZATION_CODE"
```

**Contoh Respons Berhasil:**

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

**Penjelasan Proses:**
1. Aplikasi client mengirimkan `authorization code` dan detail lain ke endpoint `token`.
2. Server mengembalikan `access token` dan `token_type`.
3. Access token ini digunakan untuk mengakses data pengguna di endpoint berikutnya.

---

### Endpoint 3: Mengakses Data Pengguna

**URL:** `https://gebyokkudus.kuduskab.go.id/api/user-sso`  
**Method:** `GET`  
**Deskripsi:** Setelah mendapatkan `access token`, gunakan endpoint ini untuk mengambil data profil pengguna yang telah terautentikasi.

**Headers:**

| Header             | Wajib | Deskripsi                                    |
|--------------------|-------|----------------------------------------------|
| `Authorization`    | Ya    | `Bearer` diikuti dengan `access token` valid |

**Contoh Permintaan (Menggunakan `curl`):**

```bash
curl -X GET "https://gebyokkudus.kuduskab.go.id/api/user-sso" \
-H "Authorization: Bearer ISI_DENGAN_ACCESS_TOKEN"
```

**Contoh Respons:**

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

**Penjelasan Struktur Respons:**

| Field             | Tipe Data        | Deskripsi                                                                 |
|-------------------|------------------|---------------------------------------------------------------------------|
| `id`              | Integer          | ID unik pengguna dalam sistem.                                           |
| `name`            | String           | Nama lengkap pengguna.                                                   |
| `email`           | String           | Alamat email utama pengguna.                                             |
| `nip`             | String           | Nomor Induk Pegawai pengguna atau `-` jika tidak tersedia.               |
| `email_pp`        | Array of String  | Daftar alamat email tambahan pengguna, jika ada.                         |
| `no_hp`           | String           | Nomor telepon pengguna.                                                  |
| `kd_satker`       | String           | Kode satuan kerja pengguna.                                              |
| `nama_satker`     | String           | Nama satuan kerja pengguna.                                              |
| `email_verified_at`| String/Null     | Tanggal dan waktu verifikasi email, `null` jika belum diverifikasi.      |
| `created_at`      | String           | Tanggal dan waktu pembuatan akun, dalam format ISO 8601.                 |
| `updated_at`      | String           | Tanggal dan waktu terakhir pembaruan data akun, dalam format ISO 8601.   |
| `role`            | String           | Peran atau jabatan pengguna dalam sistem.                                |

---

### Panduan Penggunaan

1. **Dapatkan Authorization Code:**
   - Arahkan pengguna ke endpoint `/oauth/authorize` dengan parameter yang sesuai.
   - Pengguna akan diminta untuk login dan memberikan izin kepada aplikasi.

2. **Tukar Authorization Code dengan Access Token:**
   - Kirim permintaan POST ke endpoint `/oauth/token` untuk menukar kode otorisasi dengan `access token`.
   - Token ini memungkinkan aplikasi client untuk mengakses endpoint terproteksi.

3. **Gunakan Access Token untuk Mengakses Data Pengguna:**
   - Gunakan `access token` yang diterima pada endpoint `/api/user-sso` untuk mendapatkan informasi profil pengguna.

--- 

Dengan mengikuti panduan ini, aplikasi client dapat memanfaatkan autentikasi SSO yang aman dan terpusat pada Gebyok Kudus.
