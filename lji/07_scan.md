# LJI API Documentation - QR Code Scan

## 1. API Specification

### A. Verify Scanned QR Code

- **Method & Path:** `POST /v1/scans/verify`
- **Description:** Submits the raw text data from a scanned QR code to the backend for verification and processing. The response is polymorphic and depends on whether the scan is considered valid or invalid by the business logic.

**JSON Payload**

```json
{
  "qr_code_data": "raw-text-from-the-scanned-qr-code"
}
```

**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `qr_code_data` | String | The raw, unmodified string data obtained from scanning the QR code. | Yes |

---

### **Positive Case: Valid Scan**

This response is returned when the QR code is successfully validated and processed.

**Success Response (200 OK)**

```json
{
  "status": "VALID",
  "data": {
    "transaction_detail": {
      "store_name": "SuperMart",
      "company_name": "PT. SuperMart Indonesia Tbk",
      "address": "Jl. Gatot Subroto No. 42, Jakarta",
      "transaction_id": "TRX-20260219-001",
      "transaction_timestamp": "2026-02-19T11:30:00Z",
      "total_before_tax": 100000,
      "applied_tax": 11000,
      "grand_total": 111000
    },
    "point_detail": {
      "points_received": 100,
      "valid_until": "2027-02-19T11:30:00Z"
    }
  }
}
```

**Field Description** is provided in the `history.md` documentation and is self-explanatory.

---

### **Business Logic Case: Invalid Scan**

This response is returned when the API call is successful, but the QR code fails business logic validation (e.g., it's not found, has been used, or is malformed for our system).

**Success Response (200 OK)**

```json
{
  "status": "INVALID",
  "data": {
    "invalid_detail": {
      "reason_code": "QR_CODE_NOT_FOUND",
      "description": "QR Code tidak terdaftar dalam sistem kami."
    },
    "potential_reward": {
      "min": 50,
      "max": 100
    }
  }
}
```

**Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `status` | Enum | Always `INVALID` in this case. |
| `data.invalid_detail.reason_code`| Enum | A machine-readable code for the failure (e.g., `QR_CODE_NOT_FOUND`, `QR_CODE_ALREADY_USED`, `QR_CODE_EXPIRED`). |
| `data.invalid_detail.description` | String | A human-readable explanation of why the scan is invalid. |
| `data.potential_reward` | Object | Contains the potential points a user could get for reporting this invalid scan. |


---

## 2. Validations

This section describes standard client and server errors.

**Missing Payload Field**
- **Trigger:** Sending a request with an empty or missing `qr_code_data` field.
- **HTTP Status:** `400 Bad Request`
- **Example Payload:** `{}`, or `{"qr_code_data": ""}`
- **Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "qr_code_data is a required field and cannot be empty."
  }
}
```

**QR Code Already Scanned by User**
- **Trigger:** The user attempts to scan the exact same QR code more than once.
- **HTTP Status:** `422 Unprocessable Entity`
- **Error Response:**
```json
{
  "error": {
    "code": "ALREADY_PROCESSED",
    "message": "This QR code has already been scanned and processed."
  }
}
```

**Rate Limiting**
- **Trigger:** A user or device submits scan requests too frequently in a short period.
- **HTTP Status:** `429 Too Many Requests`
- **Error Response:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "You are scanning too frequently. Please try again in a moment."
  }
}
```

---

## 3. Specific Implementation Details

- **Polymorphic Response Handling:** The client application must first check the top-level `status` field in the 200 OK response to determine how to parse the `data` object. The structure of `data` will be completely different for `VALID` and `INVALID` statuses.
- **Idempotency:** To prevent issues with network retries causing multiple scans, it is recommended that the client generates a unique idempotency key (e.g., a UUID) and sends it in an `Idempotency-Key` header. The backend should store and check this key for a short period (e.g., 24 hours) to ensure the same operation is not processed twice.
- **History Record Creation:** A successful call to this endpoint (regardless of whether the status is `VALID` or `INVALID`) should create a new entry in the user's activity history, which can then be retrieved via the `GET /v1/history` endpoint.
