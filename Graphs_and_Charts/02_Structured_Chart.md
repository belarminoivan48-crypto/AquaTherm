# Structured Chart — AquaTherm

> **Revision Note:** Updated to reflect the finalized sensor configuration — **water temperature (DS18B20)** and **salinity** — in place of dissolved oxygen (DO), per the adviser's revision.

```mermaid
flowchart TD
    Main[AquaTherm Main Module]

    Main --> A
    Main --> B
    Main --> C
    Main --> D
    Main --> E

    subgraph SG_A [Sensor Data Module]
        direction TB
        A[Sensor Data Module]
        A1[Read Water Temperature Sensor Value]
        A2[Read Salinity Sensor Value]
        A3[Validate Sensor Input]
        A4[Store Reading to Database]
        A --> A1 --> A2 --> A3 --> A4
    end

    subgraph SG_B [Water Quality Monitoring Module]
        direction TB
        B[Water Quality Monitoring Module]
        B1[Retrieve Threshold Settings]
        B2[Compare Reading vs Threshold]
        B3[Compute Water Quality Status]
        B4[Update Status Record]
        B --> B1 --> B2 --> B3 --> B4
    end

    subgraph SG_C [Alert & Notification Module]
        direction TB
        C[Alert & Notification Module]
        C1[Detect Unsafe Condition]
        C2[Compose Alert Message]
        C3[Send SMS/Push Notification]
        C4[Log Alert Record]
        C --> C1 --> C2 --> C3 --> C4
    end

    subgraph SG_D [Report Generation Module]
        direction TB
        D[Report Generation Module]
        D1[Retrieve Historical Data]
        D2[Generate Chart/Graph]
        D3[Export Report - PDF/CSV]
        D --> D1 --> D2 --> D3
    end

    subgraph SG_E [User Account Module]
        direction TB
        E[User Account Module]
        E1[Login/Authenticate]
        E2[Register Account]
        E3[Manage Profile]
        E4[Set Notification Preferences]
        E --> E1 --> E2 --> E3 --> E4
    end
```

## Module Descriptions

| Module | Function |
|---|---|
| Sensor Data Module | Handles reading, validating, and storing raw water temperature (via the DS18B20 waterproof digital sensor) and salinity (via the salinity/conductivity sensor) values from the ESP32-based IoT sensor node |
| Water Quality Monitoring Module | Compares sensor readings against the configured Safe/Warning/Critical thresholds and computes the current water quality status for both parameters |
| Alert & Notification Module | Detects unsafe conditions and triggers dual-channel (SMS + push) notifications to the fishpond owner |
| Report Generation Module | Produces historical charts, graphs, and exportable reports from stored sensor data |
| User Account Module | Manages login, registration, and account-level settings, including notification preferences |

## Explanation

The Structured Chart shows AquaTherm's software organized as a top-down hierarchy of callable modules, independent of timing or sequence — it answers *what functional units make up the system and how do they relate to one another*, which complements the DFD's focus on data movement.

- **Main Module** is the top-level control module that invokes each of the five subordinate modules as needed; it does not itself perform processing.
- **Sensor Data Module (A)** is the entry point for hardware data. Its four child steps run in sequence: read the temperature value from the DS18B20 probe, read the salinity value from the conductivity sensor, validate both readings for completeness/sanity, then persist them to the `SENSOR_READING` table — this directly implements the study's first objective (the ESP32 sensor node).
- **Water Quality Monitoring Module (B)** takes over once a reading is stored: it pulls the current threshold configuration (`NOTIFICATION_SETTING`), compares the new reading against it, computes the Safe/Warning/Critical classification, and updates the record — implementing the second objective (real-time classification on the mobile interface).
- **Alert & Notification Module (C)** is invoked only when Module B detects an unsafe condition. It composes a message describing the parameter and severity, dispatches it through both SMS and push notification channels, and logs the alert — implementing the third objective (dual-channel alerting).
- **Report Generation Module (D)** is invoked on-demand by the owner to turn historical readings and alert logs into charts and exportable reports.
- **User Account Module (E)** handles authentication and account/notification-preference management and is largely independent of the sensor pipeline, so it is drawn as a sibling rather than a downstream module.

Each subgraph's internal chain (e.g., A1 → A2 → A3 → A4) represents the *typical* call order for that module, while the top-level `Main --> A/B/C/D/E` fan-out shows that Main can invoke any module depending on the triggering event (a new sensor reading, an owner request, a scheduled report, etc.).
