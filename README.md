# Smart ESP32-S3 Automatic Egg Incubator (Wine Cooler Conversion)

## Technical Manual & Documentation

---

## 1. System Overview & Features

This document describes an advanced, standalone automatic egg incubator built inside a repurposed wine cooler chassis, powered by an **ESP32-S3** microcontroller. The system combines precise **PID temperature control** (using Peltier elements) with comprehensive environmental monitoring: it tracks egg-level average temperature, relative humidity, **absolute humidity ($g/m^3$)**, and **barometric pressure**.

Operation can be fully managed either locally via a **20x4 I2C LCD display** and a **Rotary Encoder** menu system or remotely via Home Assistant through Wi-Fi and ESPhome.

---

## 2. Functional Description & Operational Principles

* **Temperature Control (PID + Peltier):** Three waterproof DS18B20 sensors placed directly at egg level measure the temperature, and the system uses their average. Heating and cooling are handled by two TEC1-12706 Peltier elements driven by an IBT-2 (BTS7960) H-bridge via slow PWM (10 Hz).


* **Dynamic Humidity & Atomization:** A 5V USB ultrasonic atomizer module provides precise misting. The system **automatically increases** the target humidity during the lockdown/hatching phase (to 68%) based on the incubation day, while allowing manual overrides anytime.


* **Ventilation & Air Exchange:** Features a 15-second fresh air exchange cycle every 30 minutes, alongside emergency cooling if the temperature exceeds 38.5 °C.


* **Smart Egg Turning:** A 12V DC geared motor rotates the 4-tier roller trays for 15 seconds every 2 hours. Turning automatically stops prior to the hatching phase (defaulting to stop 3 days before the end of the total incubation period, e.g., day 18 for a 21-day cycle).


* **NTP Time Synchronization:** Automatically synchronizes real-time clock data via NTP upon internet connection, advancing the incubation day counter automatically at midnight.



---

## 3. Local Menu System & Display (20x4 LCD + Rotary Encoder)

In standard operation (Home Screen), the 20x4 LCD displays all critical parameters simultaneously without rolling:

```text
T:37.82C H:55% P:1013   <-- Line 1: Avg Temp, Relative Humidity, Pressure (hPa)
Nap:5/21 Hatra:16d      <-- Line 2: Elapsed Days / Total Days, Remaining Days (or [BUJTATAS])
AbsP:12.4g CelT:37.8    <-- Line 3: Absolute Humidity (g/m³) & Target Temp
PID:14% CelH:55% V:OFF  <-- Line 4: PID Output %, Target Humidity, Fan Status

```

### Using the Local Menu (Rotary Encoder):

Pressing the rotary encoder button enters the settings menu. Turn the knob to navigate through options, and press again to edit:

1. **Target Temperature** (`CelT`): Adjustable in `±0.1 °C` increments.


2. **Target Humidity** (`CelH`): Adjustable in `±1%` increments (range: 40–80%).


3. **Current Incubation Day** (`Nap`): Manual correction option.


4. **Total Incubation Days** (e.g., `21` for chickens, `28` for turkeys).


5. **Stop Turning Before Hatching** (Days before end to disable the turning motor, e.g., `-3` days).


6. **Save and Exit** (`Mentes es Kilepes`).
Note: Pressing the button during any active acoustic alarm instantly mutes the buzzer.



---

## 4. User Guide & Operating Procedure

1. **Test & PID Autotune:** Run a dry test run with the system empty. Trigger the **"Start PID Autotune"** button from Home Assistant to let the algorithm adapt to the thermal inertia of the enclosure.


2. **Initialization (Day 0):** Load the eggs and press the **"Reset Incubation Days"** button (or configure via the local menu). The day counter starts at `0` and is safely stored in non-volatile memory through power outages.


3. **Hatching / Lockdown Phase:** When the system reaches the configured lockdown threshold (e.g., day 18 of a 21-day cycle), turning automatically halts, the LCD shows hatching status, and target humidity rises to 68%.


4. **Alarms & Safety:** If overheating occurs (>38.5 °C), humidity drops drastically, or a sensor fails, the LCD switches to a flashing full-screen warning (`!!! VIGYAZAT !!!`) and the buzzer sounds.



---

## 5. Hardware Specification List

| Component Name | Type / Model | Primary Function in System |
| --- | --- | --- |
| **Microcontroller** | ESP32-S3 N16R8 (DevKitC) | Dual-core MCU, 16 MB Flash, Wi-Fi, OTA updates, local memory management.

 |
| **Terminal Adapter** | MRD076A Screw Terminal Base | Provides stable, vibration-resistant wiring for the ESP32.

 |
| **Power Supply** | 700W ATX PC Power Supply | Industrial-grade, high-capacity baseline power source.

 |
| **Power Distribution** | ATX Breakout Board | Screw-terminal breakout for the 24-pin ATX connector (12V & 5V rails).

 |
| **Heating / Cooling** | IBT-2 (BTS7960) H-Bridge + 2x TEC1-12706 Peltier | Bidirectional precision heat pump controlled via slow PWM (10 Hz).

 |
| **Switching Center** | Sainsmart 8-Channel 5V Relay Module | Optocoupled, galvanically isolated relays for fans, motor, and humidifier.

 |
| **Turning Motor** | 12V DC Geared Motor (2.5–5 RPM) | Quiet, high-torque drive for the 4-tier PVC roller trays.

 |
| **Humidifier** | USB / 5V Ultrasonic Atomizer Module | Ceramic rod cold-mist generator for precise programmable humidity.

 |
| **Local Display** | 2004A (20x4) LCD + PCF8574 I2C Adapter | 4-line status screen displaying data, pressure, absolute humidity, and menus.

 |
| **Local Interface** | Rotary Encoder Module (KY-040 type) | Turning knob and pushbutton for local menu navigation and parameter adjustments.

 |
| **Temperature Sensors** | 3x DS18B20 Waterproof Probes (1-Wire, GPIO4) | Egg-level internal temperature reading; software averages all three probes.

 |
| **Environmental Sensor** | BME280 I2C Sensor (GPIO8/9) | Measures ambient temperature, relative humidity ($H$), and barometric pressure ($P$).

 |
| **Safety Cutoff** | KSD301 40°C NC Klixon + 15A Fuse | Independent physical mechanical safety cutoff chain against overheating or shorts.

 |
| **Acoustics** | Passive Piezo Buzzer (GPIO16) | 1000 Hz acoustic alarm device for fault conditions.



---

# ESP32-S3 Smart Egg Incubator - Wiring & Schematic Guide

## 1. Microcontroller & Core Board

* **MCU:** ESP32-S3 N16R8 (DevKitC)
* **Terminal Base:** MRD076A Screw Terminal Adapter (used for secure, vibration-resistant connections)

---

## 2. Pinout & Peripheral Connections Table

| Component / Peripheral | Model / Type | ESP32-S3 Pin / Interface | Notes & Power Connections |
| --- | --- | --- | --- |
| **I2C Bus (LCD & BME280)** | 2004A LCD (I2C) & BME280 | **SDA: GPIO8**, **SCL: GPIO9** | Powered by 5V (LCD) / 3.3V (BME280), with built-in pull-ups. |
| **1-Wire Sensors** | 3x DS18B20 Waterproof Probes | **GPIO4** | Data line connected to GPIO4 with a 4.7kΩ pull-up resistor to 3.3V. Probes wired in parallel. |
| **Peltier Heating/Cooling** | IBT-2 (BTS7960) H-Bridge | **RPWM: GPIO5**, **LPWM: GPIO6** | Slow PWM (10 Hz). Powered directly from 12V ATX PSU. |
| **Ventilation Fan** | 12V DC Fan | **GPIO7** (via Relay Channel 1) | Controlled via Sainsmart 5V Relay Module. |
| **Humidifier** | USB / 5V Ultrasonic Atomizer | **GPIO12** (via Relay Channel 2) | Controlled via Sainsmart 5V Relay Module (5V rail). |
| **Egg Turner Motor** | 12V DC Geared Motor (2.5-5 RPM) | **GPIO17** (via Relay Channel 3) | Controlled via Sainsmart 5V Relay Module (12V rail). |
| **Alarm Buzzer** | Passive Piezo Buzzer | **GPIO16** | Driven by LEDC PWM channel (1000 Hz). |
| **Rotary Encoder** | KY-040 Type (CLK, DT, SW) | **CLK: GPIO10**, **DT: GPIO11**, **SW: GPIO13** | Used for local menu navigation and parameter adjustments. SW pulled low on press. |

---

## 3. Power Distribution & Safety Interlocks

* **Primary Power Supply:** 700W ATX PC Power Supply connected to an **ATX Breakout Board**.
* **12V Rail:** Powers the IBT-2 H-Bridge (Peltier elements), 12V DC turning motor, and ventilation fans.
* **5V Rail:** Powers the ESP32-S3 board, 5V relay module, 2004 LCD display, and the USB ultrasonic atomizer module.


* **Safety & Hardware Interlock Chain:**
* **15A Fuse:** Installed on the main 12V incoming power line.
* **KSD301 40°C NC Klixon (Thermal Switch):** Wired physically in series with the 12V power supply to the Peltier heating circuit. If the enclosure temperature exceeds 40°C, it physically cuts power independently of software.
