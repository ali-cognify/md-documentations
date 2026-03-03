# LJI API Documentation - Home

## 1. API Specification

### A. Get User Points

- **Method & Path:** `GET /v1/home/points`
- **Description:** Retrieves the current point balance and information about points nearing expiration for the authenticated user.

**Success Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "current_points": 1250,
    "expiring_points": {
      "amount": 250,
      "expiry_date": "2026-12-31T23:59:59Z"
    }
  }
}
```

**Success Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `status` | String | Indicates the status of the response. |
| `data.current_points` | Integer | The total amount of points currently available to the user. |
| `data.expiring_points.amount` | Integer | The amount of points that will expire soon. |
| `data.expiring_points.expiry_date` | String | ISO 8601 timestamp of when the expiring points will lapse. |


### B. Get Banners

- **Method & Path:** `GET /v1/home/banners`
- **Description:** Retrieves a list of active banners for the carousel on the home screen.

**Success Response (200 OK)**

```json
{
  "status": "success",
  "data": [
    {
      "id": "banner_001",
      "title": "Promo Akhir Tahun",
      "image_url": "https://cdn.app.com/banners/promo_1.jpg",
      "action_type": "WEBVIEW",
      "action_url": "https://app.com/articles/promo-detail"
    },
    {
      "id": "banner_002",
      "title": "Tutorial Lapor Pajak",
      "image_url": "https://cdn.app.com/banners/tutorial_1.jpg",
      "action_type": "CMS_ARTICLE",
      "action_url": "https://app.com/api/v1/cms/articles/123"
    }
  ]
}
```

**Success Response Field Description**

| Field | Type | Description |
| --- | --- | --- |
| `status` | String | Indicates the status of the response. |
| `data[].id` | String | Unique identifier for the banner. |
| `data[].title` | String | Title/Alternative text for the banner. |
| `data[].image_url` | String | Public URL of the banner image. |
| `data[].action_type` | Enum | Type of action when clicked (e.g., `WEBVIEW`, `CMS_ARTICLE`, `INTERNAL_LINK`). |
| `data[].action_url` | String | The URL or internal path to navigate to when clicked. |

---

## 2. Validations

### A. Authentication Error
- **Trigger:** Accessing the endpoint without a valid Bearer token.
- **HTTP Status:** `401 Unauthorized`
- **Error Response:**
```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Sesi berakhir. Silakan login kembali."
  }
}
```

### B. Server Error
- **Trigger:** Database or external CMS service failure.
- **HTTP Status:** `500 Internal Server Error`
- **Error Response:**
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Ada kendala sistem saat memuat data."
  }
}
```

---

## 3. Specific Implementation Details

- **Caching:** The banner list should be cached for a reasonable duration (e.g., 5-10 minutes) on the client side to improve load speed, or the server should provide ETag headers.
- **Point Expiry Calculation:** The backend should prioritize points with the earliest expiration date when user spends points (FIFO - First In First Out).
- **Banner Targeting:** The banner API could optionally support query parameters for targeting (e.g., `?location=jakarta`) if future requirements dictate region-specific promos.
- **Image Optimization:** Banners should be delivered in optimized formats (like WebP) and appropriate resolutions for mobile screens.
