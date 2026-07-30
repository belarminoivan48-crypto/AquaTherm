# Structured English — AquaTherm

## Process 2.0 – Process and Analyze Water Quality Data

```
BEGIN
    READ DO_reading FROM sensor
    READ Salinity_reading FROM sensor

    IF DO_reading IS NULL OR Salinity_reading IS NULL THEN
        LOG error "Invalid sensor data"
        EXIT process
    ENDIF

    RETRIEVE DO_min, DO_max, Salinity_min, Salinity_max FROM ThresholdSettings

    IF DO_reading < DO_min OR DO_reading > DO_max THEN
        SET DO_status = "Critical"
    ELSE IF DO_reading is within warning range THEN
        SET DO_status = "Warning"
    ELSE
        SET DO_status = "Safe"
    ENDIF

    IF Salinity_reading < Salinity_min OR Salinity_reading > Salinity_max THEN
        SET Salinity_status = "Critical"
    ELSE IF Salinity_reading is within warning range THEN
        SET Salinity_status = "Warning"
    ELSE
        SET Salinity_status = "Safe"
    ENDIF

    STORE DO_reading, Salinity_reading, DO_status, Salinity_status, timestamp TO SensorReadingsDB

    IF DO_status = "Critical" OR Salinity_status = "Critical" 
       OR DO_status = "Warning" OR Salinity_status = "Warning" THEN
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
