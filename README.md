---
Publish Date: 2026-05-24

### CANARION — Intelligent Safety for Confined Spaces

#### A multi-sensor wearable safety system that detects falls, toxic exposure, and emergencies in real time — built for the workers who keep the world running beneath our feet.

image: CANARION/canarion-cover.jpg

  - Embedded Systems
  - IoT
  - Safety Tech
  - ESP32
  - Sensor Fusion
---

> *"In a confined space, an early warning is the only difference between a rescue and a tragedy."*

---

## Acknowledgements

Built with dedication by **Team Aegis Sentinels** — **Joel Kurian Biju**, **Jovan Shaji**, and **Cyriac T Kappan** — as part of the **MYOSA 5.0** challenge.

The name *CANARION* is a tribute to the miners who once carried canaries underground as their only warning against invisible death. This project is dedicated to every worker who sustains our world from its very foundation.

---

## Overview

Beneath every functioning city — inside sewage tunnels, industrial boilers, storage silos, and underground shafts — there is a workforce that rarely makes headlines. These confined-space workers face environments where oxygen can vanish in seconds and toxic gases accumulate without warning. A worker can collapse silently. Rescuers entering to help can become victims themselves.

**CANARION** is a wearable, multi-sensor confined-space safety monitoring system designed to detect dangerous conditions *before* they become fatal. It combines embedded sensor fusion, intelligent safety state management, and wireless radio telemetry to deliver real-time situational awareness to both workers and supervisors — even when visibility is zero and ventilation is nonexistent.

### Key Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Fall Detection** | Multi-axis IMU sensor fusion — rapid descent AND slow slump |
| 2 | **Pressure Monitoring** | Barometric confirmation of vertical falls |
| 3 | **Gesture Control** | Works with heavy gloves — no buttons required |
| 4 | **Safety State Machine** | Five-tier: `SAFE` → `ADVISORY` → `WARNING` → `DANGER` → `EMERGENCY` |
| 5 | **Wireless Telemetry** | 433 MHz HC-12 radio to an out-of-hazard Base Station |
| 6 | **Black-Box Logging** | CSV telemetry log for post-incident auditing |
| 7 | **On-Body Alerts** | OLED display + piezo buzzer + NeoPixel LED |
| 8 | **Accessible Hardware** | Affordable, deployable where commercial systems can't reach |

---

## Demo / Examples

### Images

<!-- Place hardware photos in the same folder as this file before submitting -->

<p align="center">
  <img src="/assets/images/CANARION/canarion-wearable.jpg" width="800"><br/>
  <i>CANARION wearable unit — ESP32-based with OLED, NeoPixel, and sensor array</i>
</p>

<p align="center">
  <img src="/assets/images/CANARION/canarion-base-station.jpg" width="800"><br/>
  <i>Base Station — 16×2 LCD display with buzzer alarm, positioned outside the hazard zone</i>
</p>

<p align="center">
  <img src="/assets/images/CANARION/canarion-system-diagram.jpg" width="800"><br/>
  <i>System architecture — wearable to base station wireless telemetry flow</i>
</p>

### Videos

**Presentation**

<!-- Add canarion-presentation.mp4 to the same folder -->

<video controls width="100%">
  <source src="CANARION/canarion-presentation.mp4" type="video/mp4">
</video>

**Live Demo**

<!-- Add canarion-demo.mp4 to the same folder -->

<video controls width="100%">
  <source src="CANARION/canarion-demo.mp4" type="video/mp4">
</video>

---

## Features (Detailed)

### 1. Multi-Sensor Hardware Core

At the heart of the CANARION wearable is an **ESP32** development board acting as the central processing unit. It orchestrates four key sensing components:

| Sensor | Role |
|--------|------|
| **MPU6050** | 6-axis IMU — acceleration, gyroscope, orientation, impact force |
| **BMP180** | Barometric pressure — confirms vertical descent during falls |
| **APDS9960** | Ambient light + gesture recognition — glove-friendly interaction |
| **SSD1306 OLED** | Real-time on-body display of safety state and sensor readings |

---

### 2. Sensor Fusion & Emergency Detection

Hardware alone isn't enough. CANARION's intelligence lives in its **sensor fusion algorithm** — rather than reacting to any single reading, it combines multiple data streams to make confident, low-false-alarm safety decisions.

Two major emergency detection mechanisms are built in:

**Rapid Descent Detection**
Triggered only when free-fall acceleration, gyroscope spin, and a pressure drop all occur simultaneously. A simple stumble will not trigger this.

**Slow Slump Detection**
Toxic gas exposure often causes gradual, silent unconsciousness — not a dramatic fall. CANARION tracks long-term orientation and activity variance to catch this invisible danger.

```cpp
// Safety State Enum — shared between wearable and base station
enum class SafetyState : uint8_t {
    SAFE      = 0,
    ADVISORY  = 1,
    WARNING   = 2,
    DANGER    = 3,
    EMERGENCY = 4
};

inline const char* safetyStateToString(SafetyState s) {
    switch (s) {
        case SafetyState::SAFE:      return "SAFE";
        case SafetyState::ADVISORY:  return "ADVISORY";
        case SafetyState::WARNING:   return "WARNING";
        case SafetyState::DANGER:    return "DANGER";
        case SafetyState::EMERGENCY: return "EMERGENCY";
        default:                     return "UNKNOWN";
    }
}
```

---

### 3. Intentionality-Gated Gesture Engine

To prevent accidental triggers from industrial vibrations or nearby tools, CANARION uses a **proximity-gated gesture system**. The APDS9960 only enters gesture-listening mode after detecting a deliberate proximity activation — the worker must intentionally prime the sensor before any gesture command is accepted.

This keeps the system reliable in noisy, high-vibration environments where accidental inputs could be dangerous.

---

### 4. Five-Tier Safety State Machine

CANARION manages risk through five states that escalate gradually based on a calculated risk score:

| State | Severity | Behaviour |
|-------|----------|-----------|
| `SAFE` | ✅ Normal | All sensors nominal, worker active |
| `ADVISORY` | 🔵 Low | Minor anomaly detected, monitoring closely |
| `WARNING` | 🟡 Elevated | Local alerts activated (buzzer, LED, OLED) |
| `DANGER` | 🟠 High | Radio telemetry to Base Station begins |
| `EMERGENCY` | 🔴 Critical | Full alarms, continuous retransmission every 5 s |

---

### 5. HC-12 Wireless Telemetry & CANARIONv1 Protocol

The wearable transmits structured ASCII packets over **433 MHz HC-12 radio** to the Base Station, which sits safely outside the hazard zone. The packet format is compact, human-readable, and backward-compatible:

```
CANARIONv1|<STATE>|<REASON>|<SCORE>|<BATT>|<UPTIME>|<PRESSURE>|<TILT>|<ACCEL>|<ACTVAR>|<TEMP>|<LUX>
```

**Example — live emergency packet:**

```
CANARIONv1|EMERGENCY|FALL CONFIRMED|0.92|73|00142|1013.2|89.5|3.21|0.002|28.4|145
```

**Transmission rules:**

- Transmits only when state ≥ `WARNING` — silent in safe conditions (conserves power)
- Retransmits every **5 seconds** during `EMERGENCY`
- Sends a single `CLEAR` packet when conditions normalise

---

### 6. Base Station — The Supervisor's Eyes

The Base Station is a second ESP32 unit placed outside the hazard zone. It continuously parses incoming CANARIONv1 packets and responds accordingly:

- **16×2 I2C LCD** — rotates across 3 telemetry pages every 3 s (status/alerts → motion data → environmental data)
- **Buzzer** — single beep for `WARNING`/`DANGER`, continuous 500 ms toggle for `EMERGENCY`
- **CSV Serial Log** — all packets timestamped and logged for post-incident auditing

```cpp
// Base Station — emergency buzzer response logic
if (p.state == "EMERGENCY" || p.reason.indexOf("SOS") >= 0) {
    bsSosActive    = true;
    bsLastBuzzerMs = millis();
    Serial.println("[BS] EMERGENCY/SOS alert — buzzer enabled");

} else if (p.state == "WARNING" || p.state == "DANGER") {
    digitalWrite(BS_BUZZER_PIN, HIGH);
    delay(80);
    digitalWrite(BS_BUZZER_PIN, LOW);

} else if (p.state == "SAFE" && p.reason == "CLEAR") {
    bsSosActive = false;
    digitalWrite(BS_BUZZER_PIN, LOW);
    Serial.println("[BS] CLEAR received — buzzer disabled");
}
```

The LCD auto-detects I2C address (`0x27` or `0x3F`) — no manual configuration needed.

---

## Usage Instructions

### Wearable Setup

```bash
# 1. Flash wearable/src to the ESP32 wearable board
# 2. Ensure all I2C devices are correctly powered and addressed
# 3. Connect HC-12 to UART2 — GPIO16 (RX), GPIO17 (TX)
# 4. Set HC-12 SET pin HIGH to enter data mode
```

### Base Station Setup

```bash
# 1. Flash base_station/src to a second ESP32 board
# 2. Connect HC-12 — TXD -> RX2 (GPIO16), RXD -> TX2 (GPIO17)
# 3. Connect buzzer signal to GPIO18 (use transistor/driver for higher current loads)
# 4. Wire 16x2 I2C LCD — SDA -> GPIO21, SCL -> GPIO22, VCC -> 5V(VIN), GND -> GND
# 5. Open Serial monitor at 115200 baud to view live telemetry logs
```

### Testing

```bash
# 1. Power both the wearable and base station
# 2. Simulate WARNING by triggering sensor thresholds, or force SOS via gesture waggle
# 3. Watch parsed packets appear on the Base Station Serial monitor in CSV format:
#    timestamp, state, reason, score, battPct, uptimeS
```

---

## Tech Stack

| Layer | Component |
|-------|-----------|
| **Microcontroller** | ESP32 (dual-core, UART, I2C) |
| **IMU** | MPU6050 — accelerometer + gyroscope (I2C) |
| **Pressure** | BMP180 — barometric sensor (I2C) |
| **Gesture / Light** | APDS9960 — proximity, gesture, ambient light (I2C) |
| **Wearable Display** | SSD1306 OLED 128×64 (I2C) |
| **Base Station Display** | 16×2 LCD with PCF8574 I2C backpack |
| **Radio** | HC-12 433 MHz UART module (both devices) |
| **Alerts** | NeoPixel RGB LED + piezo buzzer |
| **Firmware** | Arduino framework (C/C++) |
| **Protocol** | Custom CANARIONv1 ASCII over UART |

---

## Requirements / Installation

### Arduino Libraries

Install via **Arduino Library Manager** or PlatformIO:

```bash
# Wearable
MPU6050               # I2C IMU driver
Adafruit_BMP085       # BMP180 pressure sensor
SparkFun_APDS9960     # Gesture and light sensor
Adafruit_SSD1306      # OLED display
Adafruit_NeoPixel     # NeoPixel RGB LED

# Base Station
LiquidCrystal_I2C     # 16x2 LCD with I2C backpack
```

### Hardware Wiring — Quick Reference

| Component | Signal | ESP32 Pin |
|-----------|--------|-----------|
| MPU6050 | SDA / SCL | GPIO 21 / 22 |
| BMP180 | SDA / SCL | GPIO 21 / 22 |
| APDS9960 | SDA / SCL | GPIO 21 / 22 |
| SSD1306 OLED | SDA / SCL | GPIO 21 / 22 |
| HC-12 Radio | RX / TX | GPIO 16 / 17 |
| NeoPixel | Data | GPIO 4 |
| Buzzer | Signal | GPIO 18 |

> All I2C devices share the same bus (GPIO 21/22). Ensure each device has a unique I2C address.

---

## File Structure

```
/canarion
  ├─ wearable/
  │   └─ src/
  │       ├─ canarion_main.ino       # Main wearable sketch
  │       ├─ imu_detector.h          # Fall & slump detection logic
  │       ├─ pressure_detector.h     # Barometric fall confirmation
  │       ├─ display_manager.h       # OLED rendering
  │       └─ types.h                 # Safety state enums
  ├─ base_station/
  │   └─ src/
  │       ├─ base_station.ino        # Main base station sketch
  │       ├─ hc12_receiver.h         # UART2 packet receiver
  │       ├─ status_display.h        # LCD display manager
  │       └─ logger.h                # CSV serial logger
  ├─ common/
  │   ├─ protocol.h                  # CANARIONv1 packet format & parser
  │   └─ utilities.h                 # Shared helpers (CSV escaping, etc.)
  └─ docs/
      ├─ architecture.md
      ├─ deployment.md
      └─ wiring.md
```

---

## License

This project is open-source, submitted under the **MIT License** as part of MYOSA 5.0.

---

*CANARION — Because the workers beneath our feet deserve more than silence.*
