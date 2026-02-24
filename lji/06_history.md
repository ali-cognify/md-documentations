# LJI API Documentation - User History

## 1. API Specification

### A. Get History List

- **Method & Path:** `GET /v1/history`
- **Description:** Retrieves a paginated list of user activities (scans and redeems) for the "Riwayat" screen.

**Query Parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| `type` | Enum | Filter by activity type. Can be `SCAN` or `REDEEM`. |
| `status` | Enum | For `type=SCAN` only. Filter by scan status. Can be `ALL`, `VALID`, `INVALID`. |
| `start_date`| Date | The start of the date range to filter by (e.g., `2026-01-01`). |
| `end_date` | Date | The end of the date range to filter by (e.g., `2026-01-31`). |
| `page` | Integer| The page number to retrieve (default: 1). |
| `limit` | Integer| The number of items per page (default: 20). |

**Success Response (200 OK)**

```json
{
  "pagination": {
    "total_items": 2,
    "total_pages": 1,
    "current_page": 1
  },
  "history": [
    {
      "id": "hist_1a2b3c4d5e",
      "type": "SCAN",
      "status": "VALID",
      "merchant_name": "SuperMart",
      "timestamp": "2026-02-19T11:30:00Z",
      "amount": 100,
      "reward": "1 motor N-max"
    },
    {
      "id": "hist_6f7g8h9i0j",
      "type": "REDEEM",
      "status": "SUCCESS",
      "merchant_name": "Coffee Corner",
      "timestamp": "2026-02-18T16:00:00Z",
      "amount": -50,
      "reward": null
    }
  ]
}
```

**Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `history[].id` | String | The unique identifier for the history event. |
| `history[].type` | Enum | `SCAN` or `REDEEM`. |
| `history[].status` | Enum | `VALID`, `INVALID` for scans. `SUCCESS`, `FAILED` for redeems. |
| `history[].merchant_name` | String | The name of the merchant associated with the activity. |
| `history[].timestamp` | DateTime | The timestamp of the activity. |
| `history[].amount` | Integer | The points value. Positive for earning, negative for redeeming. |
| `history[].reward` | String | A description of any prize won from a scan. `null` if no prize. |


### B. Get History Detail

- **Method & Path:** `GET /v1/history/{id}`
- **Description:** Retrieves the detailed information for a single history event. The structure of the `details` object is polymorphic and changes based on the event type and status.

**Success Response: SCAN - VALID (200 OK)**

```json
{
  "id": "hist_1a2b3c4d5e",
  "type": "SCAN",
  "status": "VALID",
  "timestamp": "2026-02-19T11:30:00Z",
  "details": {
    "transaction_detail": {
      "store_name": "SuperMart",
      "company_name": "PT. SuperMart Indonesia Tbk",
      "address": "Jl. Gatot Subroto No. 42, Jakarta",
      "transaction_id": "TRX-20260219-001",
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

**Success Response: SCAN - INVALID (200 OK)**

```json
{
  "id": "hist_2b3c4d5e6f",
  "type": "SCAN",
  "status": "INVALID",
  "timestamp": "2026-02-17T09:15:00Z",
  "details": {
    "invalid_detail": {
      "description": "QR Code tidak terdaftar dalam sistem kami."
    },
    "potential_reward": {
      "min": 50,
      "max": 100
    },
    "reporting_status": "disetujui"
  }
}
```

**Success Response: REDEEM - SUCCESS (200 OK)**

```json
{
  "id": "hist_6f7g8h9i0j",
  "type": "REDEEM",
  "status": "SUCCESS",
  "timestamp": "2026-02-18T16:00:00Z",
  "details": {
    "redeem_detail": {
      "store_name": "Coffee Corner",
      "address": "Jl. Melati No. 1, Jakarta",
      "redeem_code": "RD-ABC-123",
      "amount_redeemed": 50
    }
  }
}
```

---

## 2. Validations

### For **A. Get History List**

**Invalid Filter Value**
- **Trigger:** Providing an unsupported value in the `type` or `status` query parameters.
- **HTTP Status:** `400 Bad Request`
- **Example Request:** `GET /v1/history?type=UNKNOWN`
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Invalid value for 'type' parameter. Allowed values are SCAN, REDEEM."
  }
}
```

### For **B. Get History Detail**

**History Item Not Found**
- **Trigger:** Requesting a history `id` that does not exist or does not belong to the user.
- **HTTP Status:** `404 Not Found`
- **Example Request:** `GET /v1/history/hist_xxxxxxxx`
- **Error Response:**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "History item not found."
  }
}
```

---

## 3. Specific Implementation Details

- **Polymorphic Responses:** The `GET /v1/history/{id}` endpoint utilizes a polymorphic response structure within the `details` field. The client application must be prepared to parse this object differently based on the top-level `type` and `status` fields.
- **Date Filtering:** When filtering by date range, the `timestamp` field should be used. The API should be inclusive of the `start_date` and `end_date`.
- **Push Mechanism for Redeem:** As mentioned in the context, a successful or failed redeem might be initiated by a third party (the store). While this API provides the record for the history screen, the real-time update to the client should be handled via a push notification (e.g., WebSocket or a standard push service) to ensure the user is notified promptly.
