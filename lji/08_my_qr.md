# LJI API Documentation - "My QR" for Redemption

## 1. API Specification

### A. Generate Redeem QR Code Data

- **Method & Path:** `GET /v1/user/redeem-qr`
- **Description:** Generates a short-lived, single-use token that the mobile client will render as a QR code. This code is presented to a merchant to initiate a point deduction (redeem) from the user's account.

**Success Response (200 OK)**

```json
{
  "qr_data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiY2E4YjliZjItNmU1ZC00YzNiLThhMWYtOWI4ZDdlNmM1YjRhIiwiZXhwIjoxNjc2ODg0ODAwfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
  "expires_at": "2026-02-20T10:01:00Z",
  "listen_topic": "redeem_result_ca8b9bf2-6e5d-4c3b-8a1f-9b8d7e6c5b4a"
}
```

**Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `qr_data` | String | The secure data to be encoded into the QR code. The client should not interpret this data, only render it. |
| `expires_at` | DateTime | An ISO 8601 timestamp indicating when this `qr_data` becomes invalid (typically 60 seconds after generation). The client should use this to show a countdown timer. |
| `listen_topic`| String | A unique topic name for a WebSocket or push notification channel. The client must subscribe to this topic to receive the asynchronous result of the redemption. |

---

## 2. Validations

### Account Not Permitted to Redeem
- **Trigger:** The user's account is suspended, not verified, or otherwise ineligible to perform redemptions.
- **HTTP Status:** `403 Forbidden`
- **Error Response:**
```json
{
  "error": {
    "code": "ACTION_NOT_ALLOWED",
    "message": "Your account is not permitted to perform redemptions at this time."
  }
}
```

### Too Many Frequent Requests
- **Trigger:** The same user requests a new QR code too many times in a short period.
- **HTTP Status:** `429 Too Many Requests`
- **Error Response:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "You are generating QR codes too frequently. Please wait a moment before trying again."
  }
}
```
---

## 3. Specific Implementation Details

### Asynchronous Redemption Flow

This feature operates on a synchronous/asynchronous model, which is critical for the client to implement correctly.

1.  **Synchronous QR Generation (This API):** The client calls `GET /v1/user/redeem-qr` to get the `qr_data`. It immediately renders this data as a QR code on the screen and starts a countdown timer based on the `expires_at` timestamp. Simultaneously, it subscribes to a WebSocket channel using the provided `listen_topic`.

2.  **Merchant Scan (Out of Band):** The user presents the QR code to the merchant. The merchant's system scans the code and calls a separate backend API (not documented here) to process the point deduction using the `qr_data`.

3.  **Asynchronous Result Push:** Once the backend processes the merchant's request, it determines if the redemption was successful or failed (e.g., due to insufficient points). It then pushes a message to the client via the WebSocket `listen_topic`.

### **Example Pushed Message (Success)**
The client will receive a JSON message on the WebSocket topic like this:
```json
{
  "status": "SUCCESS",
  "detail": {
    "store_name": "Coffee Corner",
    "address": "Jl. Melati No. 1, Jakarta",
    "redeem_code": "RD-ABC-123",
    "timestamp": "2026-02-20T10:00:45Z",
    "amount_redeemed": 50
  }
}
```

### **Example Pushed Message (Failed)**
If the user did not have enough points, the client will receive this message:
```json
{
  "status": "FAILED",
  "detail": {
    "reason": "Not enough points."
  }
}
```

The client application should listen for these messages and update the UI accordingly to show the user the final result of their redemption. If the connection is lost, the result will still be recorded and visible in the user's "History" screen.
