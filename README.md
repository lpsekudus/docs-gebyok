# **Dokumentasi API Single Sign-On (SSO) Gebyok Kudus**

API Single Sign-On (SSO) Gebyok Kudus menyediakan layanan autentikasi dan otorisasi terpusat menggunakan protokol OAuth 2.0. API ini memungkinkan aplikasi klien mengakses data pengguna setelah melalui proses otorisasi. 

---

## **Daftar Endpoint**

1. **Mendapatkan Authorization Code**  
2. **Mendapatkan Access Token**  
3. **Mengakses Data Pengguna**

---

### **1. Endpoint: Mendapatkan Authorization Code**
- **URL**: `https://gebyokkudus.kuduskab.go.id/oauth/authorize`
- **Method**: `GET`  
- **Deskripsi**: Digunakan untuk meminta **Authorization Code** yang akan ditukarkan dengan **Access Token**. Authorization Code ini hanya berlaku satu kali dan memiliki waktu kedaluwarsa.

#### **Parameter Permintaan**
| Nama           | Wajib | Deskripsi                                                                 |
|-----------------|-------|---------------------------------------------------------------------------|
| `client_id`     | Ya    | ID unik aplikasi klien yang telah terdaftar di Gebyok Kudus.                   |
| `redirect_uri`  | Ya    | URL callback untuk menerima kode otorisasi setelah proses login selesai. |
| `response_type` | Ya    | Jenis respons yang diminta. Untuk Authorization Code, gunakan `code`.    |
| `scope`         | Ya    | Izin yang diminta oleh aplikasi klien, seperti `view-user`.              |
| `state`         | Ya    | Nilai unik dari klien untuk mencegah serangan CSRF.                                 |

#### **Contoh Permintaan**
```http
GET https://gebyokkudus.kuduskab.go.id/oauth/authorize?client_id=123456&redirect_uri=https://client-app.com/callback&response_type=code&scope=view-user&state=xyz123
```

#### **Proses**
1. Aplikasi klien mengarahkan pengguna ke endpoint ini.
2. Pengguna diminta login dan memberikan izin kepada aplikasi klien.
3. Setelah izin diberikan, server mengarahkan pengguna kembali ke `redirect_uri` dengan Authorization Code yang akan ditukarkan di endpoint berikutnya.

---

### **2. Endpoint: Mendapatkan Access Token**
- **URL**: `https://gebyokkudus.kuduskab.go.id/oauth/token`
- **Method**: `POST`  
- **Deskripsi**: Digunakan untuk menukar **Authorization Code** dengan **Access Token**.

#### **Parameter Permintaan**
| Nama           | Wajib | Deskripsi                                                                 |
|-----------------|-------|---------------------------------------------------------------------------|
| `grant_type`    | Ya    | Jenis grant. Untuk Authorization Code, gunakan `authorization_code`.     |
| `client_id`     | Ya    | ID unik aplikasi klien yang telah terdaftar di Gebyok Kudus.                   |
| `client_secret` | Ya    | Kunci rahasia aplikasi klien.                                            |
| `redirect_uri`  | Ya    | URL callback yang sama dengan yang digunakan saat mendapatkan kode.      |
| `code`          | Ya    | Authorization Code yang diperoleh dari endpoint sebelumnya.             |

#### **Headers**
| Header          | Wajib | Deskripsi                               |
|------------------|-------|-----------------------------------------|
| `Content-Type`   | Ya    | `application/x-www-form-urlencoded`    |

#### **Contoh Permintaan**
```bash
curl -X POST "https://gebyokkudus.kuduskab.go.id/oauth/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=authorization_code" \
-d "client_id=123456" \
-d "client_secret=abcdef" \
-d "redirect_uri=https://client-app.com/callback" \
-d "code=authorization_code_here"
```

#### **Contoh Respons**
```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

#### **Proses**
1. Aplikasi klien mengirimkan permintaan dengan Authorization Code dan detail lainnya.
2. Server memberikan **Access Token** yang dapat digunakan untuk mengakses data pengguna di endpoint berikutnya.
3. **Access Token** memiliki waktu kedaluwarsa (dalam detik, misalnya `3600` detik).

---

### **3. Endpoint: Mengakses Data Pengguna**
- **URL**: `https://gebyokkudus.kuduskab.go.id/api/user-sso`
- **Method**: `GET`  
- **Deskripsi**: Digunakan untuk mendapatkan data profil pengguna yang telah terautentikasi menggunakan **Access Token**.

#### **Headers**
| Header           | Wajib | Deskripsi                                      |
|-------------------|-------|-----------------------------------------------|
| `Authorization`   | Ya    | Bearer token yang valid (`Bearer <access_token>`) |

#### **Contoh Permintaan**
```bash
curl -X GET "https://gebyokkudus.kuduskab.go.id/api/user-sso" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
```

#### **Contoh Respons**
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

#### **Penjelasan Struktur Respons**
| Field              | Tipe Data      | Deskripsi                                      |
|--------------------|----------------|-----------------------------------------------|
| `id`               | Integer        | ID unik pengguna.                             |
| `name`             | String         | Nama lengkap pengguna.                        |
| `email`            | String         | Alamat email utama pengguna.                  |
| `nip`              | String         | Nomor Induk Pegawai atau `-` jika tidak ada.  |
| `email_pp`         | Array of String| Daftar email relasi Pejabat Pengadaan pengguna.               |
| `no_hp`            | String         | Nomor telepon pengguna.                       |
| `kd_satker`        | String         | Kode satuan kerja pengguna.                   |
| `nama_satker`      | String         | Nama satuan kerja pengguna.                   |
| `email_verified_at`| String/Null    | Waktu verifikasi email, `null` jika belum.    |
| `created_at`       | String         | Waktu pembuatan akun (format ISO 8601).       |
| `updated_at`       | String         | Waktu pembaruan terakhir data.                |
| `role`             | String         | Peran pengguna dalam sistem.                  |

---

## **Panduan Penggunaan**
1. **Dapatkan Authorization Code**  
   Arahkan pengguna ke endpoint `/oauth/authorize` dengan parameter yang diperlukan.  
2. **Tukar Authorization Code dengan Access Token**  
   Kirimkan Authorization Code ke endpoint `/oauth/token` untuk mendapatkan Access Token.  
3. **Akses Data Pengguna**  
   Gunakan Access Token yang diterima untuk mengakses endpoint `/api/user-sso`.  

---

### Dokumentasi Error API Gebyok Kudus

Dokumentasi ini menjelaskan kemungkinan error yang dapat terjadi pada sistem Single Sign-On (SSO) menggunakan OAuth2 di Gebyok Kudus. Penanganan error yang tepat akan membantu klien untuk memberikan feedback yang jelas dan memudahkan proses debugging.


### **Error pada Endpoint**

#### 1. **Mendapatkan Authorization Code**
   - Endpoint: `/oauth/authorize`
   - Fungsi: Mengarahkan pengguna untuk login dan memberikan authorization code.
   - **Kemungkinan Error**:

     - **Invalid Client**  
       - **Error Code**: `invalid_client`  
       - **Deskripsi**: Kesalahan ini terjadi ketika aplikasi klien tidak dapat terautentikasi dengan benar di server OAuth.
       - **Pesan**:  
         ```json
         {
           "error": "invalid_client",
           "error_description": "Client authentication failed",
           "message": "Client authentication failed"
         }
         ```

     - **Invalid Redirect URI**  
       - **Error Code**: `invalid_request`  
       - **Deskripsi**: Redirect URI yang diberikan tidak sesuai atau tidak terdaftar di aplikasi klien.
       - **Pesan**:  
         ```json
         {
           "error": "invalid_request",
           "error_description": "Invalid redirect URI",
           "message": "Invalid redirect URI"
         }
         ```

     - **Invalid Scope**  
       - **Error Code**: `invalid_scope`  
       - **Deskripsi**: Scope yang diminta tidak valid atau tidak didukung oleh server OAuth.
       - **Pesan**:  
         ```json
         {
           "error": "invalid_scope",
           "error_description": "Scope is invalid",
           "message": "Invalid scope requested"
         }
         ```

#### 2. **Mendapatkan Access Token**
   - Endpoint: `/oauth/token`
   - Fungsi: Mengambil token akses dengan menggunakan authorization code.
   - **Kemungkinan Error**:

     - **Invalid Grant**  
       - **Error Code**: `invalid_grant`  
       - **Deskripsi**: Kesalahan ini terjadi jika authorization code yang diberikan tidak valid atau sudah kadaluarsa.
       - **Pesan**:  
         ```json
         {
           "error": "invalid_grant",
           "error_description": "The provided authorization grant is invalid, expired, or revoked",
           "message": "Authorization grant is invalid or expired"
         }
         ```

     - **Invalid Client**  
       - **Error Code**: `invalid_client`  
       - **Deskripsi**: Klien gagal dalam autentikasi karena informasi `client_id` atau `client_secret` yang salah.
       - **Pesan**:  
         ```json
         {
           "error": "invalid_client",
           "error_description": "Client authentication failed",
           "message": "Client authentication failed"
         }
         ```

     - **Missing Client Credentials**  
       - **Error Code**: `invalid_request`  
       - **Deskripsi**: Tidak ada `client_id` atau `client_secret` yang diberikan pada permintaan.
       - **Pesan**:  
         ```json
         {
           "error": "invalid_request",
           "error_description": "Missing client credentials",
           "message": "Client credentials are required"
         }
         ```

     - **Unauthorized Client**  
       - **Error Code**: `unauthorized_client`  
       - **Deskripsi**: Klien tidak diizinkan untuk menggunakan grant type yang diminta.
       - **Pesan**:  
         ```json
         {
           "error": "unauthorized_client",
           "error_description": "The client is not authorized to use this grant type",
           "message": "Client is unauthorized for this grant type"
         }
         ```

#### 3. **Mengakses Data Pengguna**
   - Endpoint: `/api/user-sso`
   - Fungsi: Mengakses data pengguna yang telah login dengan token akses.
   - **Kemungkinan Error**:

     - **Unauthorized**  
       - **Error Code**: `unauthorized`  
       - **Deskripsi**: Token akses tidak valid atau tidak ada token akses dalam permintaan.
       - **Pesan**:  
         ```json
         {
           "error": "unauthorized",
           "error_description": "Access token is missing or invalid",
           "message": "Unauthorized access. Please check your access token."
         }
         ```

     - **Token Expired**  
       - **Error Code**: `token_expired`  
       - **Deskripsi**: Token akses yang digunakan telah kadaluarsa.
       - **Pesan**:  
         ```json
         {
           "error": "token_expired",
           "error_description": "The access token has expired",
           "message": "Access token has expired. Please reauthorize."
         }
         ```

     - **Insufficient Scope**  
       - **Error Code**: `insufficient_scope`  
       - **Deskripsi**: Token akses yang diberikan tidak memiliki izin untuk mengakses data yang diminta.
       - **Pesan**:  
         ```json
         {
           "error": "insufficient_scope",
           "error_description": "The access token does not have sufficient scope",
           "message": "You do not have sufficient scope to access this data"
         }
         ```

     - **Internal Server Error**  
       - **Error Code**: `server_error`  
       - **Deskripsi**: Terjadi kesalahan pada server saat memproses permintaan.
       - **Pesan**:  
         ```json
         {
           "error": "server_error",
           "error_description": "Internal server error",
           "message": "An internal server error occurred. Please try again later."
         }
         ```

---

Dengan mengikuti langkah-langkah di atas, aplikasi klien dapat menggunakan API SSO Gebyok Kudus secara aman dan efisien.
