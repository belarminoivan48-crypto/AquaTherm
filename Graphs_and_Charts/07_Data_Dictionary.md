# Data Dictionary — AquaTherm

> **Revision Note:** Updated to reflect the finalized sensor configuration — **water temperature (DS18B20)** and **salinity** — in place of dissolved oxygen (DO), per the adviser's revision. Affected fields: `SENSOR_DEVICE.device_type`, `SENSOR_READING.do_value` → `temperature_value`, `SENSOR_READING.do_status` → `temperature_status`, and `NOTIFICATION_SETTING.do_threshold_min/max` → `temperature_threshold_min/max`.

## Table: USER

Stores the account and contact information of every fishpond owner (and admin) who logs into AquaTherm. This is the table alert messages ultimately resolve a contact number against.

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

Represents a physical brackish-water fishpond registered under a user's account — e.g., the study's partner fishpond in Barangay Tococ, Sto. Tomas, La Union.

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| fishpond_id | INT | - | PK, Auto Increment | Unique identifier for each fishpond |
| user_id | INT | - | FK → USER.user_id | Owner of the fishpond |
| fishpond_name | VARCHAR | 100 | NOT NULL | Name/label of the fishpond |
| location | VARCHAR | 150 | NOT NULL | Physical address/barangay location |
| area_size | FLOAT | - | NULLABLE | Size of the fishpond in hectares/sqm |
| date_registered | DATE | - | NOT NULL | Date the fishpond was registered in the system |

## Table: SENSOR_DEVICE

Represents a physical IoT sensor node (ESP32 microcontroller + DS18B20 temperature sensor + salinity/conductivity sensor) installed at a fishpond.

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| device_id | INT | - | PK, Auto Increment | Unique identifier for each IoT device |
| fishpond_id | INT | - | FK → FISHPOND.fishpond_id | Fishpond where the device is installed |
| device_name | VARCHAR | 50 | NOT NULL | Label of the device (e.g., "Pond A Sensor") |
| device_type | VARCHAR | 30 | NOT NULL | Type of sensor (Temperature, Salinity, Combined) |
| status | VARCHAR | 20 | DEFAULT 'Active' | Current device status (Active/Inactive/Faulty) |
| date_installed | DATE | - | NOT NULL | Date the device was installed |

## Table: SENSOR_READING

Stores each individual temperature/salinity reading transmitted by a sensor node, together with its computed Safe/Warning/Critical classification. This is the core operational table behind Objectives 1 and 2.

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| reading_id | INT | - | PK, Auto Increment | Unique identifier for each reading entry |
| device_id | INT | - | FK → SENSOR_DEVICE.device_id | Device that generated the reading |
| temperature_value | FLOAT | - | NOT NULL | Water temperature reading in degrees Celsius (°C), captured by the DS18B20 waterproof digital temperature sensor |
| salinity_value | FLOAT | - | NOT NULL | Salinity reading in ppt (parts per thousand), captured by the salinity/conductivity sensor |
| temperature_status | VARCHAR | 20 | NOT NULL | Computed status (Safe/Warning/Critical) |
| salinity_status | VARCHAR | 20 | NOT NULL | Computed status (Safe/Warning/Critical) |
| timestamp | DATETIME | - | NOT NULL | Date and time the reading was recorded |

## Table: ALERT

Stores a record for every Warning/Critical condition detected, addressed to the responsible user for SMS/push delivery.

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

Tracks each individual delivery attempt of an alert per channel (SMS or push), supporting the study's dual-channel alerting objective and its performance-efficiency evaluation.

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| log_id | INT | - | PK, Auto Increment | Unique identifier for each notification log entry |
| alert_id | INT | - | FK → ALERT.alert_id | Related alert record |
| channel | VARCHAR | 20 | NOT NULL | Delivery channel (SMS, Push Notification) |
| delivery_status | VARCHAR | 20 | NOT NULL | Delivery result (Sent/Failed/Pending) |
| date_sent | DATETIME | - | NOT NULL | Date and time the notification was sent |

## Table: NOTIFICATION_SETTING

Stores each user's configurable Safe/Warning/Critical threshold ranges and channel preferences, so classification logic (Process 2.0) never relies on hard-coded values.

| Field Name | Data Type | Size | Constraint | Description |
|---|---|---|---|---|
| setting_id | INT | - | PK, Auto Increment | Unique identifier for each setting entry |
| user_id | INT | - | FK → USER.user_id | User associated with the settings |
| sms_enabled | BOOLEAN | - | DEFAULT TRUE | Whether SMS alerts are enabled |
| push_enabled | BOOLEAN | - | DEFAULT TRUE | Whether push notifications are enabled |
| temperature_threshold_min | FLOAT | - | NOT NULL | Minimum safe water temperature threshold, in °C, set by user/admin |
| temperature_threshold_max | FLOAT | - | NOT NULL | Maximum safe water temperature threshold, in °C, set by user/admin |
| salinity_threshold_min | FLOAT | - | NOT NULL | Minimum safe salinity threshold, in ppt |
| salinity_threshold_max | FLOAT | - | NOT NULL | Maximum safe salinity threshold, in ppt |

## Explanation

The Data Dictionary is the authoritative, field-level reference for the database implied by the ERD — every field named in the DFD's data stores, the Structured English's `RETRIEVE`/`STORE` statements, and the Pseudocode's `SAVE_TO_DATABASE()` calls resolves to an exact row here (name, type, size, constraint, and meaning), which keeps all seven design documents internally consistent.

A few details worth highlighting for this revision:

- **`SENSOR_READING.temperature_value` and `salinity_value`** are the two numeric values the ESP32 sensor node transmits every cycle; keeping them as separate `FLOAT` columns (rather than one generic "reading" column) is what allows `temperature_status` and `salinity_status` to be classified and stored independently, exactly as described in Process 2.0.
- **`SENSOR_DEVICE.device_type`** now lists `Temperature`, `Salinity`, or `Combined` as valid values — `Combined` covers AquaTherm's actual sensor node, which houses both sensors on a single ESP32, while the separate options remain available if a future deployment splits sensing across multiple physical devices.
- **`NOTIFICATION_SETTING.temperature_threshold_min/max`** exist specifically so the Safe/Warning/Critical boundaries are *configurable per user* rather than fixed constants in code — operationally, this is where the baseline readings gathered at the partner fishpond (per the study's descriptive research component) and the literature-based milkfish temperature/salinity tolerances get translated into the actual values the system classifies against.
- Foreign keys throughout (`FK →`) enforce the same one-to-many relationships drawn in the ERD — e.g., `SENSOR_READING.device_id → SENSOR_DEVICE.device_id` ensures every reading is traceable to the exact device and, transitively, the exact fishpond that produced it.
