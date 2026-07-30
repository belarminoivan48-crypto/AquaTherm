# Pseudocode — AquaTherm

## Main Monitoring Loop (Microcontroller/ESP32 Side)

```
START

DEFINE DO_MIN = 3.0        // mg/L - minimum safe dissolved oxygen
DEFINE DO_MAX = 8.0        // mg/L - maximum safe dissolved oxygen
DEFINE SALINITY_MIN = 10   // ppt - minimum safe salinity
DEFINE SALINITY_MAX = 35   // ppt - maximum safe salinity
DEFINE READ_INTERVAL = 300 // seconds (5-minute interval)

LOOP FOREVER
    doValue = READ_SENSOR(DO_SENSOR_PIN)
    salinityValue = READ_SENSOR(SALINITY_SENSOR_PIN)
    currentTime = GET_TIMESTAMP()

    SEND_TO_SERVER(doValue, salinityValue, currentTime)

    WAIT(READ_INTERVAL)
END LOOP

END
```

## Server-Side Processing and Alert Logic

```
FUNCTION processReading(doValue, salinityValue, timestamp)

    doStatus = evaluateDO(doValue)
    salinityStatus = evaluateSalinity(salinityValue)

    SAVE_TO_DATABASE(doValue, salinityValue, doStatus, salinityStatus, timestamp)

    IF doStatus == "CRITICAL" OR salinityStatus == "CRITICAL" THEN
        triggerAlert("CRITICAL", doValue, salinityValue)
    ELSE IF doStatus == "WARNING" OR salinityStatus == "WARNING" THEN
        triggerAlert("WARNING", doValue, salinityValue)
    END IF

END FUNCTION


FUNCTION evaluateDO(doValue)
    IF doValue < DO_MIN THEN
        RETURN "CRITICAL"
    ELSE IF doValue < (DO_MIN + 1.0) THEN
        RETURN "WARNING"
    ELSE IF doValue > DO_MAX THEN
        RETURN "CRITICAL"
    ELSE
        RETURN "SAFE"
    END IF
END FUNCTION


FUNCTION evaluateSalinity(salinityValue)
    IF salinityValue < SALINITY_MIN OR salinityValue > SALINITY_MAX THEN
        RETURN "CRITICAL"
    ELSE IF salinityValue < (SALINITY_MIN + 2) THEN
        RETURN "WARNING"
    ELSE
        RETURN "SAFE"
    END IF
END FUNCTION


FUNCTION triggerAlert(level, doValue, salinityValue)
    message = BUILD_MESSAGE(level, doValue, salinityValue)
    contactNumber = GET_USER_CONTACT()

    SEND_SMS(contactNumber, message)
    SEND_PUSH_NOTIFICATION(message)
    LOG_ALERT(message, level, CURRENT_TIMESTAMP())
END FUNCTION
```
