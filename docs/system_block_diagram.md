# ESP32 Structural Tension and Vibration Monitoring System

## Simplified Block-Level System Diagram

```text
Mechanical Structure
       |
       +---- Load Cell ---- HX711 ----+
       |                              |
       +---- MPU6050 -----------------+--> ESP32
                                      |
                                      v
                                Feature Extraction
                                      |
                                      v
                                ML Classifier
                                      |
                         +------------+------------+
                         |            |            |
                       NORMAL       FAULT        BREAK
                         |            |            |
                    Green LED     Buzzer/LED   Fail-safe
                                                  |
                                             Laser/Relay
                                             Shutdown
```

## Engineering Subsystem Legend

1. Sensing Subsystem
   - MPU6050 (I2C) for vibration/motion: acceleration X/Y/Z, optional gyro X/Y/Z.
   - Load cell (4-wire Wheatstone bridge) + HX711 ADC for tension force.

2. Signal Acquisition Subsystem
   - ESP32 reads MPU6050 over I2C and HX711 over DOUT/SCK.
   - Common reference ground for low-voltage electronics.

3. ML Processing Subsystem (Firmware/Logical)
   - Feature extraction on ESP32:
     - Acceleration magnitude
     - RMS acceleration
     - Peak acceleration
     - Duration of vibration
     - Frequency-domain features
     - Load/tension trends from HX711
   - Classifier outputs: NORMAL, FAULT, BREAK.

4. Alert and Fail-Safe Subsystem
   - NORMAL: green LED status.
   - FAULT: red LED + buzzer alert.
   - BREAK: fail-safe output via relay and/or isolated optocoupler interlock line.

5. Power Subsystem
   - 5 V input rail.
   - 3.3 V logic rail from ESP32 board regulator for ESP32/MPU6050/HX711 interface.
   - Local decoupling capacitors on sensor and controller rails.

6. Communications Subsystem (Logical)
   - ESP32 Wi-Fi telemetry/event publishing to local server/cloud/API.
   - This is a logical data path, not a direct electrical wire in the schematic.

## Electrical Interface Map (as implemented in schematic)

- I2C:
  - `GPIO21` -> `SDA` (MPU6050)
  - `GPIO22` -> `SCL` (MPU6050)
  - Optional interrupt: `GPIO19` <- `MPU6050 INT`

- HX711:
  - `GPIO4` <- `HX711 DOUT`
  - `GPIO5` -> `HX711 PD_SCK`

- Status/Alert Outputs:
  - `GPIO16` -> Green LED (NORMAL)
  - `GPIO17` -> Red LED (FAULT/BREAK)
  - `GPIO25` -> Buzzer driver transistor
  - `GPIO18` -> Relay module input (optional external fail-safe)
  - `GPIO23` -> Optocoupler LED drive (isolated interlock output)

## Notes for Technical Report

- The laser load is not directly driven by any ESP32 GPIO.
- External hazardous-energy path is represented through relay contacts and an isolated interlock terminal.
- The load cell is represented as a real 4-wire Wheatstone bridge feeding HX711 differential inputs (`INA+`, `INA-`).
