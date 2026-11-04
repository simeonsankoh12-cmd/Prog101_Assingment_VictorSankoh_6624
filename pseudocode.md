# Pseudocode: Flood Early Warning and Community Alert System

```text
PROGRAM FloodEarlyWarningSystem

// ---------- CONSTANTS ----------
CONSTANT WATCH_RAIN   = 50      // mm in 24 hours
CONSTANT WARNING_RAIN = 100
CONSTANT DANGER_RAIN  = 150
CONSTANT WATCH_RIVER   = 2.0    // metres
CONSTANT WARNING_RIVER = 3.0
CONSTANT DANGER_RIVER  = 4.0
CONSTANT MAX_RAIN  = 500
CONSTANT MAX_RIVER = 15

// ---------- MAIN MODULE ----------
FUNCTION main()
    DECLARE ZONES AS ARRAY OF STRING
    DECLARE zoneName, riskLevel AS STRING
    DECLARE rainfallMM, riverLevelM AS REAL
    DECLARE isValid AS BOOLEAN
    DECLARE zoneIndex, sent, alertsTotal AS INTEGER

    ZONES <- ["Kroo Bay", "Kissy", "Culvert", "Regent", "Dwarzark", "Susan's Bay"]
    zoneIndex <- 1
    alertsTotal <- 0

    WHILE zoneIndex <= LENGTH(ZONES) DO
        zoneName <- ZONES[zoneIndex]
        INPUT rainfallMM, riverLevelM

        isValid <- validateReadings(rainfallMM, riverLevelM)

        IF isValid = FALSE THEN
            OUTPUT "Invalid reading - check sensor for ", zoneName
        ELSE
            riskLevel <- calculateRiskLevel(rainfallMM, riverLevelM)
            IF riskLevel = "SAFE" THEN
                OUTPUT zoneName, " is SAFE"
            ELSE
                sent <- sendAlerts(zoneName, riskLevel)
                alertsTotal <- alertsTotal + sent
                CALL logIncident(zoneName, riskLevel, rainfallMM, riverLevelM)
            END IF
        END IF

        zoneIndex <- zoneIndex + 1
    END WHILE

    CALL generateSummaryReport(alertsTotal)
END FUNCTION

// ---------- VALIDATION ----------
FUNCTION validateReadings(rainfallMM, riverLevelM) RETURNS BOOLEAN
    IF IS_NUMBER(rainfallMM) AND IS_NUMBER(riverLevelM) THEN
        IF rainfallMM >= 0 AND rainfallMM <= MAX_RAIN AND
           riverLevelM >= 0 AND riverLevelM <= MAX_RIVER THEN
            RETURN TRUE
        END IF
    END IF
    RETURN FALSE
END FUNCTION

// ---------- RISK CLASSIFICATION ----------
FUNCTION calculateRiskLevel(rainfallMM, riverLevelM) RETURNS STRING
    DECLARE riskLevel AS STRING
    IF rainfallMM >= DANGER_RAIN OR riverLevelM >= DANGER_RIVER THEN
        riskLevel <- "DANGER"
    ELSE IF rainfallMM >= WARNING_RAIN OR riverLevelM >= WARNING_RIVER THEN
        riskLevel <- "WARNING"
    ELSE IF rainfallMM >= WATCH_RAIN OR riverLevelM >= WATCH_RIVER THEN
        riskLevel <- "WATCH"
    ELSE
        riskLevel <- "SAFE"
    END IF
    RETURN riskLevel
END FUNCTION

// ---------- ALERTING ----------
FUNCTION sendAlerts(zoneName, riskLevel) RETURNS INTEGER
    DECLARE residentList AS ARRAY OF STRING
    DECLARE message AS STRING
    DECLARE index, sent AS INTEGER
    DECLARE delivered AS BOOLEAN

    residentList <- getZoneResidents(zoneName)
    message <- composeAlert(zoneName, riskLevel)
    sent <- 0
    index <- 1

    WHILE index <= LENGTH(residentList) DO
        delivered <- SEND_SMS(residentList[index], message)
        IF delivered = TRUE THEN
            sent <- sent + 1
        ELSE
            ADD residentList[index] TO retryQueue
        END IF
        index <- index + 1
    END WHILE

    IF riskLevel = "DANGER" THEN
        CALL notifyAuthorities(zoneName)
    END IF

    RETURN sent
END FUNCTION

FUNCTION getZoneResidents(zoneName) RETURNS ARRAY OF STRING
    RETURN registered phone numbers for zoneName from the resident register
END FUNCTION

FUNCTION composeAlert(zoneName, riskLevel) RETURNS STRING
    IF riskLevel = "DANGER" THEN
        RETURN "DANGER " + zoneName + ": Evacuate now to the nearest safe centre."
    ELSE IF riskLevel = "WARNING" THEN
        RETURN "WARNING " + zoneName + ": Flooding likely. Move valuables and prepare to leave."
    ELSE
        RETURN "WATCH " + zoneName + ": Heavy rain expected. Clear drains and stay informed."
    END IF
END FUNCTION

FUNCTION notifyAuthorities(zoneName)
    SEND_MESSAGE TO NDMA_CONTACT, "DANGER level flooding risk in " + zoneName
END FUNCTION

// ---------- RECORDS AND REPORTING ----------
FUNCTION logIncident(zoneName, riskLevel, rainfallMM, riverLevelM)
    APPEND (DATE_TIME, zoneName, riskLevel, rainfallMM, riverLevelM) TO incidentLog
END FUNCTION

FUNCTION generateSummaryReport(alertsTotal)
    OUTPUT "===== FLOOD MONITORING SUMMARY ====="
    OUTPUT "Total alerts sent: ", alertsTotal
    OUTPUT "Incidents logged : ", LENGTH(incidentLog)
END FUNCTION

END PROGRAM
```
