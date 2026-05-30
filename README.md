# RuView IoT Home Lab

> **WiFi Sensing + IoT Automation Platform**
> Detect people through walls using WiFi CSI, then automate your home with sensors, motors, and lights.

## Overview

This project combines the **RuView WiFi Sensing Platform** with a custom **IoT Home Automation Lab** using ESP32-C6 nodes, a Raspberry Pi 5 hub, and various sensors/actuators. All items are fully programmable via WiFi 6 / Bluetooth 5.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    HOME NETWORK (WiFi 6)                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐      UDP Stream      ┌─────────────────┐ │
│   │ ESP32-C6    │ ◄──────────────────► │  Raspberry Pi 5 │ │
│   │ Node 1      │   Port 5005 (CSI)    │     (Hub)       │ │
│   │ Living Room │                      │  RuView Server  │ │
│   └─────────────┘                      │  Docker Compose │ │
│                                        │                 │ │
│   ┌─────────────┐                      │  • Rust Axum    │ │
│   │ ESP32-C6    │ ◄──────────────────► │  • Python WS    │ │
│   │ Node 2      │   Port 5005 (CSI)    │  • React UI     │ │
│   │ Bedroom     │                      │  • nvsim WASM   │ │
│   └─────────────┘                      └─────────────────┘ │
│                                                             │
│   ┌─────────────┐      UDP Stream                          │
│   │ ESP32-C6    │ ◄──────────────────►                     │
│   │ Node 3      │   Port 5005 (CSI)                        │
│   │ Kitchen     │                                          │
│   └─────────────┘                                          │
│                                                             │
│   ┌─────────────┐      UDP Stream                          │
│   │ ESP32-C6    │ ◄──────────────────►                     │
│   │ Node 4      │   Port 5005 (CSI)                        │
│   │ Hallway     │                                          │
│   └─────────────┘                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Hardware Inventory (Amazon India Cart)

### Core Hub (1x)
| Item | Qty | Purpose |
|------|-----|---------|
| Raspberry Pi 5 8GB RAM Starter Kit | 1 | Central hub: RuView server, Docker, WiFi AP |

### WiFi Sensing Nodes (4x)
| Item | Qty | Purpose |
|------|-----|---------|
| Espressif ESP32-C6-DevKitC-1-N8 | 4 | WiFi 6 + Bluetooth 5 nodes for CSI streaming |

### Sensors (All connect to ESP32 GPIO)
| Item | Qty | Protocol | Purpose |
|------|-----|----------|---------|
| BME280 (Temp/Humidity/Pressure) | 1 | I2C | Environmental monitoring |
| DHT22 (Temp/Humidity) | 1 | 1-Wire | Backup environmental sensor |
| HC-SR501 PIR Motion Sensor | 1 | Digital GPIO | Motion detection |
| Soil Moisture Sensor (in kit) | 1 | Analog GPIO | Plant watering automation |

### Actuators (All connect to ESP32 GPIO)
| Item | Qty | Control | Purpose |
|------|-----|---------|---------|
| L298N Motor Driver Module | 1 | PWM/GPIO | DC motor / stepper control |
| SG90 Micro Servo Motors | 2 | PWM | Mechanical movement |
| 4-Channel Relay Module | 1 | Digital GPIO | Switch AC devices (lights, fans) |
| WS2812B Addressable LED Strip 5M | 1 | Digital GPIO | Programmable RGB lighting |
| Mini Submersible Pump (in kit) | 1 | Relay/GPIO | Automated plant watering |

### Prototyping Tools
| Item | Qty | Purpose |
|------|-----|---------|
| PiDuino Breadboard + Jumper Kit | 1 | Circuit prototyping |
| 60W Soldering Iron | 1 | Permanent connections |
| Spigen USB-A to C Cable 1M (2-pack) | 1 | Power/data for ESP32 |

### Plant Watering Kit (All-in-One)
| Item | Qty | Purpose |
|------|-----|---------|
| DIY Smart Plant Watering System Kit | 1 | Soil moisture + pump + relay + pipes |

## How Everything Connects

### ESP32-C6 Pinout for Projects

```
ESP32-C6-DevKitC-1
├── GPIO 1-10    → Digital sensors (PIR, Relay modules)
├── GPIO 11-20   → PWM outputs (Servo, LED strip)
├── GPIO 21-22   → I2C (BME280: SDA=GPIO21, SCL=GPIO22)
├── GPIO 23-25   → 1-Wire (DHT22)
├── ADC1_CH0-5   → Analog sensors (Soil moisture)
├── USB-C        → Power + Programming
└── WiFi 6       → UDP stream to Pi 5 + AWS IoT MQTT
```

### RuView CSI Streaming
- Each ESP32-C6 runs RuView firmware
- Time Division Multiplexing (TDM) coordinates 4 nodes
- UDP packets stream CSI data to Pi 5 on port 5005
- Rust Axum server processes per-node state
- React UI shows real-time presence detection

### IoT Automation Projects

#### 1. Smart Plant Watering
```
Soil Moisture Sensor ──► ESP32-C6 ──► Relay ──► Mini Water Pump
         │                                  │
         └──── Threshold < 40% ─────────────┘
```
- **Code**: Arduino IDE / MicroPython
- **Trigger**: Soil moisture < threshold
- **Action**: Relay ON → Pump runs for 5 seconds

#### 2. Motion-Activated Lights
```
HC-SR501 PIR ──► ESP32-C6 ──► Relay Module ──► AC Light/Fan
```
- **Code**: Arduino IDE / MicroPython
- **Trigger**: PIR HIGH (motion detected)
- **Action**: Relay ON for 2 minutes
- **Bonus**: Publish MQTT event to AWS IoT

#### 3. Environmental Dashboard
```
BME280 (I2C) ──► ESP32-C6 ──► WiFi ──► Raspberry Pi ──► Web UI
DHT22 (1-Wire) ──► ESP32-C6 ──► WiFi ──► Raspberry Pi ──► Web UI
```
- **Data**: Temperature, Humidity, Pressure
- **Display**: React dashboard on Pi 5
- **Alert**: Push notification if temp > 35°C

#### 4. WS2812B Ambient Lighting
```
ESP32-C6 ──► GPIO ──► WS2812B LED Strip
```
- **Code**: FastLED or NeoPixel library
- **Modes**: Static color, rainbow cycle, music sync (via microphone input)
- **Control**: Web UI or MQTT command

#### 5. Motorized Curtains / Fan
```
ESP32-C6 ──► L298N ──► DC Motor (curtains) or Stepper
ESP32-C6 ──► PWM ──► SG90 Servo (valves, locks, pan-tilt)
```
- **Control**: Web UI buttons or automated schedules

## Customizability

Every component is **fully programmable**:

| Component | Programming Method | Customization |
|-----------|-------------------|---------------|
| ESP32-C6 | Arduino IDE, MicroPython, ESP-IDF | Full control over GPIO, WiFi, Bluetooth |
| Raspberry Pi 5 | Python, Rust, Node.js | Server, dashboard, ML inference |
| BME280/DHT22 | I2C/1-Wire libraries | Read intervals, thresholds, alerts |
| HC-SR501 | Digital read | Sensitivity, delay time via potentiometers |
| L298N | PWM + digital | Speed, direction, braking |
| SG90 Servo | PWM (50Hz, 0.5-2.5ms pulse) | Angle 0-180°, speed control |
| Relay Module | Digital write | Timing, sequencing, safety interlocks |
| WS2812B | FastLED/NeoPixel | Colors, patterns, animations, brightness |
| Soil Moisture | Analog read | Calibration, watering schedules |

### WiFi / Bluetooth Capabilities
- **WiFi 6 (802.11ax)**: High-speed data, CSI sensing, MQTT to AWS
- **Bluetooth 5**: BLE sensors, beacon detection, phone pairing
- **802.15.4**: Zigbee/Thread mesh networking (future expansion)

## AWS IoT Integration (Future)

```
ESP32-C6 ──► WiFi ──► AWS IoT Core (MQTT)
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    AWS Lambda      AWS Timestream    AWS SNS
    (Rules)         (Metrics DB)      (Alerts)
          │               │               │
          └───────────────┴───────────────┘
                          │
                    React Dashboard
                    (Amplify Hosting)
```

**Use Cases**:
- Store sensor data in Timestream
- Lambda functions for anomaly detection
- SNS alerts for critical events (fire, flood)
- DynamoDB for device state
- S3 for firmware OTA updates

## RuView Local Setup

### Prerequisites
- Docker + Docker Compose
- Rust toolchain
- Node.js 20+
- Python 3.11+

### Quick Start
```bash
git clone https://github.com/ruvnet/RuView.git
cd RuView/v2

# Start services
docker compose -f docker/docker-compose.yml up -d

# Run Rust tests
cargo test -p homecore-api

# Build UI
cd ui && npm install && npm run dev

# Access UI
open http://localhost:3000/ui/
```

### ESP32 Firmware
```bash
cd esp32-firmware
idf.py set-target esp32c6
idf.py build
idf.py flash
```

## Project Roadmap

- [ ] Set up Raspberry Pi 5 with Docker
- [ ] Flash 4x ESP32-C6 with RuView firmware
- [ ] Verify CSI streaming and presence detection
- [ ] Build environmental sensor node (BME280 + DHT22)
- [ ] Build motion-activated light controller
- [ ] Build smart plant watering system
- [ ] Deploy WS2812B ambient lighting
- [ ] Motor control experiments (L298N + SG90)
- [ ] AWS IoT Core integration
- [ ] Mobile app for remote control

## Links

- [RuView Original Repo](https://github.com/ruvnet/RuView)
- [ESP32-C6 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-c6_datasheet_en.pdf)
- [Raspberry Pi 5 Specs](https://www.raspberrypi.com/products/raspberry-pi-5/)
- [AWS IoT Core Docs](https://docs.aws.amazon.com/iot/)

---

**Note**: Do NOT purchase until ready. Cart is ready for review. All items are WiFi/Bluetooth capable and fully customizable via open-source tools.
