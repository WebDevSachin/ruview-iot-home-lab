# Setup Guide

## Phase 1: Raspberry Pi 5 Hub Setup

### 1.1 Flash microSD Card
- Use Raspberry Pi Imager
- Select Raspberry Pi OS 64-bit (Bookworm)
- Enable SSH and set username/password

### 1.2 Install Dependencies
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Install Python
sudo apt install -y python3-pip python3-venv

# Reboot
sudo reboot
```

### 1.3 Clone and Run RuView
```bash
git clone https://github.com/ruvnet/RuView.git
cd RuView/v2

# Build and start Docker services
cd docker
docker compose up -d

# Verify
docker compose ps
```

### 1.4 Fix CORS for Local UI
If UI shows "Backend unavailable":
```bash
# Edit ui/config/api.config.js
# Change base URL to: const _origin = 'http://localhost:3000'
# Change WS host to: const host = 'localhost:3001'

# Rebuild UI
cd ../ui
npm install
npm run build
```

## Phase 2: ESP32-C6 Node Setup (x4)

### 2.1 Install ESP-IDF
```bash
mkdir -p ~/esp
cd ~/esp
git clone -b v5.2 --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh esp32c6
. ./export.sh
```

### 2.2 Flash RuView Firmware
```bash
cd ~/RuView/esp32-firmware
idf.py set-target esp32c6
idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
```

### 2.3 Configure WiFi Credentials
In `menuconfig` or `sdkconfig`:
```
CONFIG_WIFI_SSID="YourHomeWiFi"
CONFIG_WIFI_PASSWORD="YourPassword"
CONFIG_RUVIEW_SERVER_IP="192.168.1.100"  # Pi 5 IP
CONFIG_RUVIEW_SERVER_PORT=5005
```

### 2.4 Assign Node IDs
Flash each ESP32 with a unique node ID:
- Node 1: `CONFIG_NODE_ID=1` (Living Room)
- Node 2: `CONFIG_NODE_ID=2` (Bedroom)
- Node 3: `CONFIG_NODE_ID=3` (Kitchen)
- Node 4: `CONFIG_NODE_ID=4` (Hallway)

## Phase 3: Sensor Wiring

### 3.1 BME280 (I2C)
```
BME280 Pin    ESP32-C6 Pin
--------      -----------
VCC           3.3V
GND           GND
SCL           GPIO 22
SDA           GPIO 21
```

### 3.2 DHT22 (1-Wire)
```
DHT22 Pin     ESP32-C6 Pin
--------      -----------
VCC           3.3V
GND           GND
DATA          GPIO 4
```

### 3.3 HC-SR501 PIR
```
PIR Pin       ESP32-C6 Pin
--------      -----------
VCC           5V
GND           GND
OUT           GPIO 5
```

### 3.4 4-Channel Relay
```
Relay Pin     ESP32-C6 Pin
--------      -----------
VCC           5V
GND           GND
IN1           GPIO 12
IN2           GPIO 13
IN3           GPIO 14
IN4           GPIO 15
```

### 3.5 WS2812B LED Strip
```
LED Pin       ESP32-C6 Pin
--------      -----------
5V            5V (external PSU recommended)
GND           GND
DATA          GPIO 8
```

### 3.6 L298N Motor Driver
```
L298N Pin     ESP32-C6 Pin
--------      -----------
ENA           GPIO 18 (PWM)
IN1           GPIO 19
IN2           GPIO 20
GND           GND
+5V           5V
```

### 3.7 SG90 Servo
```
Servo Pin     ESP32-C6 Pin
--------      -----------
VCC (Red)     5V
GND (Brown)   GND
Signal (Orange) GPIO 16
```

### 3.8 Plant Watering Kit
```
Soil Sensor   ESP32-C6 Pin
--------      -----------
VCC           3.3V
GND           GND
AO            GPIO 1 (ADC)

Pump          Relay Module
----          ------------
+             Relay COM
-             Power supply GND
```

## Phase 4: Arduino IDE Setup (Alternative to ESP-IDF)

For simpler IoT projects, use Arduino IDE:

1. **Add ESP32 Board Manager URL**:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```

2. **Install Board**: Tools → Board → ESP32 Arduino → ESP32C6 Dev Module

3. **Install Libraries**:
   - `Adafruit BME280 Library`
   - `DHT sensor library`
   - `FastLED`
   - `PubSubClient` (for MQTT)

4. **Example Sketch** (Motion Light):
   ```cpp
   #define PIR_PIN 5
   #define RELAY_PIN 12

   void setup() {
     pinMode(PIR_PIN, INPUT);
     pinMode(RELAY_PIN, OUTPUT);
     Serial.begin(115200);
   }

   void loop() {
     if (digitalRead(PIR_PIN) == HIGH) {
       digitalWrite(RELAY_PIN, HIGH);
       Serial.println("Motion detected! Light ON");
       delay(120000); // 2 minutes
       digitalWrite(RELAY_PIN, LOW);
     }
     delay(100);
   }
   ```

## Phase 5: AWS IoT Integration

### 5.1 Create Thing in AWS IoT Core
```bash
# Install AWS CLI
pip install awscli
aws configure

# Create thing
cd ~/ruview-iot-home-lab/aws
bash create-thing.sh esp32-node-1
```

### 5.2 Flash Certificates to ESP32
```bash
# Convert certificates to Arduino format
openssl x509 -in certificate.pem.crt -out certificate.der -outform DER
openssl rsa -in private.pem.key -out private.der -outform DER

# Upload via SPIFFS or embed in firmware
```

### 5.3 MQTT Publish Example
```cpp
#include <WiFi.h>
#include <PubSubClient.h>

const char* mqtt_server = "your-endpoint.iot.ap-south-1.amazonaws.com";

void publishSensorData(float temp, float humidity) {
  String payload = "{\"temperature\":" + String(temp) +
                   ",\"humidity\":" + String(humidity) + "}";
  client.publish("home/sensors/living-room", payload.c_str());
}
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| ESP32 not detected | Hold BOOT button while clicking RESET |
| Docker containers exit | Check `docker logs` for missing env vars |
| UI shows "Backend unavailable" | Hardcode API URL in `api.config.js` |
| CORS errors | Ensure UI and API are same origin, or configure `homecore_cors_origins` |
| Sensor reads 0 | Check wiring, pull-up resistors for I2C |
| Relay not switching | Ensure 5V power, check transistor direction |
| WS2812B flickering | Add 470Ω resistor on data line, capacitor across power |

## Next Steps

1. [ ] Complete Phase 1 (Pi setup)
2. [ ] Flash first ESP32-C6 with RuView firmware
3. [ ] Verify CSI data appears in UI
4. [ ] Wire BME280 and test environmental readings
5. [ ] Build first automation: Motion → Light
6. [ ] Expand to all 4 nodes
7. [ ] Add AWS IoT data streaming
