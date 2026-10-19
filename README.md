# Cabin Safety Monitoring System

An **ATmega328P-based embedded automotive safety prototype** integrating environmental sensors, gas detection, ultrasonic proximity sensing, and inertial motion monitoring. The system evaluates sensor readings, displays hazard status on an LCD, generates visual and audible alerts, and controls a servo-based braking prototype.

## Overview

The system combines multiple sensor inputs with threshold-based decision logic to identify potentially hazardous cabin and vehicle conditions. It performs startup initialization and MPU6050 availability checks, calibrates gyroscope offsets, and evaluates sensor data periodically to determine the appropriate system status and response.

## Features

- **Temperature and Humidity Monitoring:** Reads DHT11 data using a manually implemented timing-based communication protocol.
- **Gas Hazard Detection:** Reads analog values from MQ-6 and MQ-7 sensors and compares them against configurable baseline thresholds.
- **Obstacle Detection:** Uses the HC-SR04 ultrasonic sensor to measure distance and identify warning and critical proximity conditions.
- **Motion and Tilt Monitoring:** Reads MPU6050 accelerometer and gyroscope registers through I²C.
- **Gyroscope Calibration:** Estimates gyroscope offsets using multiple startup samples.
- **Hazard Evaluation:** Detects configured impact, tilt, gas, temperature, and obstacle conditions.
- **State-Based Alerts:** Prioritizes hazards and displays descriptive system statuses.
- **LCD Interface:** Displays the current status and servo brake state on a 16×2 LCD.
- **Visual and Audible Alerts:** Controls green, yellow, and red LEDs and a buzzer.
- **Servo-Based Brake Prototype:** Applies or releases the servo according to obstacle conditions, latched hazards, and MPU6050 availability.

## Hardware Components

| Component | Purpose |
|---|---|
| ATmega328P | Main microcontroller |
| DHT11 | Temperature and humidity sensing |
| MQ-6 | LPG-related gas sensing |
| MQ-7 | Carbon monoxide-related gas sensing |
| HC-SR04 | Ultrasonic distance measurement |
| MPU6050 | Accelerometer and gyroscope measurements |
| 16×2 LCD | System status display |
| SG90 servo | Brake-control prototype |
| Green LED | Normal status indication |
| Yellow LED | Warning indication |
| Red LED | Critical alert indication |
| Buzzer | Audible hazard indication |

## Pin Configuration

| Component | ATmega328P Pin |
|---|---|
| DHT11 data | D2 |
| Servo signal | D0 / RX |
| HC-SR04 Trigger | D10 |
| HC-SR04 Echo | D13 |
| MQ-7 analog output | A0 |
| MQ-6 analog output | A1 |
| Green LED | A2 |
| Yellow LED | A3 |
| Red LED | D11 |
| Buzzer | D12 |
| LCD RS | D8 |
| LCD Enable | D9 |
| LCD D4–D7 | D4–D7 |
| MPU6050 SDA/SCL | I²C pins |

**Important:** Verify the board's actual pin mapping before wiring. On an ATmega328P board such as the Arduino Uno, D0 is also the hardware UART RX pin. The supplied firmware assigns D0 to the servo and therefore avoids using `Serial.begin()`.

## System Architecture

```text
 DHT11 ─────────────┐
 MQ-6 / MQ-7 ───────┤
 HC-SR04 ───────────┤
 MPU6050 ───────────┤
                    ▼
             ┌─────────────┐
             │ ATmega328P  │
             │             │
             │ Sensor Read │
             │ I²C / ADC   │
             │ Thresholds  │
             │ Hazard Logic│
             └──────┬──────┘
                    │
          ┌─────────┼──────────┐
          ▼         ▼          ▼
       16×2 LCD   LEDs/Buzzer  Servo
       Status     Alerts      Brake
```

## Operating Logic

The firmware evaluates sensor readings in a prioritized sequence.

| Condition | System Response |
|---|---|
| Stable sensor conditions | Green LED; `System Stable` |
| Warm cabin | Yellow LED; `Cabin Warm` |
| Gas warning threshold crossed | Yellow LED; `Leak Suspected` |
| Obstacle within warning range | Yellow LED; `Obstacle Ahead` |
| Critical obstacle proximity | Red LED and buzzer; brake applied |
| Configured impact detected | Critical hazard latched; brake applied |
| Fire-risk condition detected | Critical alert; brake applied |
| MPU6050 unavailable | Brake remains applied |

Additional tilt and combined-condition checks are implemented in the firmware. Critical hazards are latched until the controller is reset.

## Configurable Thresholds

The following values are defined in the source code and can be adjusted during controlled testing.

| Parameter | Configured Value |
|---|---:|
| MQ-6 baseline | 200 ADC counts |
| MQ-7 baseline | 200 ADC counts |
| Gas warning rise | 25 counts |
| Gas hazard rise | 60 counts |
| Warm cabin threshold | 35°C |
| Hot cabin threshold | 45°C |
| Presence distance | 30 cm |
| Obstacle warning distance | 20 cm |
| Critical obstacle distance | 10 cm |
| Tilt warning angle | 15° |
| Rollover angle threshold | 60° |
| Acceleration magnitude threshold | 2.5 g |
| Gyroscope impact threshold | 100°/s |
| Sensor evaluation interval | 1.5 seconds |

These are **prototype thresholds**, not validated automotive safety limits. MQ sensor ADC values depend on sensor calibration, operating conditions, and the surrounding environment.

## Software and Libraries

- **Language:** Embedded C/C++ using the Arduino framework
- **Microcontroller:** ATmega328P
- **Libraries:** `Wire.h`, `LiquidCrystal.h`, `Servo.h`, `math.h`
- **Communication:** I²C for MPU6050 register access
- **Programming Concepts:** GPIO, ADC, timing-based sensor communication, threshold logic, state-based control, sensor calibration, and fault handling

The DHT11 communication routine and MPU6050 register-reading logic are implemented directly in the project source.

## Getting Started

1. Clone or download this repository.
2. Install the required Arduino libraries and select the correct ATmega328P-based board.
3. Connect the sensors, LCD, LEDs, buzzer, and servo using the pin configuration above.
4. Verify power requirements, common ground, I²C wiring, and sensor output compatibility.
5. Open the `.ino` source file in the Arduino IDE.
6. Compile and upload the firmware.
7. Keep the MPU6050 stationary during startup gyroscope calibration.
8. Test sensor readings and alert transitions in a controlled environment.

Check the source code for the complete pin definitions, thresholds, and initialization sequence.

## Limitations

- Gas detection thresholds are based on raw analog readings and require calibration for reliable measurements.
- Impact and tilt detection use configured thresholds and are not substitutes for validated crash-detection algorithms.
- The servo represents a braking-control prototype and must not be connected to or relied upon as a real vehicle braking system.
- The firmware depends on correct sensor wiring, stable power, and appropriate sensor initialization.
- The current implementation uses blocking delays and a periodic sensor evaluation loop.

## Future Improvements

- Add calibrated gas concentration estimation and sensor fault diagnostics.
- Improve filtering and validation of accelerometer and gyroscope data.
- Introduce a non-blocking scheduling approach for sensor acquisition.
- Add event logging and wireless monitoring.
- Validate threshold selection through repeatable test cases.

## Disclaimer

This project is intended for **educational and prototyping purposes only**. It is not a certified automotive safety system and must not be used as the sole basis for real-world vehicle braking, collision avoidance, or gas-safety decisions.

## Author

**Shrawani Wagh**  
Electronics and Telecommunication Engineering  
[GitHub](https://github.com/shrawani0408-boop) · [LinkedIn](https://linkedin.com/in/shrawani-wagh-842a10332)
