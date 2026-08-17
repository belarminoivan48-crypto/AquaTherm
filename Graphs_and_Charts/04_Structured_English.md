# Structured English — AquaTherm

> **Revision Note:** Updated to reflect the finalized sensor configuration — **water temperature (DS18B20)** and **salinity** — in place of dissolved oxygen (DO), per the adviser's revision.

## Process 2.0 – Process and Analyze Water Quality Data

```
BEGIN
    READ Temperature_reading FROM sensor
    READ Salinity_reading FROM sensor

    IF Temperature_reading IS NULL OR Salinity_reading IS NULL THEN
        LOG error "Invalid sensor data"
        EXIT process
    ENDIF

    RETRIEVE Temperature_min, Temperature_max, Salinity_min, Salinity_max FROM ThresholdSettings

    IF Temperature_reading < Temperature_min OR Temperature_reading > Temperature_max THEN
        SET Temperature_status = "Critical"
    ELSE IF Temperature_reading is within warning range THEN
        SET Temperature_status = "Warning"
    ELSE
        SET Temperature_status = "Safe"
    ENDIF

    IF Salinity_reading < Salinity_min OR Salinity_reading > Salinity_max THEN
        SET Salinity_status = "Critical"
    ELSE IF Salinity_reading is within warning range THEN
        SET Salinity_status = "Warning"
    ELSE
        SET Salinity_status = "Safe"
    ENDIF

    STORE Temperature_reading, Salinity_reading, Temperature_status, Salinity_status, timestamp TO SensorReadingsDB

    IF Temperature_status = "Critical" OR Salinity_status = "Critical" 
       OR Temperature_status = "Warning" OR Salinity_status = "Warning" THEN
        CALL Process_3.0_GenerateAlert
    ENDIF
END
```

## Process 3.0 – Generate Alerts and Notifications

```
BEGIN
    RECEIVE water_quality_status FROM Process 2.0

    IF water_quality_status = "Critical" THEN
        SET message = "CRITICAL: Water quality unsafe. Immediate action required."
    ELSE IF water_quality_status = "Warning" THEN
        SET message = "WARNING: Water quality approaching unsafe levels."
    ENDIF

    RETRIEVE registered_contact_number FROM UserAccountsDB

    SEND SMS(message, registered_contact_number)
    SEND push_notification(message) TO mobile app

    STORE alert_message, timestamp, status TO AlertLogsDB
END
```

## Explanation

Structured English translates the DFD's Process 2.0 and Process 3.0 into precise, program-like statements using a controlled vocabulary (BEGIN/END, IF/ELSE, RETRIEVE, STORE) — a middle ground between a narrative description and actual source code, meant to be unambiguous to both technical and non-technical readers (e.g., the thesis panel).

**Process 2.0 – Process and Analyze Water Quality Data**
1. The process first reads the two values the sensor node produces every cycle: `Temperature_reading` (°C, from the DS18B20 sensor) and `Salinity_reading` (ppt, from the salinity/conductivity sensor).
2. A null-check guards against incomplete or failed sensor transmissions — if either reading is missing, the process logs an error and exits rather than storing bad data or triggering a false alert.
3. It then retrieves the current Safe/Warning/Critical boundary values from `ThresholdSettings` (the `NOTIFICATION_SETTING` table), which, per the study's methodology, are configured using both baseline readings gathered at the partner fishpond and the temperature/salinity tolerances established in the reviewed literature.
4. Each parameter is evaluated **independently** against its own thresholds, producing a `Temperature_status` and a `Salinity_status` of `Safe`, `Warning`, or `Critical`.
5. Both raw readings and both computed statuses are stored together with a timestamp in the `SensorReadingsDB` — this is the persisted record later shown on the mobile interface and used for report generation.
6. If **either** parameter resolves to `Warning` or `Critical`, Process 2.0 hands off to Process 3.0, since an unsafe condition in temperature alone (e.g., a sudden heat spike) or salinity alone (e.g., after heavy rainfall) is enough to put the milkfish stock at risk.

**Process 3.0 – Generate Alerts and Notifications**
1. This process receives the water quality status passed from Process 2.0 and composes a message whose wording depends on severity — a more urgent tone for `Critical`, a cautionary tone for `Warning`.
2. It looks up the fishpond owner's registered contact number from `UserAccountsDB` so the alert reaches the correct person.
3. It sends the message through **both** channels — SMS and mobile push notification — fulfilling the study's dual-channel alerting objective, which exists specifically so the owner is more likely to receive the warning in time to act (e.g., switching on an aerator or adjusting stocking density) even if one channel fails or is delayed.
4. Finally, it logs the alert message, timestamp, and status to `AlertLogsDB` for the historical alert record used later in Process 4.0 (Manage Records & Reports).
