# Structured Chart — AquaTherm

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
        A1[Read DO Sensor Value]
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
| Sensor Data Module | Handles reading, validating, and storing raw DO and salinity values from IoT sensors |
| Water Quality Monitoring Module | Compares sensor readings against defined thresholds and computes the current water quality status |
| Alert & Notification Module | Detects unsafe conditions and triggers SMS/push notifications to the fishpond owner |
| Report Generation Module | Produces historical charts, graphs, and exportable reports from stored sensor data |
| User Account Module | Manages login, registration, and account-level settings |
