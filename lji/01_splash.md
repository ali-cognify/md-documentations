# LJI API Documentation - Splash & App Versioning

## 1. API Specification

### A. Check App Version

- **Method & Path:** `GET /v1/app/check-version`
- **Description:** Checks the current version of the mobile application against the latest available version on the server to determine if an update is required or recommended.

**Query Parameters**

| Parameter | Type | Description | Required |
| --- | --- | --- | --- |
| `platform` | Enum | The mobile platform, either `android` or `ios`. | Yes |
| `current_version` | String | The version string currently installed on the user's device (e.g., `1.0.2`). | Yes |

**Success Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "update_status": "FORCE_UPDATE",
    "latest_version": "1.1.0",
    "download_url": "https://play.google.com/store/apps/details?id=id.gov.jakarta.lji",
    "message": "Pembaruan wajib tersedia. Silakan perbarui aplikasi Anda untuk melanjutkan.",
    "maintenance": {
      "is_under_maintenance": false,
      "estimated_end_time": null
    }
  }
}
```

**Success Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `status` | String | Indicates the status of the response. |
| `data.update_status` | Enum | Result of the version check: `UP_TO_DATE`, `OPTIONAL_UPDATE`, or `FORCE_UPDATE`. |
| `data.latest_version` | String | The most recent version available on the store. |
| `data.download_url` | String | The link to the App Store or Google Play Store. |
| `data.message` | String | Localized message to be displayed to the user. |
| `data.maintenance.is_under_maintenance` | Boolean | If true, the app should show a maintenance screen. |
| `data.maintenance.estimated_end_time` | String | ISO 8601 timestamp of when maintenance is expected to finish. |

---

## 2. Validations

### A. Field Kosong (Empty Field)
- **Trigger:** Missing `platform` or `current_version` in query parameters.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Platform and current_version are required."
  }
}
```

### B. Platform Tidak Valid (Invalid Platform)
- **Trigger:** Providing a platform other than `android` or `ios`.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_PLATFORM",
    "message": "Platform must be 'android' or 'ios'."
  }
}
```

---

## 3. Specific Implementation Details

- **Version Comparison:** The backend should use semantic versioning (SemVer) comparison logic to determine the `update_status`.
- **Update Logic:**
    - `UP_TO_DATE`: `current_version` == `latest_version`.
    - `OPTIONAL_UPDATE`: `current_version` < `latest_version`, but the minimum required version is still met.
    - `FORCE_UPDATE`: `current_version` is below the `minimum_required_version` defined in the server configuration.
- **Maintenance Mode:** This endpoint is typically the first call made by the app. Including a `maintenance` object allows the backend to gracefully shut down app access during system updates or outages.
- **Localization:** The `message` field should be returned in the user's preferred language (e.g., based on the `Accept-Language` header).
