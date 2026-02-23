# LJI API Documentation - Login

## 1. API Specification

### A. Login with Phone & Password

- **Method & Path:** `POST /v1/auth/login`
- **Description:** Validates user credentials and returns an access token.

**JSON Payload**

```json
{
  "phone_number": "8123456789",
  "country_code": "+62",
  "password": "user_password_here",
  "remember_me": true
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `phone_number` | String | User's phone number without the leading zero. | Yes |
| `country_code` | String | Country code for the phone number (e.g., "+62"). | Yes |
| `password` | String | User's password. | Yes |
| `remember_me`| Boolean | If true, the session will be extended. | No |

**Success Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "token_type": "Bearer",
    "access_token": "eyJhbG...",
    "refresh_token": "def456...",
    "expires_in": 3600,
    "user": {
      "id": "user_123",
      "full_name": "John Doe",
      "phone_number": "+628123456789",
      "is_verified": true
    }
  }
}
```

**Success Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `status` | String | Indicates the status of the response, e.g., "success". |
| `data` | Object | Contains the authentication data. |
| `token_type`| String | Type of token, typically "Bearer". |
| `access_token`| String | The JWT access token for authenticating subsequent requests. |
| `refresh_token`| String | The token used to obtain a new access token. |
| `expires_in`| Integer | The lifetime in seconds of the access token. |
| `user` | Object | Object containing user information. |
| `user.id` | String | The unique identifier for the user. |
| `user.full_name`| String | The full name of the user. |
| `user.phone_number`| String | The user's registered phone number. |
| `user.is_verified`| Boolean | Flag indicating if the user's account is verified. |


### B. Social Login (Google/Apple)

- **Method & Path:** `POST /v1/auth/social-login`
- **Description:** Authenticates users using identity tokens from Google or Apple.

**JSON Payload**

```json
{
  "provider": "google",
  "id_token": "eyJhbG...",
  "device_id": "uuid-string"
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `provider` | String | The social provider, either "google" or "apple". | Yes |
| `id_token` | String | The identity token received from the mobile SDK (Google/Apple). | Yes |
| `device_id`| String | A unique identifier for the device. | Yes |

**Success Response (200 OK)**
*Returns the same success response structure as **Login with Phone & Password**.*


### C. Guest Login

- **Method & Path:** `POST /v1/auth/guest-login`
- **Description:** Creates a temporary, restricted session for a guest user.

**JSON Payload**
*(No payload required)*

**Success Response (200 OK)**
*Returns the same success response structure as **Login with Phone & Password**, but with a restricted scope JWT and limited user data.*

---

## 2. Validations

This section outlines potential validation errors and their corresponding JSON responses.

### A. Field Kosong (Empty Field)
- **Trigger:** Submitting a required field without a value.
- **UI Screen Reference:** Field Kosong
- **HTTP Status:** `400 Bad Request`

**Example Payload:**
```json
{
  "phone_number": "",
  "country_code": "+62",
  "password": "user_password_here",
  "remember_me": true
}
```

**Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Wajib diisi"
  }
}
```

### B. Format Invalid (Invalid Format)
- **Trigger:** Phone number is not between 9-12 digits.
- **UI Screen Reference:** Format Invalid
- **HTTP Status:** `400 Bad Request`

**Example Payload:**
```json
{
  "phone_number": "123",
  "country_code": "+62",
  "password": "user_password_here",
  "remember_me": true
}
```

**Error Response:**
```json
{
  "error": {
    "code": "INVALID_FORMAT",
    "message": "Wajib 9-12 digit (tanpa 0 di depan)"
  }
}
```

### C. Password Salah (Incorrect Password)
- **Trigger:** Providing an incorrect password for a registered phone number.
- **UI Screen Reference:** Password Salah
- **HTTP Status:** `401 Unauthorized`

**Example Payload:**
```json
{
  "phone_number": "8123456789",
  "country_code": "+62",
  "password": "wrong_password",
  "remember_me": true
}
```

**Error Response:**
```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Kata sandi salah"
  }
}
```

### D. Akun Tidak Ditemukan (Account Not Found)
- **Trigger:** Attempting to log in with a phone number that is not registered.
- **UI Screen Reference:** Akun Tidak Ditemukan
- **HTTP Status:** `404 Not Found`

**Example Payload:**
```json
{
  "phone_number": "8999999999",
  "country_code": "+62",
  "password": "any_password",
  "remember_me": true
}
```

**Error Response:**
```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "No. Handphone ini belum terdaftar."
  }
}
```

### F. Terlalu Banyak Percobaan (Too Many Attempts)
- **Trigger:** Exceeding the allowed number of login attempts.
- **UI Screen Reference:** Terlalu Banyak Percobaan
- **HTTP Status:** `429 Too Many Requests`

**Example Payload:**
*(Same as any other login attempt after the rate limit is reached)*

**Error Response:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Terlalu Banyak Percobaan. Coba lagi dalam 05:00"
  }
}
```

### H. Error Server (Server Error)
- **Trigger:** An unexpected error occurs on the server.
- **UI Screen Reference:** Error Server (500)
- **HTTP Status:** `500 Internal Server Error`

**Example Payload:**
*(N/A - Can occur with any payload)*

**Error Response:**
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Ada kendala sistem. Coba beberapa saat lagi."
  }
}
```

### I. Akun Ditangguhkan (Account Suspended)
- **Trigger:** Attempting to log in with a suspended account.
- **UI Screen Reference:** Akun Ditangguhkan
- **HTTP Status:** `403 Forbidden`

**Example Payload:**
```json
{
  "phone_number": "8123456789",
  "country_code": "+62",
  "password": "correct_password_for_suspended_account",
  "remember_me": true
}
```

**Error Response:**
```json
{
  "error": {
    "code": "ACCOUNT_SUSPENDED",
    "message": "Akun ditangguhkan sementara"
  }
}
```

### J. Social Login Gagal (Social Login Failed)
- **Trigger:** The identity token from Google/Apple is invalid or expired.
- **UI Screen Reference:** Social Login Gagal
- **HTTP Status:** `401 Unauthorized`

**Example Payload:**
```json
{
  "provider": "google",
  "id_token": "invalid_or_expired_token",
  "device_id": "uuid-string"
}
```

**Error Response:**
```json
{
  "error": {
    "code": "SOCIAL_AUTH_FAILED",
    "message": "Gagal login dengan Google. Coba metode lain."
  }
}
```

---

## 3. Specific Implementation Details

- **Validation Patterns (Regex):** The backend and frontend should align on the phone number regex: `^[1-9][0-9]{8,11}$` (stripping the leading 0 as requested in Negative Case B).
- **Retry Logic:** For "Negative Case G" (Connection issue), this is typically handled by the mobile client before hitting the API, but the API should support a `Retry-After` header for the 429 Rate Limit error.
- **Security:** Ensure all endpoints are served over **HTTPS** and passwords are never returned in any response (even encrypted).
