---
publishDate: 2026-05-24T00:00:00Z
title: Canarion - Autonomous Sub-1 GHz Worker Safety Ecosystem
excerpt: An intelligent RF-based wearable and base station ecosystem that autonomously monitors hazardous environments and detects emergencies without relying on cellular networks.
image: canarion/canarion-cover.jpg
tags:
  - iot
  - wearable
  - safety
  - arduino
  - esp32
  - myosa
---

> Autonomous safety monitoring when every second counts.

---

## Acknowledgements

Special thanks to the IEEE MYOSA (LearnTheEasyWay) initiative, MakeSense EduTech, and Pegasus Automation for providing the core sensor libraries and hardware platform that made this prototype possible.

---

## Overview

Every year, thousands of industrial workers operate in confined spaces, hazardous environments, and isolated zones. Existing safety monitors often rely on spotty cellular coverage or require an active button press—which is impossible if a worker is unconscious from a fall, structural collapse, or toxic gas exposure. 

**Canarion** is a completely autonomous, RF-based worker safety ecosystem. It uses a custom Composite Safety Scoring Engine that continuously blends data from multiple sensors to autonomously determine if a worker is in danger, alerting a remote Base Station instantly without relying on the internet.

**Key features:**
* **Dynamic Composite Scoring Engine** measuring Immobility, Tilt, Pressure, and Light.
* **Auto-Escalating State Machine** (`SAFE` ➔ `ADVISORY` ➔ `WARNING` ➔ `DANGER`).
* **Smart Movement Auto-Clear** to instantly prevent alarm fatigue.
* **Hard-Path Emergency Detection** for Free-Falls, Man-Down, and Manual SOS.
* **Sub-1 GHz RF Telemetry** bypassing the need for Wi-Fi or cellular infrastructure.

---

## Demo / Examples

### **Images**

<p align="center">
  <img src="/assets/images/CANARION/canarion-wearable.jpg" width="800"><br/>
  <i>CANARION wearable unit — ESP32-based with OLED, NeoPixel, and sensor array</i>
</p>

<p align="center">
  <img src="/assets/images/CANARION/canarion-base-station.jpg" width="800"><br/>
  <i>Base Station — 16×2 LCD display with buzzer alarm, positioned outside the hazard zone</i>
</p>

<p align="center">
  <img src="/assets/images/CANARION/canarion-base-station-display.jpg" width="800"><br/>
  <i>Base Station Display Content - State and other parameters on the 16x2 LCD display</i>
</p>

<p align="center">
  <img src="/assets/images/CANARION/canarion-system-diagram.jpg" width="800"><br/>
  <i>System architecture — wearable to base station wireless telemetry flow</i>
</p>

### **Videos**

**Presentation**
https://github.com/user-attachments/assets/072589d9-9e0d-4eef-936c-297825fb0a41

<video controls width="100%">
  <source src="/canarion-presentation.mp4" type="video/mp4">
</video>

**Live Demo**
https://github.com/user-attachments/assets/1716b9a6-cfe9-4def-8e47-fb05754ba1c5

<video controls width="100%">
  <source src="/canarion-demo.mp4" type="video/mp4">
</video>

---

## Features (Detailed)

Canarion's architecture is split into a Wearable module (worn by the worker) and a Base Station (monitored by the command center).

### **1. Dynamic Composite Scoring Engine**
Rather than using simple timers, the Wearable continuously calculates a 0 to 100% Hazard Score. It blends data from the IMU (stillness and tilt), the Barometric Pressure sensor (sudden altitude/environmental changes), and the APDS-9960 sensor (ambient darkness). As the worker enters a hazardous state, the score naturally climbs, pushing the state machine through `SAFE`, `ADVISORY`, `WARNING`, and `DANGER`.

### **2. The "Are You Okay?" (AYO) Challenge**
When the Hazard Score hits 50% (`WARNING`), the OLED display pops up an "Are You Okay?" challenge. The worker must respond by holding their hand over the contactless gesture sensor for 1 second. If they fail to respond within 15 seconds, the system assumes they are incapacitated and escalates to a full `EMERGENCY`.

### **3. Smart Movement Auto-Clear**
To solve industry-wide "Alarm Fatigue," Canarion uses an auto-clear algorithm. If a worker triggers an AYO popup or a Man-Down timer simply because they were resting, standing back up and moving instantly drops the Hazard Score to zero and silences the alarms across the entire network—requiring zero button presses.

### **4. Hard-Path Emergencies**
For catastrophic events, the scoring engine is bypassed:
* **Confirmed Fall:** Requires 3 simultaneous events to prevent false positives: 0G Free-fall + >2.5G Impact + Barometric Pressure Drop (altitude change).
* **Man-Down:** Triggers if the worker is perfectly still and tilted 90-degrees sideways for 10 seconds.
* **Manual SOS:** Triggered by rapidly swiping a hand (Left-Right-Left-Right) over the gesture sensor.

---

## Usage Instructions

1. Power on the **Base Station** first. It will display a "Waiting for Signal..." prompt.
2. Power on the **Wearable**. It will immediately begin transmitting telemetry packets via the HC-12 transceiver.
3. The Base Station will automatically rotate through 3 pages of live telemetry (Status, Motion, Environment) every 3 seconds.
4. **Triggering an Emergency:** Lay the Wearable on its side and leave it perfectly still for 10 seconds to trigger the Man-Down emergency. Shake the device to automatically clear it.
5. **Contactless Navigation:** Swipe UP/DOWN or LEFT/RIGHT over the Wearable's gesture sensor to manually page through the OLED telemetry screens.

If you are a developer testing the system while plugged into a laptop, you can simulate a fall via the Serial Monitor:
```plaintext
z
```

---

## Tech Stack

* **Hardware:** Arduino Microcontrollers, MPU6050 (IMU), BMP180 (Pressure), APDS-9960 (Gesture/Lux), HC-12 (Sub-1 GHz RF), OLED 128x64, LCD 16x2.
* **Software:** C++ / Arduino Framework
* **Libraries:** MYOSA Platform Libraries, Adafruit GFX, Adafruit SSD1306, Wire.

---

## Requirements / Installation

No external dependencies are required beyond the standard Arduino IDE and the MYOSA hardware libraries.

1. Clone the repository to your local machine.
2. Ensure the `MYOSA`, `LightProximityAndGesture`, `OLED`, and `TempAndHumidity` libraries are installed in your `Arduino/libraries` folder.
3. Flash `canarion_main.ino` to the Wearable module.
4. Flash `base_station_main.ino` to the Base Station module.

---

## File Structure

```
/canarion
  ├─ wearable/
  │   ├─ src/
  │   │   ├─ canarion_main/
  │   │   │   └─ canarion_main.ino
  │   │   ├─ imu_detector.h
  │   │   ├─ pressure_detector.h
  │   │   ├─ gesture_detector.h
  │   │   └─ display_manager.h
  ├─ base_station/
  │   ├─ src/
  │   │   ├─ base_station_main/
  │   │   │   └─ base_station_main.ino
  │   │   └─ status_display.h
  ├─ common/
  │   └─ protocol.h
  ├─ canarion-cover.jpg
  ├─ canarion-demo.mp4
  └─ canarion.md
```

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## Contribution Notes

We welcome contributions! If you would like to help improve the Composite Scoring weights or expand the RF protocol to support mesh networking between multiple workers, please open an Issue or submit a Pull Request.
