# LJI API Documentation - Notifications

## 1. API Specification

### A. Get Notification List

- **Method & Path:** `GET /v1/notifications`
- **Description:** Retrieves a paginated list of notifications for the authenticated user.

**Query Parameters**
- `page` (Integer, optional, default: 1): The page number to retrieve.
- `limit` (Integer, optional, default: 20): The number of notifications per page.

**Success Response (200 OK)**

```json
{
  "pagination": {
    "total_items": 3,
    "total_pages": 1,
    "current_page": 1
  },
  "notifications": [
    {
      "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
      "type": "REPORT_APPROVED",
      "title": "Laporan Disetujui",
      "description": "Laporan Anda untuk 'Hotel Mitra Nyaman' telah disetujui. Poin Anda telah ditambahkan.",
      "is_read": false,
      "created_at": "2026-02-19T10:00:00Z",
      "action_metadata": {
        "report_id": "c7a8b9f2-6e5d-4c3b-8a1f-9b8d7e6c5b4a"
      }
    },
    {
      "id": "b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7e",
      "type": "IDENTITY_VERIFICATION",
      "title": "Lengkapi Profil Anda",
      "description": "Selangkah lagi untuk menjadi member terverifikasi. Lengkapi data diri Anda sekarang.",
      "is_read": false,
      "created_at": "2026-02-18T15:30:00Z",
      "action_metadata": null
    },
    {
      "id": "c3d4e5f6-a7b8-9c0d-1e2f-3a4b5c6d7e8f",
      "type": "ENGAGEMENT",
      "title": "Kami Merindukanmu!",
      "description": "Sudah lama tidak bertemu. Cek promo dan hadiah terbaru yang mungkin Anda lewatkan.",
      "is_read": true,
      "created_at": "2026-02-15T09:00:00Z",
      "action_metadata": null
    }
  ]
}
```

**Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `pagination` | Object | Contains pagination details. |
| `notifications` | Array | A list of notification objects. |
| `notifications[].id` | UUID | The unique identifier for the notification. |
| `notifications[].type` | Enum | The type of notification. Possible values: `REPORT_PROCESSED`, `REPORT_APPROVED`, `REPORT_REJECTED`, `IDENTITY_VERIFICATION`, `WINNER_ANNOUNCEMENT`, `ENGAGEMENT`. |
| `notifications[].title` | String | The main title of the notification. |
| `notifications[].description` | String | The detailed body text of the notification. |
| `notifications[].is_read` | Boolean | `true` if the user has marked the notification as read. |
| `notifications[].created_at`| DateTime | The timestamp when the notification was created. |
| `notifications[].action_metadata`| Object | An object containing data for client-side actions, like the `report_id` for report-related notifications. Can be `null`. |


### B. Get Notification Status

- **Method & Path:** `GET /v1/notifications/status`
- **Description:** A lightweight endpoint to retrieve the count of unread notifications, typically for displaying a badge.

**Success Response (200 OK)**

```json
{
  "unread_count": 2
}
```

### C. Mark Notifications as Read

- **Method & Path:** `POST /v1/notifications/mark-as-read`
- **Description:** Marks one or more notifications as read.

**JSON Payload**

```json
{
  "notification_ids": [
    "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7e"
  ]
}
```
**Payload Field Description**

| Field | Type | Description | Required |
| --- | --- | --- | --- |
| `notification_ids`| Array of UUIDs | A list of notification IDs to mark as read. To mark all as read, the client can send all unread notification IDs it has. | Yes |

**Success Response (204 No Content)**
*The server will respond with a `204 No Content` status on a successful update with no response body.*

---

## 2. Validations

### For **C. Mark Notifications as Read**

**Invalid UUID Format**
- **Trigger:** Sending a string in the `notification_ids` array that is not a valid UUID.
- **HTTP Status:** `400 Bad Request`
- **Example Payload:**
```json
{
  "notification_ids": [
    "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "this-is-not-a-uuid"
  ]
}
```
- **Error Response:**
```json
{
  "error": {
    "code": "INVALID_FORMAT",
    "message": "One or more notification IDs have an invalid format."
  }
}
```

**Notification Not Found**
- **Trigger:** Sending a valid UUID for a notification that does not exist or does not belong to the authenticated user.
- **HTTP Status:** `404 Not Found`
- **Example Payload:**
```json
{
  "notification_ids": [
    "00000000-0000-0000-0000-000000000000"
  ]
}
```
- **Error Response:**
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "One or more notifications could not be found."
  }
}
```

---

## 3. Specific Implementation Details

- **Push Notifications vs. In-App Notifications:** This API specification covers the in-app notification center. The actual delivery of real-time push notifications should be handled separately by a push notification service (like Firebase Cloud Messaging or Apple Push Notification Service). When a push is sent, a corresponding entry should be created in the database that is then served by this API.
- **Scalability:** The `mark-as-read` endpoint is designed to accept multiple IDs to reduce the number of network requests from the client, improving performance and user experience.
- **Data Action:** The `action_metadata` field provides flexibility for the client. For a report-related notification, the client can use the `report_id` to build a deep link that takes the user directly to the report detail screen.
