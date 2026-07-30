# Data Dictionary — AquaTherm

## Table: USER

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| user_id | INT | - | PK, Auto Increment | Unique identifier for each user |
| full_name | VARCHAR | 100 | NOT NULL | Full name of the fishpond owner/user |
| contact_number | VARCHAR | 15 | NOT NULL | Mobile number used for SMS alerts |
| email | VARCHAR | 100 | UNIQUE | Email address of the user |
| username | VARCHAR | 50 | UNIQUE, NOT NULL | Login username |
| password | VARCHAR | 255 | NOT NULL | Encrypted/hashed password |
| role | VARCHAR | 20 | DEFAULT 'Owner' | User role (Owner, Admin) |

## Table: FISHPOND

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| fishpond_id | INT | - | PK, Auto Increment | Unique identifier for each fishpond |
| user_id | INT | - | FK → USER.user_id | Owner of the fishpond |
| fishpond_name | VARCHAR | 100 | NOT NULL | Name/label of the fishpond |
| location | VARCHAR | 150 | NOT NULL | Physical address/barangay location |
| area_size | FLOAT | - | NULLABLE | Size of the fishpond in hectares/sqm |
| date_registered | DATE | - | NOT NULL | Date the fishpond was registered in the system |

## Table: SENSOR_DEVICE

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| device_id | INT | - | PK, Auto Increment | Unique identifier for each IoT device |
| fishpond_id | INT | - | FK → FISHPOND.fishpond_id | Fishpond where the device is installed |
| device_name | VARCHAR | 50 | NOT NULL | Label of the device (e.g., "Pond A Sensor") |
| device_type | VARCHAR | 30 | NOT NULL | Type of sensor (DO, Salinity, Combined) |
| status | VARCHAR | 20 | DEFAULT 'Active' | Current device status (Active/Inactive/Faulty) |
| date_installed | DATE | - | NOT NULL | Date the device was installed |

## Table: SENSOR_READING

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| reading_id | INT | - | PK, Auto Increment | Unique identifier for each reading entry |
| device_id | INT | - | FK → SENSOR_DEVICE.device_id | Device that generated the reading |
| do_value | FLOAT | - | NOT NULL | Dissolved oxygen reading in mg/L |
| salinity_value | FLOAT | - | NOT NULL | Salinity reading in ppt (parts per thousand) |
| do_status | VARCHAR | 20 | NOT NULL | Computed status (Safe/Warning/Critical) |
| salinity_status | VARCHAR | 20 | NOT NULL | Computed status (Safe/Warning/Critical) |
| timestamp | DATETIME | - | NOT NULL | Date and time the reading was recorded |

## Table: ALERT

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| alert_id | INT | - | PK, Auto Increment | Unique identifier for each alert |
| reading_id | INT | - | FK → SENSOR_READING.reading_id | Reading that triggered the alert |
| user_id | INT | - | FK → USER.user_id | User who receives the alert |
| alert_level | VARCHAR | 20 | NOT NULL | Severity level (Warning/Critical) |
| message | VARCHAR | 255 | NOT NULL | Alert message content |
| date_triggered | DATETIME | - | NOT NULL | Date and time the alert was generated |
| status | VARCHAR | 20 | DEFAULT 'Unread' | Alert status (Unread/Read/Resolved) |

## Table: NOTIFICATION_LOG

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| log_id | INT | - | PK, Auto Increment | Unique identifier for each notification log entry |
| alert_id | INT | - | FK → ALERT.alert_id | Related alert record |
| channel | VARCHAR | 20 | NOT NULL | Delivery channel (SMS, Push Notification) |
| delivery_status | VARCHAR | 20 | NOT NULL | Delivery result (Sent/Failed/Pending) |
| date_sent | DATETIME | - | NOT NULL | Date and time the notification was sent |

## Table: NOTIFICATION_SETTING

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| setting_id | INT | - | PK, Auto Increment | Unique identifier for each setting entry |
| user_id | INT | - | FK → USER.user_id | User associated with the settings |
| sms_enabled | BOOLEAN | - | DEFAULT TRUE | Whether SMS alerts are enabled |
| push_enabled | BOOLEAN | - | DEFAULT TRUE | Whether push notifications are enabled |
| do_threshold_min | FLOAT | - | NOT NULL | Minimum safe DO threshold set by user/admin |
| do_threshold_max | FLOAT | - | NOT NULL | Maximum safe DO threshold set by user/admin |
| salinity_threshold_min | FLOAT | - | NOT NULL | Minimum safe salinity threshold |
| salinity_threshold_max | FLOAT | - | NOT NULL | Maximum safe salinity threshold |
