# LJI API Documentation - Store Reports

## 1. API Specification

### A. Submit New Report

- **Method & Path:** `POST /v1/reports`
- **Description:** Submits a new report about a store or establishment. This endpoint accepts `multipart/form-data` to handle image uploads.

**Request Body (multipart/form-data)**

| Field | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| `issue_type_id` | `UUID` | ID of the issue selected from the dropdown. | Yes |
| `category_id` | `UUID` | ID for the category (Store, Hotel, Restaurant). | Yes |
| `store_name` | `String` | The name of the reported establishment. | Yes |
| `location_address`| `String` | Full address or GPS coordinate string. | Yes |
| `proof_images` | `File[]` | Up to 5 images as evidence. | Yes |
| `notes` | `String` | Optional additional comments from the user. | No |

**Success Response (201 Created)**

```json
{
  "status": "success",
  "message": "Laporan berhasil dikirim dan sedang diproses.",
  "data": {
    "report_id": "c7a8b9f2-6e5d-4c3b-8a1f-9b8d7e6c5b4a",
    "status": "diproses",
    "submitted_at": "2026-02-19T14:30:00Z"
  }
}
```

### B. Get Report History

- **Method & Path:** `GET /v1/reports`
- **Description:** Retrieves a paginated list of reports submitted by the authenticated user.
- **Query Parameters:**
    - `status` (String): Filter by `diproses`, `disetujui`, or `ditolak`.
    - `page` (Integer): The page number to retrieve.
    - `limit` (Integer): The number of reports per page.

**Success Response (200 OK)**

```json
{
  "total_reports": 3,
  "page": 1,
  "limit": 10,
  "reports": [
    {
      "id": "c7a8b9f2-6e5d-4c3b-8a1f-9b8d7e6c5b4a",
      "store_name": "Hotel Mitra Nyaman",
      "status": "diproses",
      "submitted_at": "2026-02-19T14:30:00Z"
    },
    {
      "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
      "store_name": "Restoran Enak",
      "status": "disetujui",
      "submitted_at": "2026-02-18T11:00:00Z"
    }
  ]
}
```
**Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `total_reports` | Integer | The total number of reports matching the filter. |
| `page` | Integer | The current page number. |
| `limit` | Integer | The number of items per page. |
| `reports` | Array | An array of report summary objects. |
| `reports[].id` | UUID | The unique identifier for the report. |
| `reports[].store_name` | String | The name of the reported store. |
| `reports[].status` | String | The current status of the report. |
| `reports[].submitted_at`| DateTime | The timestamp when the report was submitted. |

### C. Get Report Detail

- **Method & Path:** `GET /v1/reports/{report_id}`
- **Description:** Fetches the full details of a specific report.

**Success Response (200 OK)**

```json
{
  "id": "c7a8b9f2-6e5d-4c3b-8a1f-9b8d7e6c5b4a",
  "store_name": "Hotel Mitra Nyaman",
  "status": "diproses",
  "submitted_at": "2026-02-19T14:30:00Z",
  "issue_type": "No QR Code",
  "category": "Hotel",
  "location_address": "Jl. Jend. Sudirman No.1, Jakarta",
  "notes": "QR Code tidak ditemukan di meja resepsionis.",
  "proof_image_urls": [
    "https://cdn.app.com/reports/image1.jpg",
    "https://cdn.app.com/reports/image2.jpg"
  ]
}
```
**Response Field Description** is self-explanatory based on the example.

### D. Get Master Data: Report Issues

- **Method & Path:** `GET /v1/master/report-issues`
- **Description:** Retrieves the list of possible issues to populate the report submission form.

**Success Response (200 OK)**
```json
[
    {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "name": "No QR Code"
    },
    {
        "id": "a9b8c7d6-e5f4-3a2b-1c0d-9e8f7a6b5c4d",
        "name": "Store Permanently Closed"
    }
]
```

---

## 2. Validations

### For **A. Submit New Report**

**Missing Required Field**
- **Trigger:** Submitting the form without a mandatory field like `store_name`.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "store_name is a required field."
  }
}
```

**Too Many Images**
- **Trigger:** Attaching more than 5 files to the `proof_images` field.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "You can upload a maximum of 5 images."
  }
}
```

**Duplicate Report (Idempotency)**
- **Trigger:** Submitting the same report twice in quick succession with the same idempotency key.
- **HTTP Status:** `409 Conflict`
- **Error Response:**
```json
{
  "error": {
    "code": "DUPLICATE_REQUEST",
    "message": "This report has already been submitted.",
    "conflicting_report_id": "c7a8b9f2-6e5d-4c3b-8a1f-9b8d7e6c5b4a"
  }
}
```

### For **C. Get Report Detail**

**Report Not Found**
- **Trigger:** Requesting a `report_id` that does not exist or does not belong to the user.
- **HTTP Status:** `404 Not Found`
- **Example Request:** `GET /v1/reports/00000000-0000-0000-0000-000000000000`
- **Error Response:**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Report not found."
  }
}
```

---

## 3. Specific Implementation Details

- **Image Management:** For the `proof_images`, the backend should upload these to a secure cloud storage bucket (like AWS S3) and only store the resulting URLs in the database. These URLs are then returned in the "Get Report Detail" endpoint.
- **Asynchronous Point Calculation:** Points should not be awarded immediately upon submission. After an admin approves a report (`status` changes to `disetujui`), a background job should be triggered to credit the user's account. This decouples the reporting system from the points/rewards system.
- **Idempotency:** To prevent duplicate reports from accidental double-clicks or network retries, the `POST /v1/reports` endpoint should support an `Idempotency-Key` header. The client should generate a unique UUID for this key for each submission attempt.
