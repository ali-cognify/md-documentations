# LJI API Documentation - Store Reports

## 1. API Specification

### A. Create New Report

- **Method & Path:** `POST /v1/reports`
- **Description:** Submits a new report about a tax object (store, hotel, etc.). This endpoint accepts `multipart/form-data` to handle image uploads.

**Request Body (multipart/form-data)**

| Field | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| `tax_object` | Enum | The type of establishment. E.g., `RESTORAN`, `HOTEL`. | Yes |
| `tax_object_name` | String | The name of the restaurant or hotel being reported. | Yes |
| `report_category` | Enum | The category of the issue. E.g., `NO_QR_CODE`, `INVALID_QR_CODE`, `OTHER`.| Yes |
| `tax_object_address`| String | The full address of the establishment. | Yes |
| `images` | File[] | Up to 5 images as evidence. | Yes |
| `remarks` | String | Optional additional comments from the user. | No |

**Success Response (201 Created)**

```json
{
  "message": "Report successfully submitted for review.",
  "report_id": "REP-20260220-00123",
  "estimated_follow_up": {
    "min_days": 1,
    "max_days": 3
  }
}
```

### B. Get Report List

- **Method & Path:** `GET /v1/reports`
- **Description:** Retrieves a paginated list of reports submitted by the user.
- **Query Parameters:**
    - `status` (Enum): Filter by `VERIFIKASI`, `DISETUJUI`, or `DITOLAK`.
    - `sort` (Enum): Sort by `timestamp_asc` or `timestamp_desc`.
    - `page` (Integer): The page number to retrieve.
    - `limit` (Integer): The number of reports per page.

**Success Response (200 OK)**

```json
{
  "pagination": { "total_items": 1, "total_pages": 1, "current_page": 1 },
  "reports": [
    {
      "report_number": "REP-20260220-00123",
      "tax_object_name": "Hotel Mitra Nyaman",
      "address": "Jl. Jend. Sudirman No.1, Jakarta",
      "timestamp": "2026-02-20T10:24:12Z",
      "report_category": "NO_QR_CODE",
      "status": "VERIFIKASI"
    }
  ]
}
```

### C. Get Report Detail

- **Method & Path:** `GET /v1/reports/{report_number}`
- **Description:** Fetches the full, polymorphic details of a specific report.

**Success Response (200 OK - Status: DISETUJUI)**

```json
{
  "tax_object": "HOTEL",
  "tax_object_name": "Hotel Mitra Nyaman",
  "report_category": "NO_QR_CODE",
  "tax_object_address": "Jl. Jend. Sudirman No.1, Jakarta",
  "report_number": "REP-20260220-00123",
  "report_status": "DISETUJUI",
  "timestamp": "2026-02-20T10:24:12Z",
  "images": [
      "https://cdn.app.com/reports/img1.jpg"
  ],
  "remarks": "QR Code tidak ada di meja resepsionis.",
  "result": {
      "received_points": 150,
      "approval_time": "2026-02-21T11:00:00Z"
  }
}
```

**Success Response (200 OK - Status: DITOLAK)**
```json
{
  "tax_object": "RESTORAN",
  // ... other fields
  "report_status": "DITOLAK",
  "result": {
      "rejection_time": "2026-02-21T09:30:00Z",
      "reject_reason": "Laporan tidak memiliki bukti yang cukup."
  }
}
```

### D. Get Master Data for Forms

- **Method & Path:** `GET /v1/master-data/report-options`
- **Description:** Retrieves the enum options for populating the dropdowns in the "Create New Report" form.

**Success Response (200 OK)**
```json
{
    "tax_objects": [
        {"key": "RESTORAN", "display_name": "Restoran/Rumah Makan"},
        {"key": "HOTEL", "display_name": "Hotel/Penginapan"}
    ],
    "report_categories": [
        {"key": "NO_QR_CODE", "display_name": "Belum memiliki Kode QR Pajak"},
        {"key": "INVALID_QR_CODE", "display_name": "Kode QR Pajak Invalid"},
        {"key": "OTHER", "display_name": "Lainnya"}
    ]
}
```

## 2. Validations

### For **A. Create New Report**

**Missing Required Field**
- **Trigger:** Submitting the form without a mandatory field like `tax_object_name`.
- **HTTP Status:** `400 Bad Request`
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "tax_object_name is a required field."
  }
}
```

**Too Many Images**
- **Trigger:** Attaching more than 5 files to the `images` field.
- **HTTP Status:** `413 Payload Too Large`
- **Error Response:**
```json
{
  "error": {
    "code": "PAYLOAD_TOO_LARGE",
    "message": "You can upload a maximum of 5 images."
  }
}
```

### For **C. Get Report Detail**

**Report Not Found**
- **Trigger:** Requesting a `report_number` that does not exist or does not belong to the user.
- **HTTP Status:** `404 Not Found`
- **Error Response:**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Report not found."
  }
}
```

## 3. Specific Implementation Details

- **Polymorphic `result` field:** In the `GET /v1/reports/{id}` response, the `result` object's structure changes based on the `report_status`. The client must check the status before attempting to access fields within `result`.
- **Image Handling:** The backend should process the uploaded images, store them in a secure cloud storage service, and associate the resulting URLs with the report. These URLs are returned in the detail view.
- **Master Data:** Caching the response from `GET /v1/master-data/report-options` on the client for a reasonable duration (e.g., 24 hours) is recommended to reduce unnecessary network calls.
