# LJI API Documentation - Profile & Settings

This document outlines the main aggregated profile endpoint as well as the individual endpoints used to modify profile and settings data.

## 1. API Specification

### A. Get Aggregated User Profile (`/v1/user/me`)

This is the primary endpoint for fetching all data related to the user's profile and settings in a single call.

- **Method & Path:** `GET /v1/user/me`
- **Description:** Retrieves a comprehensive object containing the user's profile, identity verification status, and all their personal settings.

**Success Response (200 OK)**
```json
{
  "profile": {
    "full_name": "John Doe",
    "email": "john.doe@example.com",
    "phone_number": "+628123456789",
    "profile_picture_url": "https://cdn.app.com/path/to/image.jpg",
    "points_balance": 1500
  },
  "identity": {
    "status": "VERIFIED",
    "id_card_number": "3171************01",
    "last_updated": "2026-01-15T10:00:00Z"
  },
  "settings": {
    "notifications": {
      "report_result": true,
      "winning_information": true,
      "promotions": true
    },
    "language": "id",
    "privacy": {
      "camera_sync_enabled": false,
      "location_sync_enabled": false
    },
    "security": {
      "is_pin_set": true,
      "is_biometric_enabled": false
    }
  }
}
```
*Note: The response structure is extensive. Fields are described in their corresponding `PUT`/`PATCH` sections below.*

---

### B. ID Card Verification Endpoints

**1. Upload ID Card for OCR**
- **Method & Path:** `POST /v1/user/identity/ocr`
- **Description:** Uploads an ID card image. The backend performs OCR and returns the extracted data, which the user can then confirm or edit.
- **Request:** `multipart/form-data` with a field named `image`.
- **Success Response (200 OK):**
```json
{
  "status": "SUCCESS",
  "data": {
    "province": "DKI JAKARTA",
    "city": "JAKARTA PUSAT",
    "id_card_number": "3171...",
    "name": "JOHN DOE",
    "address": "JL. CONTOH",
    "rt": "001",
    "rw": "002"
  }
}
```
- **Failure Response (422 Unprocessable Entity - Unparseable Image):**
```json
{
  "error": {
    "code": "OCR_FAILED",
    "message": "Image is unclear or not a valid ID card. Please upload a clearer photo."
  }
}
```

**2. Submit Identity Data**
- **Method & Path:** `PUT /v1/user/identity`
- **Description:** Submits the final, user-confirmed identity data for backend verification.
- **Payload:** The JSON object from the OCR response, potentially edited by the user.
- **Success Response:** `204 No Content`.

---

### C. Update Profile & Settings Endpoints

**1. Update Basic Profile**
- **Method & Path:** `PATCH /v1/user/profile`
- **Description:** Updates the user's name, email, or phone number.
- **Payload:** `{ "full_name": "John Doenut", "email": "j.doenut@example.com" }`
- **Success Response:** `200 OK` with the updated profile object.

**2. Update General Settings**
- **Method & Path:** `PUT /v1/user/settings`
- **Description:** A single endpoint to update various user-configurable settings.
- **Payload:**
```json
{
  "notifications": {
    "report_result": true,
    "winning_information": false,
    "promotions": true
  },
  "language": "en"
}
```
- **Success Response:** `204 No Content`.

**3. Set/Change Security PIN**
- **Method & Path:** `PUT /v1/user/security/pin`
- **Description:** Sets or changes the user's 6-digit security PIN. Requires current password for authorization.
- **Payload:**
```json
{
  "current_password": "user_current_password",
  "new_pin": "123456"
}
```
- **Success Response:** `204 No Content`.

---

### D. Static Content API

- **Method & Path:** `GET /v1/content?slug={slug}&language={lang}`
- **Description:** Fetches static content like Help Center articles, T&C, and Privacy Policy. The content is internationalized.
- **Query Parameters:**
    - `slug` (required): The unique identifier for the content (e.g., `help-center`, `terms-and-conditions`).
    - `language` (optional, default: 'id'): `id` or `en`.
- **Success Response (200 OK):**
```json
{
    "slug": "help-center",
    "title": "Help Center",
    "modified_at": "2026-02-20T10:00:00Z",
    "content_type": "markdown",
    "body": "## Brief Guides\n\n* Redeem\n* History\n\n## FAQs\n\n**Question 1?**\nAnswer 1."
}
```

## 2. Validations

### For `PATCH /v1/user/profile`

**Email Already in Use**
- **Trigger:** User tries to change their email to one that is already registered to another user.
- **HTTP Status:** `409 Conflict`
- **Error Response:**
```json
{
  "error": {
    "code": "EMAIL_EXISTS",
    "message": "This email address is already in use by another account."
  }
}
```

## 3. Specific Implementation Details

- **Biometric Security:** The backend does not handle fingerprints. It only stores a boolean flag `is_biometric_enabled`. The mobile client reads this flag from `GET /v1/user/me`. If `true`, the app prompts for a biometric scan *using the device's OS-level APIs*. On a successful scan, the app should then call a separate endpoint (e.g., `POST /v1/auth/token/biometric`) to get a short-lived API token.
- **Static Content:** The `GET /v1/content` endpoint is designed to be a single source for all Markdown/HTML based static pages. This simplifies content management and internationalization (i18n). A separate endpoint could provide the hierarchy or Table of Contents if needed.
- **Aggregation vs. Granularity:** The `GET /v1/user/me` endpoint is for convenience on the main profile screen. For screens that only edit one part of the profile (e.g., "Notification Settings"), it is more efficient for the client to use the smaller, more focused `PUT` endpoints (e.g., `PUT /v1/user/settings`) rather than re-submitting the entire aggregated object.
