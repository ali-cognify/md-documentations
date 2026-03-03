# LJI API Documentation - Summary Table

| Screen | API Name | Request Data | Response Data |
| :--- | :--- | :--- | :--- |
| **Splash** | Check App Version | <ul><li>platform</li><li>current_version</li></ul> | <ul><li>update_status</li><li>latest_version</li><li>download_url</li><li>message</li><li>maintenance info</li></ul> |
| **Login** | Login with Phone & Password | <ul><li>phone_number</li><li>country_code</li><li>password</li><li>remember_me</li></ul> | <ul><li>access_token</li><li>refresh_token</li><li>token_type</li><li>expires_in</li><li>user profile data</li></ul> |
| **Login** | Social Login | <ul><li>provider</li><li>id_token</li><li>device_id</li></ul> | <ul><li>access_token</li><li>refresh_token</li><li>token_type</li><li>user profile data</li></ul> |
| **Login** | Guest Login | <i>None</i> | <ul><li>access_token (restricted)</li><li>limited user data</li></ul> |
| **Register** | Register Account | <ul><li>country_code</li><li>phone_number</li><li>full_name</li><li>password</li><li>password_confirmation</li><li>agree_terms</li></ul> | <ul><li>phone_number</li><li>otp_metadata (expiry)</li></ul> |
| **Register** | Send/Resend OTP | <ul><li>country_code</li><li>phone_number</li></ul> | <ul><li>otp_metadata (expiry)</li></ul> |
| **Register** | Verify OTP | <ul><li>country_code</li><li>phone_number</li><li>otp_code</li></ul> | <ul><li>access_token</li><li>user profile data</li></ul> |
| **Home** | Get User Points | <i>None</i> | <ul><li>current_points</li><li>expiring_points (amount & date)</li></ul> |
| **Home** | Get Banners | <i>None</i> | <ul><li>list of banners (image, action URL, etc)</li></ul> |
| **Notification** | Get Notification List | <ul><li>page</li><li>limit</li></ul> | <ul><li>pagination info</li><li>list of notifications</li></ul> |
| **Notification** | Get Notification Status | <i>None</i> | <ul><li>unread_count</li></ul> |
| **Notification** | Mark as Read | <ul><li>notification_ids</li></ul> | <i>No Content</i> |
| **History** | Get History List | <ul><li>type</li><li>status</li><li>date range</li><li>pagination</li></ul> | <ul><li>pagination info</li><li>list of activities (scan/redeem)</li></ul> |
| **History** | Get History Detail | <ul><li>id (path)</li></ul> | <ul><li>activity details (polymorphic)</li><li>transaction/point details</li></ul> |
| **Scan** | Verify QR Code | <ul><li>qr_code_data</li></ul> | <ul><li>scan status (VALID/INVALID)</li><li>transaction/reward details</li></ul> |
| **My QR** | Generate Redeem QR | <i>None</i> | <ul><li>qr_data (token)</li><li>expiry</li><li>WebSocket listen topic</li></ul> |
| **Profile** | Get Aggregated Profile | <i>None</i> | <ul><li>profile info</li><li>identity status</li><li>all user settings</li></ul> |
| **Profile** | Upload ID Card (OCR) | <ul><li>image (file)</li></ul> | <ul><li>extracted identity data</li></ul> |
| **Profile** | Update Profile | <ul><li>full_name</li><li>email</li></ul> | <ul><li>updated profile object</li></ul> |
| **Profile** | Update Settings | <ul><li>notification toggles</li><li>language preference</li></ul> | <i>No Content</i> |
| **Profile** | Change Security PIN | <ul><li>current_password</li><li>new_pin</li></ul> | <i>No Content</i> |
| **Static Content** | Get Static Page | <ul><li>slug</li><li>language</li></ul> | <ul><li>title</li><li>body (markdown/html)</li></ul> |
| **Store Report** | Create New Report | <ul><li>tax_object info</li><li>address</li><li>images</li><li>remarks</li></ul> | <ul><li>report_id</li><li>estimated follow-up time</li></ul> |
| **Store Report** | Get Report List | <ul><li>status filter</li><li>sorting</li><li>pagination</li></ul> | <ul><li>pagination info</li><li>list of reports</li></ul> |
| **Store Report** | Get Report Detail | <ul><li>report_number (path)</li></ul> | <ul><li>report details</li><li>approval/rejection info</li><li>images</li></ul> |
| **Store Report** | Get Master Data | <i>None</i> | <ul><li>tax_objects list</li><li>report_categories list</li></ul> |
