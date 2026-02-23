# LJI API Documentation - Privacy & Account

## 1. API Specification

### A. Get Security Settings

- **Method & Path:** `GET /v1/user/settings/security`
- **Description:** Retrieves the user's current security settings, such as app lock and PIN status.

**Success Response (200 OK)**

```json
{
  "app_lock_enabled": true,
  "biometric_enabled": true,
  "pin_lock_enabled": true,
  "lock_timeout_seconds": 30
}
```

**Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `app_lock_enabled` | Boolean | `true` if the main app lock feature is enabled. |
| `biometric_enabled` | Boolean | `true` if biometric authentication (Face/Touch ID) is active. |
| `pin_lock_enabled`| Boolean | `true` if a PIN is required for secure actions. |
| `lock_timeout_seconds`| Integer | The number of seconds of inactivity before the app requires re-authentication. |

### B. Get Privacy Settings

- **Method & Path:** `GET /v1/user/settings/privacy`
- **Description:** Retrieves device permission states that the user has granted or denied via the app.

**Success Response (200 OK)**

```json
{
  "permissions": {
    "camera": true,
    "location": false
  }
}
```

**Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `permissions` | Object | A map of permissions and their status. |
| `permissions.camera` | Boolean | `true` if the app has been granted camera access. |
| `permissions.location`| Boolean | `true` if the app has been granted location access. |

### C. Request Data Export

- **Method & Path:** `POST /v1/user/privacy/export-data`
- **Description:** Initiates an asynchronous job to export the user's data. The data will be sent to the user's verified email address.

**Success Response (202 Accepted)**

```json
{
  "message": "Permintaan unduh data sedang diproses dan akan dikirim ke email Anda.",
  "job_id": "export_88291",
  "estimated_completion": "2026-02-20T11:00:00Z"
}
```

### D. Clear Activity History

- **Method & Path:** `DELETE /v1/user/privacy/history`
- **Description:** Deletes user activity logs, such as search history or viewed items. Can target all history or a specific type.
- **Query Parameters:** `?type=<history_type>` (e.g., `search`, `view`). If omitted, deletes all history.

**Success Response (204 No Content)**
*The server will respond with a `204 No Content` status on successful deletion with no response body.*


### E. Deactivate/Delete Account

- **Method & Path:** `POST /v1/user/account/deactivate`
- **Description:** Initiates the account deactivation or permanent deletion process. Requires password confirmation for security.

**JSON Payload**

```json
{
  "reason": "I am no longer using the application.",
  "password_confirmation": "current_user_password"
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `reason` | String | The reason for deactivation. | No |
| `password_confirmation` | String | The user's current password to confirm the action. | Yes |


**Success Response (200 OK)**
```json
{
  "message": "Akun Anda telah dinonaktifkan dan akan dihapus secara permanen dalam 30 hari.",
  "deactivation_complete_until": "2026-03-21T10:00:00Z"
}
```

---

## 2. Validations

### For **C. Request Data Export**

**Rate Limit Exceeded**
- **Trigger:** The user requests a data export when a recent one is already in progress or was just completed.
- **HTTP Status:** `429 Too Many Requests`
- **Error Response:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Anda baru saja meminta ekspor data. Silakan coba lagi nanti."
  }
}
```

### For **D. Clear Activity History**

**Invalid History Type**
- **Trigger:** Providing an unsupported `type` in the query parameter.
- **HTTP Status:** `400 Bad Request`
- **Example Request:** `DELETE /v1/user/privacy/history?type=invalid_type`
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Tipe riwayat tidak valid."
  }
}
```

### For **E. Deactivate/Delete Account**

**Incorrect Password Confirmation**
- **Trigger:** The provided `password_confirmation` does not match the user's current password.
- **HTTP Status:** `401 Unauthorized`
- **Example Payload:**
```json
{
  "reason": "Test",
  "password_confirmation": "wrong_password"
}
```
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Kata sandi yang Anda masukkan salah."
  }
}
```

---

## 3. Specific Implementation Details

- **Account Deletion Grace Period:** As shown in the success response for account deactivation, a 30-day grace period is implemented. If the user logs in during this period, the deactivation is canceled. After 30 days, the user's data is permanently purged from the system.
- **Asynchronous Data Export:** The data export process is handled in the background to avoid long-running requests. The user is notified by email with a secure, time-limited link to download their data archive once it's ready.
- **Authentication:** All endpoints listed in this document require a valid `Bearer Token` in the `Authorization` header. Unauthorized access attempts should result in a `401 Unauthorized` response.
