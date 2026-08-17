# HIPO (Hierarchy Input Process Output) Diagram — AquaTherm

> **Revision Note:** Updated to reflect the finalized sensor configuration — **water temperature (DS18B20)** and **salinity** — in place of dissolved oxygen (DO), per the adviser's revision.

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
        P1a[1.1 Read Water Temperature Value]
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
| 1.0 Collect Sensor Data | Digital signal from the waterproof DS18B20 temperature sensor and the salinity/conductivity sensor, connected to the ESP32 microcontroller | Reads and converts sensor signals into usable numeric values at fixed intervals | Raw water temperature (°C) and salinity (ppt) readings |
| 2.0 Process Water Quality Data | Raw sensor readings, threshold settings | Validates data, compares to safe/unsafe threshold ranges, computes status | Water quality status (Safe / Warning / Critical) for temperature and salinity |
| 3.0 Generate Alerts | Water quality status = Warning/Critical | Composes alert message and sends via SMS/push notification | Alert notification sent to fishpond owner, alert log entry |
| 4.0 Manage Reports | Historical sensor and alert data | Aggregates data into charts/graphs, formats exportable reports | Line/bar charts, PDF/CSV report file |
| 5.0 Manage User Accounts | Username, password, profile details | Authenticates login, manages registration and profile updates | Authenticated session, updated user profile |

## Explanation

The HIPO diagram documents AquaTherm at two levels simultaneously: the **Hierarchy Chart** (visual, top-down decomposition of processes) and the **IPO Table** (textual detail of what goes into, happens inside, and comes out of each process). Together they satisfy the "quick design" phase of the Prototyping model described in the study's methodology, giving the researchers a shared overview before building the actual prototype.

- **1.0 Collect Sensor Data** corresponds to the physical IoT sensor node built around the ESP32 microcontroller. Its two sub-functions, **1.1 Read Water Temperature Value** and **1.2 Read Salinity Value**, map directly onto the two sensors named in the study's first objective — the DS18B20 waterproof digital temperature sensor and the salinity/conductivity sensor — each polled at a fixed interval and transmitted to the server over Wi-Fi (with local buffering as a fallback if the connection drops).
- **2.0 Process Water Quality Data** is where incoming values are validated, checked against the Safe/Warning/Critical threshold ranges configured in the system, and classified — the same logic detailed further in the Structured English and Pseudocode documents.
- **3.0 Generate Alerts** only executes when Process 2.0 flags a Warning or Critical status, and is responsible for the study's third objective: dual-channel (SMS + push notification) alerting to the fishpond owner.
- **4.0 Manage Reports** lets the owner review historical trends of temperature and salinity, and export them, supporting long-term recordkeeping beyond real-time alerting.
- **5.0 Manage User Accounts** covers authentication and profile management so that readings, alerts, and reports are always tied to the correct fishpond owner's account.

Reading the IPO table alongside the hierarchy chart clarifies not just *which* modules exist, but exactly *what data crosses each module's boundary* — which is what distinguishes a HIPO diagram from a plain structured/hierarchy chart.
