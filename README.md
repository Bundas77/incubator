# Smart ESP32-S3 Automatic Egg Incubator (Wine Cooler Conversion)

An advanced, IoT-enabled, standalone automatic egg incubator built inside a repurposed wine cooler chassis. Powered by an **ESP32-S3** microcontroller and managed via **ESPhome** and **Home Assistant**, featuring a local **20x4 I2C LCD**, a **Rotary Encoder** menu system, and dynamic **Multi-language support (English, Hungarian, Swedish)**.

---

## 🌟 Key Features

- **Precise PID Thermal Control:** Manages heating and cooling via dual TEC1-12706 Peltier elements driven by an IBT-2 (BTS7960) H-bridge with slow PWM (10 Hz). Temperature is averaged across three waterproof DS18B20 probes placed at egg level.
- **Dynamic Humidity Management:** Managed by an ultrasonic 5V atomizer module. Automatically adjusts target humidity based on the incubation lifecycle (e.g., shifting to 68% during the lockdown phase).
- **Multi-language Local Menu:** Switch dynamically between **English, Magyar (Hungarian), and Svenska (Swedish)** directly on the 20x4 LCD using the rotary encoder.
- **Advanced Environmental Monitoring:** Tracks temperature, relative humidity, absolute humidity ($g/m^3$), and barometric pressure using a BME280 sensor.
- **Smart Egg Turning & Ventilation:** Automatic 15-second rolling cycles every 2 hours (auto-disabled during lockdown) and scheduled 30-minute fresh air ventilation cycles.
- **Safety & Fail-safes:** Independent mechanical 40°C thermal cutout (KSD301 Klixon) and 15A fuse protection alongside software alarms.

---

## 🖥️ Local Menu System (20x4 LCD + Rotary Encoder)

In standard operation (Home Screen), the 20x4 LCD displays all critical parameters simultaneously without rolling:

```text
T:37.82C H:55% P:1013   <-- Line 1: Avg Temp, Relative Humidity, Pressure (hPa)
Nap:5/21 Hatra:16d      <-- Line 2: Elapsed Days / Total Days, Remaining Days (or [LOCKDOWN])
AbsP:12.4g CelT:37.8    <-- Line 3: Absolute Humidity (g/m³) & Target Temp
PID:14% CelH:55% V:OFF  <-- Line 4: PID Output %, Target Humidity, Fan Status

---

# ESP32-S3 Smart Egg Incubator - Wiring & Schematic Guide

## 1. Core Microcontroller & Base
- **MCU:** ESP32-S3 N16R8 (DevKitC)
- **Terminal Adapter:** MRD076A Screw Terminal Adapter (ensures stable, vibration-resistant screw connections)

---

## 2. Pinout & Peripheral Connections Table

| Component / Peripheral | Model / Type | ESP32-S3 Pin / Interface | Power Rail / Notes |
| :--- | :--- | :--- | :--- |
| **I2C Bus (LCD & BME280)** | 2004A LCD (I2C) & BME280 | **SDA: GPIO8**, **SCL: GPIO9** | Powered by 5V (LCD) / 3.3V (BME280) |
| **1-Wire Temperature Sensors** | 3x DS18B20 Waterproof Probes | **GPIO4** | Data line with a 4.7kΩ pull-up resistor to 3.3V (Probes wired in parallel) |
| **Peltier Heating/Cooling** | IBT-2 (BTS7960) H-Bridge | **RPWM: GPIO5**, **LPWM: GPIO6** | Slow PWM (10 Hz). Powered from 12V ATX PSU |
| **Ventilation Fan** | 12V DC Fan | **GPIO7** (via Relay Channel 1) | Controlled via Sainsmart 5V Relay Module |
| **Ultrasonic Humidifier** | USB / 5V Ultrasonic Atomizer | **GPIO12** (via Relay Channel 2) | Controlled via Sainsmart 5V Relay Module (5V rail) |
| **Egg Turner Motor** | 12V DC Geared Motor (2.5-5 RPM) | **GPIO17** (via Relay Channel 3) | Controlled via Sainsmart 5V Relay Module (12V rail) |
| **Alarm Buzzer** | Passive Piezo Buzzer | **GPIO16** | Driven by LEDC PWM channel (1000 Hz) |
| **Rotary Encoder** | KY-040 Type (CLK, DT, SW) | **CLK: GPIO10**, **DT: GPIO11**, **SW: GPIO13** | Local menu navigation and pushbutton input |

---

## 3. Power Distribution & Safety Interlock Chain

- **Primary Power Supply:** 700W ATX PC Power Supply connected to an **ATX Breakout Board**.
  - **12V Rail:** Powers the IBT-2 H-Bridge (Peltier elements), 12V DC turning motor, and ventilation fans.
  - **5V Rail:** Powers the ESP32-S3 board, 5V relay module, 2004 LCD display, and the USB ultrasonic atomizer module.
- **Hardware Safety Interlocks:**
  - **15A Fuse:** Installed on the main 12V incoming power line.
  - **KSD301 40°C NC Klixon (Thermal Switch):** Wired physically in series with the 12V power supply to the Peltier heating circuit. If the enclosure temperature exceeds 40°C, it physically cuts power independently of software.
