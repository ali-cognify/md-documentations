# LJI API Documentation - Register & OTP

## 1. API Specification

### A. Register Account

- **Method & Path:** `POST /v1/auth/register`
- **Description:** Registers a new user and triggers an OTP send to the provided mobile number.

**JSON Payload**

```json
{
  "country_code": "+62",
  "phone_number": "8123456789",
  "full_name": "John Doe",
  "password": "secure_password",
  "password_confirmation": "secure_password",
  "agree_terms": true
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `country_code` | String | Country code for the phone number (e.g., "+62"). | Yes |
| `phone_number` | String | User's phone number without the leading zero. | Yes |
| `full_name` | String | User's full name. | Yes |
| `password` | String | Desired password for the account. | Yes |
| `password_confirmation` | String | Must match the `password` field. | Yes |
| `agree_terms` | Boolean | Must be true to indicate agreement with T&C and Privacy Policy. | Yes |

**Success Response (200 OK)**

```json
{
  "status": "success",
  "message": "Registration data saved. Please verify OTP.",
  "data": {
    "phone_number": "+628123456789",
    "otp_metadata": {
      "expired_at": "2026-03-03T10:15:00Z",
      "expires_in": 300
    }
  }
}
```

**Success Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `status` | String | Indicates the status of the response. |
| `data.otp_metadata.expired_at` | String | ISO 8601 timestamp when the current OTP will expire. |
| `data.otp_metadata.expires_in` | Integer | Remaining time in seconds before OTP expires. |


### B. Send/Resend OTP

- **Method & Path:** `POST /v1/auth/otp/resend`
- **Description:** Resends a verification OTP to the user's mobile number.

**JSON Payload**

```json
{
  "country_code": "+62",
  "phone_number": "8123456789"
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `country_code` | String | Registered country code. | Yes |
| `phone_number` | String | Registered phone number. | Yes |

**Success Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "expired_at": "2026-03-03T10:20:00Z",
    "expires_in": 300
  }
}
```


### C. Verify OTP

- **Method & Path:** `POST /v1/auth/otp/verify`
- **Description:** Validates the OTP code to complete the registration process and activate the account.

**JSON Payload**

```json
{
  "country_code": "+62",
  "phone_number": "8123456789",
  "otp_code": "123456"
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `country_code` | String | Registered country code. | Yes |
| `phone_number` | String | Registered phone number. | Yes |
| `otp_code` | String | The 6-digit OTP code received by the user. | Yes |

**Success Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "token_type": "Bearer",
    "access_token": "eyJhbG...",
    "user": {
      "id": "user_123",
      "full_name": "John Doe",
      "phone_number": "+628123456789",
      "is_verified": true
    }
  }
}
```

---

## 2. Validations

### A. Field Kosong (Empty Field)
- **Trigger:** Submitting registration or OTP without required fields.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Field [field_name] wajib diisi."
  }
}
```

### B. Password Mismatch
- **Trigger:** `password` and `password_confirmation` do not match.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "PASSWORD_MISMATCH",
    "message": "Konfirmasi kata sandi tidak sesuai."
  }
}
```

### C. Terms Not Agreed
- **Trigger:** `agree_terms` is false.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "AGREEMENT_REQUIRED",
    "message": "Anda harus menyetujui Syarat & Ketentuan."
  }
}
```

### D. Phone Number Already Registered
- **Trigger:** Attempting to register with a phone number that is already active.
- **HTTP Status:** `409 Conflict`
- **Error Response:**
```json
{
  "error": {
    "code": "PHONE_ALREADY_EXISTS",
    "message": "Nomor handphone sudah terdaftar."
  }
}
```

### E. Invalid/Expired OTP
- **Trigger:** Entering a wrong OTP or an OTP that has passed its `expired_at` time.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_OTP",
    "message": "Kode OTP salah atau sudah kadaluwarsa."
  }
}
```

---

## 3. Specific Implementation Details

- **Phone Number Normalization:** The backend must normalize phone numbers (e.g., removing leading zeros) before storing or checking against the database.
- **OTP Generation:** OTP should be a 6-digit numeric code.
- **OTP Rate Limiting:** Implement a cooldown period (e.g., 60 seconds) between "Resend OTP" requests to prevent spam.
- **Security:** Passwords must be hashed using a strong algorithm (e.g., Argon2 or BCrypt) before storage. The `otp_code` should also be stored securely and deleted after successful verification or expiry.
- **Session Continuity:** The registration data can be stored in a temporary state (e.g., Redis or a 'pending_users' table) until the OTP is successfully verified.
