# HIPO (Hierarchy Input Process Output) Diagram — AquaTherm

## Hierarchy Chart

```mermaid
flowchart TD
    Main[0.0 AquaTherm System]

    Main --> P1
    Main --> P2
    Main --> P3
    Main --> P4
    Main --> P5

    subgraph SG1 [1.0 Collect Sensor Data]
        direction TB
        P1[1.0 Collect Sensor Data]
        P1a[1.1 Read DO Value]
        P1b[1.2 Read Salinity Value]
        P1 --> P1a --> P1b
    end

    subgraph SG2 [2.0 Process Water Quality Data]
        direction TB
        P2[2.0 Process Water Quality Data]
        P2a[2.1 Validate Data]
        P2b[2.2 Compare to Threshold]
        P2c[2.3 Compute Status]
        P2 --> P2a --> P2b --> P2c
    end

    subgraph SG3 [3.0 Generate Alerts]
        direction TB
        P3[3.0 Generate Alerts]
        P3a[3.1 Detect Unsafe Condition]
        P3b[3.2 Send SMS/Push Alert]
        P3 --> P3a --> P3b
    end

    subgraph SG4 [4.0 Manage Reports]
        direction TB
        P4[4.0 Manage Reports]
        P4a[4.1 Generate Chart]
        P4b[4.2 Export Report]
        P4 --> P4a --> P4b
    end

    subgraph SG5 [5.0 Manage User Accounts]
        direction TB
        P5[5.0 Manage User Accounts]
        P5a[5.1 Login/Register]
        P5b[5.2 Manage Profile]
        P5 --> P5a --> P5b
    end
```

## IPO (Input-Process-Output) Table

| Process | Input | Process Description | Output |
|---|---|---|---|
| 1.0 Collect Sensor Data | Raw analog/digital signal from DO and salinity sensors | Reads and converts sensor signals into usable numeric values | Raw DO (mg/L) and Salinity (ppt) readings |
| 2.0 Process Water Quality Data | Raw sensor readings, threshold settings | Validates data, compares to safe/unsafe threshold ranges, computes status | Water quality status (Safe / Warning / Critical) |
| 3.0 Generate Alerts | Water quality status = Warning/Critical | Composes alert message and sends via SMS/push notification | Alert notification sent to fishpond owner, alert log entry |
| 4.0 Manage Reports | Historical sensor and alert data | Aggregates data into charts/graphs, formats exportable reports | Line/bar charts, PDF/CSV report file |
| 5.0 Manage User Accounts | Username, password, profile details | Authenticates login, manages registration and profile updates | Authenticated session, updated user profile |
