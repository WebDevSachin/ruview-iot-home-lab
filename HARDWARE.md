# Hardware Guide

## What Was Removed from Cart

The following items were **removed** because they were duplicates, pre-built non-programmable controllers, or not compatible with our ESP32-based architecture:

### Removed: Pre-Built LED Controllers
- **Auslese RGB Pixel LED Controller** (IR remote) — Pre-built, not programmable by ESP32
- **Auslese Robotics SP621E Bluetooth LED Controller** — Pre-built, not programmable by ESP32

### Removed: Overpriced LED Strip
- **Rongda Smart Home 5M WS2812B** — ₹9,470 for 5M strip was excessive

### Removed: Duplicate Motor Drivers (kept 1)
- Tinkro L298N
- Self Lub L298N
- SmartElex L298N (had onboard Arduino Uno — not what we need)

### Removed: Duplicate Plant Watering Kit
- E IDEA ROBOTICS Automatic Plant Watering Kit

### Removed: Duplicate Servos
- Super Debug (6 Pcs) SG90

### Removed: Excess USB Cables (kept 1 good pack)
- UGREEN 0.5M 2-pack
- Umake USB A to C
- UGREEN USB 3.0 5Gbps
- Verilux 8M/26Ft
- UGREEN 1M Nylon Braided

### Removed: Duplicate Breadboard Kit
- Tishvi Jumper Wires Kit + Breadboard

### Removed: Duplicate Relay
- Robocraze 2-Channel Relay

### Removed: AC-Powered Motion Sensors (not ESP32-compatible)
- ConnectedHome 360° PIR (12M coverage)
- ConnectedHome Mini Motion Sensor (In-ceiling)
- RITRON Wall Mounted PIR

### Removed: Sensor Combos / Duplicates
- TESTIN ELECTRONICS Super Combo (PIR + LDR + resistors)
- Hexonix DHT22
- FlyRobo BME280

### Removed: Duplicate Soldering Iron
- Party Town Professional Soldering Iron Kit

## Final Cart: 16 Items

| # | Item | Qty | Price (₹) | Why We Need It |
|---|------|-----|-----------|----------------|
| 1 | Raspberry Pi 5 8GB Starter Kit | 1 | 22,880.51 | Hub: RuView server, Docker, UI |
| 2 | ESP32-C6-DevKitC-1-N8 | 4 | 8,305.08 | WiFi 6 / BT 5 nodes for CSI + IoT |
| 3 | WS2812B LED Strip 5M | 1 | 1,142.37 | Programmable RGB lighting |
| 4 | L298N Motor Driver | 1 | 168.64 | DC motor / stepper control |
| 5 | DIY Plant Watering Kit | 1 | 209.32 | Soil moisture + pump + relay |
| 6 | SG90 Servo Motors | 2 | 262.71 | Mechanical movement |
| 7 | Spigen USB-A to C 1M (2-pack) | 1 | 422.88 | Power/data cables |
| 8 | PiDuino Breadboard + Jumpers | 1 | 300.85 | Circuit prototyping |
| 9 | Robocraze 4-Ch Relay Module | 1 | 216.95 | Switch AC devices |
| 10 | HC-SR501 PIR Motion Sensor | 1 | 237.28 | Motion detection |
| 11 | HiLetgo DHT22 | 1 | 342.90 | Temperature / humidity |
| 12 | Allianztec BME280 | 1 | 520.24 | Temp / humidity / pressure |
| 13 | Party Town 60W Soldering Iron | 1 | 570.48 | Soldering connections |

**Total: ~₹41,798**

## WiFi / Bluetooth Capability Check

| Item | WiFi | Bluetooth | Programmable | Notes |
|------|------|-----------|--------------|-------|
| Raspberry Pi 5 | ✅ WiFi 6E | ✅ BT 5.0 | ✅ Full Linux | Runs our server |
| ESP32-C6 | ✅ WiFi 6 | ✅ BT 5.0 | ✅ Arduino/MicroPython/IDF | Main controller |
| BME280 | ❌ | ❌ | ✅ Via ESP32 I2C | Sensor only |
| DHT22 | ❌ | ❌ | ✅ Via ESP32 1-Wire | Sensor only |
| HC-SR501 | ❌ | ❌ | ✅ Via ESP32 GPIO | Sensor only |
| Relay Module | ❌ | ❌ | ✅ Via ESP32 GPIO | Actuator only |
| L298N | ❌ | ❌ | ✅ Via ESP32 PWM | Actuator only |
| SG90 Servo | ❌ | ❌ | ✅ Via ESP32 PWM | Actuator only |
| WS2812B | ❌ | ❌ | ✅ Via ESP32 GPIO | Actuator only |
| Soil Moisture | ❌ | ❌ | ✅ Via ESP32 ADC | Sensor only |

**All sensors/actuators connect to ESP32, which has WiFi 6 + Bluetooth 5. This makes every item network-connected and fully customizable.**
