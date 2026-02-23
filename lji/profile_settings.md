# LJI API Documentation - Profile & Settings

## 1. API Specification

### A. Get User Profile

- **Method & Path:** `GET /v1/user/profile`
- **Description:** Retrieves the user's basic information and verification status for the "Kelola Akun" and "Profil" screens.

**Success Response (200 OK)**

```json
{
  "full_name": "Clarissa Putri",
  "email": "cl****ri@gmail.com",
  "phone_number": "+628123****9",
  "profile_picture_url": "https://cdn.app.com/path/to/image.jpg",
  "is_ktp_verified": false
}
```

**Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `full_name` | String | User's full name. |
| `email` | String | User's email address (partially masked). |
| `phone_number`| String | User's phone number (partially masked). |
| `profile_picture_url`| String | URL to the user's profile picture. Can be null. |
| `is_ktp_verified`| Boolean | `true` if the user has completed KTP verification. |

### B. Update User Profile

- **Method & Path:** `PATCH /v1/user/profile`
- **Description:** Updates user identity details (Name/Email/Phone). Partial updates are allowed.

**JSON Payload**

```json
{
  "full_name": "Clarissa Putri Anindya",
  "email": "clarissa.putri@example.com"
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `full_name` | String | The user's new full name. | No |
| `email` | String | The user's new email. Will require verification. | No |
| `phone_number`| String | The user's new phone number. Will require verification. | No |

**Success Response (200 OK)**
*Returns the updated user profile object as shown in **Get User Profile**.*

### C. Upload Profile Photo

- **Method & Path:** `POST /v1/user/profile/photo`
- **Description:** Uploads a new profile picture from the gallery or camera. Uses multipart form data.

**Request Body**
- `image_file`: The image file to upload.

**Success Response (200 OK)**
```json
{
  "profile_picture_url": "https://cdn.app.com/path/to/new_image.jpg"
}
```

### D. Update Notification Preferences

- **Method & Path:** `PUT /v1/user/settings/notifications`
- **Description:** Saves the user's notification toggle states.

**JSON Payload**
```json
{
  "important_updates": true,
  "reward_info": false,
  "promotions": true,
  "weekly_summary": false
}
```

**Success Response (204 No Content)**
*The server will respond with a `204 No Content` status on successful update with no response body.*


### E. Change Password

- **Method & Path:** `POST /v1/user/account/change-password`
- **Description:** Updates the user's password after verifying their current one.

**JSON Payload**

```json
{
  "current_password": "old_password_123",
  "new_password": "new_secure_password_456",
  "confirm_new_password": "new_secure_password_456"
}
```
**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `current_password`| String | The user's current password. | Yes |
| `new_password` | String | The user's new password. Must meet complexity requirements. | Yes |
| `confirm_new_password`| String | Confirmation of the new password. Must match `new_password`. | Yes |

**Success Response (204 No Content)**
*The server will respond with a `204 No Content` status on successful update with no response body.*

---

## 2. Validations

### For **B. Update User Profile**

**Invalid Email Format**
- **Trigger:** Submitting an email address with an invalid format.
- **HTTP Status:** `400 Bad Request`
- **Example Payload:**
```json
{
  "email": "invalid-email-format"
}
```
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_FORMAT",
    "message": "Format email tidak valid."
  }
}
```

**Full Name Too Long**
- **Trigger:** Submitting a `full_name` that exceeds the maximum length (e.g., 100 characters).
- **HTTP Status:** `400 Bad Request`
- **Example Payload:**
```json
{
  "full_name": "This name is far too long and surely exceeds the database constraints that have been put in place for this particular field..."
}
```
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Nama lengkap tidak boleh lebih dari 100 karakter."
  }
}
```

### For **E. Change Password**

**Incorrect Current Password**
- **Trigger:** The `current_password` does not match the stored password.
- **HTTP Status:** `401 Unauthorized`
- **Example Payload:**
```json
{
  "current_password": "wrong_old_password",
  "new_password": "new_secure_password_456",
  "confirm_new_password": "new_secure_password_456"
}
```
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Kata sandi saat ini salah."
  }
}
```

**New Passwords Do Not Match**
- **Trigger:** The `new_password` and `confirm_new_password` fields do not match.
- **HTTP Status:** `400 Bad Request`
- **Example Payload:**
```json
{
  "current_password": "old_password_123",
  "new_password": "new_secure_password_456",
  "confirm_new_password": "a_different_password"
}
```
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Konfirmasi kata sandi baru tidak cocok."
  }
}
```

**New Password Same as Old Password**
- **Trigger:** The `new_password` is the same as the `current_password`.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Kata sandi baru tidak boleh sama dengan kata sandi lama."
  }
}
```

---

## 3. Specific Implementation Details

- **Session Invalidation on Password Change:** For enhanced security, after a successful password change, all other active sessions and their corresponding refresh tokens for that user should be invalidated immediately.

- **Multipart/form-data for Photo Upload:** The `POST /v1/user/profile/photo` endpoint must handle `multipart/form-data` requests to accept the image file. The server should validate the file type (e.g., JPEG, PNG) and size.
