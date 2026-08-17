# Pseudocode — AquaTherm

> **Revision Note:** Updated to reflect the finalized sensor configuration — **water temperature (DS18B20)** and **salinity** — in place of dissolved oxygen (DO), per the adviser's revision. Threshold constants below are **illustrative starting values drawn from milkfish (bangus) aquaculture literature** (optimal growth range of roughly 25–32°C and brackish-pond salinity of roughly 10–30 ppt); the actual operating values used during testing are calibrated using the baseline readings gathered at the partner fishpond in Barangay Tococ and are stored as configurable values in the `NOTIFICATION_SETTING` table rather than hard-coded in production.

## Main Monitoring Loop (Microcontroller/ESP32 Side)

```
START

DEFINE TEMP_MIN = 25.0        // °C  - minimum safe water temperature (illustrative baseline)
DEFINE TEMP_MAX = 32.0        // °C  - maximum safe water temperature (illustrative baseline)
DEFINE SALINITY_MIN = 10      // ppt - minimum safe salinity
DEFINE SALINITY_MAX = 30      // ppt - maximum safe salinity
DEFINE READ_INTERVAL = 300    // seconds (5-minute interval)

LOOP FOREVER
    tempValue = READ_SENSOR(TEMP_SENSOR_PIN)        // DS18B20 waterproof digital temperature sensor
    salinityValue = READ_SENSOR(SALINITY_SENSOR_PIN) // Salinity/conductivity sensor
    currentTime = GET_TIMESTAMP()

    IF WIFI_CONNECTED() THEN
        SEND_TO_SERVER(tempValue, salinityValue, currentTime)
        FLUSH_BUFFERED_READINGS()   // send any readings saved during a prior outage
    ELSE
        BUFFER_LOCALLY(tempValue, salinityValue, currentTime)  // fallback while Wi-Fi is unavailable
    END IF

    WAIT(READ_INTERVAL)
END LOOP

END
```

## Server-Side Processing and Alert Logic

```
FUNCTION processReading(tempValue, salinityValue, timestamp)

    tempStatus = evaluateTemperature(tempValue)
    salinityStatus = evaluateSalinity(salinityValue)

    SAVE_TO_DATABASE(tempValue, salinityValue, tempStatus, salinityStatus, timestamp)

    IF tempStatus == "CRITICAL" OR salinityStatus == "CRITICAL" THEN
        triggerAlert("CRITICAL", tempValue, salinityValue)
    ELSE IF tempStatus == "WARNING" OR salinityStatus == "WARNING" THEN
        triggerAlert("WARNING", tempValue, salinityValue)
    END IF

END FUNCTION


FUNCTION evaluateTemperature(tempValue)
    IF tempValue < TEMP_MIN THEN
        RETURN "CRITICAL"
    ELSE IF tempValue < (TEMP_MIN + 2.0) THEN
        RETURN "WARNING"
    ELSE IF tempValue > TEMP_MAX THEN
        RETURN "CRITICAL"
    ELSE IF tempValue > (TEMP_MAX - 2.0) THEN
        RETURN "WARNING"
    ELSE
        RETURN "SAFE"
    END IF
END FUNCTION


FUNCTION evaluateSalinity(salinityValue)
    IF salinityValue < SALINITY_MIN OR salinityValue > SALINITY_MAX THEN
        RETURN "CRITICAL"
    ELSE IF salinityValue < (SALINITY_MIN + 2) OR salinityValue > (SALINITY_MAX - 2) THEN
        RETURN "WARNING"
    ELSE
        RETURN "SAFE"
    END IF
END FUNCTION


FUNCTION triggerAlert(level, tempValue, salinityValue)
    message = BUILD_MESSAGE(level, tempValue, salinityValue)
    contactNumber = GET_USER_CONTACT()

    SEND_SMS(contactNumber, message)
    SEND_PUSH_NOTIFICATION(message)
    LOG_ALERT(message, level, CURRENT_TIMESTAMP())
END FUNCTION
```

## Explanation

The pseudocode expresses the same logic as the Structured English document, but at a level detailed enough to guide actual firmware/backend implementation, and splits cleanly along the system's two physical halves:

**Main Monitoring Loop (ESP32 side).** This runs directly on the IoT sensor node described in the study's first objective — an ESP32 microcontroller paired with the DS18B20 waterproof temperature sensor and the salinity/conductivity sensor, housed in an IP65-rated enclosure and powered by a solar panel with a rechargeable battery. Every `READ_INTERVAL` (5 minutes), it polls both sensors and timestamps the reading. A key addition in this revision is the **Wi-Fi fallback branch**: because the fishpond is a field deployment without a fixed, always-on connection, the loop checks connectivity before transmitting — if Wi-Fi is available it sends the reading (and flushes anything queued from an earlier outage); if not, it buffers the reading locally instead of losing it. This directly implements the "local buffering built in as a fallback whenever the Wi-Fi connection is temporarily unavailable" behavior specified in the study's Materials and Procedures.

**Server-Side Processing and Alert Logic.** Once a reading reaches the server, `processReading()` evaluates temperature and salinity independently through two dedicated functions:
- `evaluateTemperature()` flags `CRITICAL` once the reading falls outside the safe band (`TEMP_MIN`–`TEMP_MAX`), and `WARNING` when it is still safe but within a 2°C buffer of either boundary — giving the owner advance notice before a reading actually becomes unsafe.
- `evaluateSalinity()` follows the same safe/warning/critical logic using a comparable buffer around the salinity range.

Whenever either function returns `WARNING` or `CRITICAL`, `triggerAlert()` builds a message describing the severity and the offending value(s), retrieves the owner's contact number, and dispatches it through **both** SMS and push notification before logging the alert — mirroring Process 3.0 in the Structured English and completing the study's third objective (the dual-channel alert module).

All numeric thresholds are written as named constants rather than embedded directly in the conditional logic, so that they can later be sourced from the `NOTIFICATION_SETTING` table (see the Data Dictionary) and adjusted without changing the underlying algorithm — consistent with the thesis's design intent that threshold ranges be configurable rather than fixed.
