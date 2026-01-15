# DIY AIR PURIFIER — ESPHome Configuration

Production-grade DIY air purifier controller using ESP32 + ESPHome (CLI only).

Two firmware versions available:
- **Manual** (`dotpwm.yaml`) — Manual control only, lightweight, simple
- **Automatic with PM2.5 Sensor** (`sensor_pwm_auto.yaml`) — Auto fan speed based on air quality, tested on real hardware

---

## QUICK START

### Prerequisites
```bash
pip install esphome
```

### Clone & Configure
```bash
git clone https://github.com/rahulshah456/diy-smart-purifier.git
```

### Choose Your Version

#### Option 1: Manual Version (No Sensor)
```bash
# Edit dotpwm.yaml - update WiFi SSID/password in the config
esphome run .\dotpwm.yaml
```

#### Option 2: Automatic with PM2.5 Sensor (Recommended)
```bash
# Edit sensor_pwm_auto.yaml - update WiFi SSID/password in the config
esphome run .\sensor_pwm_auto.yaml
```

---

## HARDWARE REQUIRED

### Core Components
- ESP32 Dev Board (ESP32-WROOM-32)
- 4-pin PWM BLDC fan (PC fan)
- 12V power supply (**minimum 3A, recommended 5A**)
- DC buck converter (12V → 5V)
- 1000 µF electrolytic capacitor

### Display
- SSD1306 OLED (128×64, I²C)

### For Automatic Version Only
- PMS9003M PM2.5 laser sensor
- Push button (mode cycling) — optional but recommended

---

## FAN CONNECTOR PINOUT

| Wire Color | Function | Connection |
|-----------:|:--------:|:-----------|
| Black | Ground | PSU GND |
| Red | +12V | PSU +12V |
| Blue | PWM | ESP32 GPIO25 |
| Yellow | Tach | ESP32 GPIO27 (+10k pull-up to 3.3V) |

⚠ Fan **must not** be powered from ESP32.

---

## POWER WIRING

```
12V PSU
├── Fan (direct 12V)
├── Buck → 5V → ESP32 VIN/5V
└── 1000µF capacitor (close to ESP32)

All grounds must be common.
```

---

## OLED WIRING (I²C)

| OLED Pin | ESP32 |
|:--------:|:-----:|
| GND | GND |
| VCC | 3.3V |
| SDA | GPIO21 |
| SCL | GPIO22 |

---

## SENSOR WIRING (Automatic Version Only)

### PMS9003M Sensor
| PMS9003M | ESP32 |
|:-------:|:-----:|
| VCC | 5V |
| GND | GND |
| TX | GPIO16 |
| RX | GPIO17 |

### Mode Button (Optional)
| Button | ESP32 |
|:------:|:------:|
| One side | GPIO14 |
| Other side | GND |

Configured as `INPUT_PULLUP`.

---

## GPIO ASSIGNMENT (SAFE)

| GPIO | Purpose | Version |
|:-----:|:--------|:-------:|
| GPIO25 | PWM output | Both |
| GPIO27 | Fan tach | Both |
| GPIO21 | I²C SDA | Both |
| GPIO22 | I²C SCL | Both |
| GPIO16 | UART RX (sensor) | Auto only |
| GPIO17 | UART TX (sensor) | Auto only |
| GPIO14 | Mode button | Auto only |
| GPIO13 | WS2812 LED strip | Auto only |

❌ **Never use GPIO12** (boot pin).

---

## FIRMWARE VERSIONS

### Manual Version (`dotpwm.yaml`)

**Features:**
- Three manual fan modes: QUIET (25%), NORMAL (60%), MAX (100%)
- PWM control at 25 kHz
- Accurate RPM feedback (tachometer)
- Persistent fan speed (survives reboot)
- OLED status screen
- Web UI control buttons
- Simple and stable

**OLED Display:**
```
DIY AIR PURIFIER
IP: 192.168.1.17
Fan: 60%
RPM: 2450
UP TIME: 00:14:32
```

**Use Case:** Always-on purifier where simplicity and predictability matter.

---

### Automatic Version with PM2.5 Sensor (`sensor_pwm_auto.yaml`)

**Features:** ✓ All manual features **plus**

- **PM2.5 laser sensor** (PMSX003/PMS9003M) for real-time air quality monitoring
- **Four operating modes:**
  - QUIET — 25% speed (silent operation)
  - NORMAL — 60% speed (balanced)
  - MAX — 100% speed (maximum cleaning)
  - AUTO — Dynamic speed based on PM2.5 readings (10-100%)
- **Auto Mode Fan Curve** (PM2.5 based):
  | PM2.5 (µg/m³) | Fan Speed |
  |:-------------:|:---------:|
  | ≤ 3 | 10% |
  | ≤ 6 | 20% |
  | ≤ 9 | 30% |
  | ≤ 18 | 40% |
  | ≤ 30 | 55% |
  | ≤ 40 | 70% |
  | ≤ 54 | 85% |
  | ≤ 75 | 90% |
  | ≤ 99 | 95% |
  | > 99 | 100% |

- **Physical mode button** cycles through all modes (GPIO14)
- **Web UI dropdown** for mode selection with real-time sync
- **RGB LED strip** (WS2812, 8 LEDs) shows air quality:
  - Green: PM2.5 ≤ 6 (Good)
  - Cyan: PM2.5 ≤ 9 (Fair)
  - Yellow: PM2.5 ≤ 18 (Moderate)
  - Orange: PM2.5 ≤ 30 (Unhealthy for Sensitive)
  - Red: PM2.5 ≤ 54 (Unhealthy)
  - Dark Red: PM2.5 ≤ 75 (Very Unhealthy)
  - Purple: PM2.5 ≤ 99 (Hazardous)
  - Pink: PM2.5 > 99 (Emergency)

- **All particle counts** (0.3μm to 10.0μm) displayed in web UI
- **Real-time sensor data** updated every 30 seconds
- **AQI Status** text indicator (Good/Fair/Moderate/Poor/Very Poor/Severe)
- **Persistent mode storage** across reboots

**OLED Display:**
```
Up Time: 00:14:32
Mode: AUTO
Speed: 65%  |  2150 RPM
PM25: 42 ug/m3
WIFI: 192.168.1.17
```

**Web UI Displays:**
- Purifier Mode (dropdown: QUIET/NORMAL/MAX/AUTO)
- PM2.5 reading (µg/m³)
- PM1.0 and PM10.0 readings
- Particle counts (0.3μm to 10.0μm in per liter)
- AQI Status (air quality level)
- Fan RPM
- Uptime (HH:MM:SS)
- WiFi IP address
- AQI LED switch (on/off)

**Test Status:** ✓ Tested and validated on real hardware

---

## SOFTWARE SETUP

### Install ESPHome
```bash
pip install esphome
```

### Clone Repository
```bash
git clone https://github.com/rahulshah456/diy-smart-purifier.git
cd diy-smart-purifier
```

### Configure WiFi
Edit your chosen firmware file and update credentials:

Both YAML files need WiFi configuration. Update these sections in your chosen firmware:

**In `dotpwm.yaml` or `sensor_pwm_auto.yaml`:**

Find the WiFi section and update:
```yaml
wifi:
  ssid: "Your_WiFi_SSID"           # ← Change to your WiFi name
  password: "Your_WiFi_Password"    # ← Change to your WiFi password
  
  # Fallback AP for configuration
  ap:
    ssid: "Dotpwm Fallback"
    password: "82jNkVOU8RN1"       # ← Optional: change if desired
```

**Also update OTA password if desired:**
```yaml
ota:
  - platform: esphome
    password: "your_secure_password"  # ← Change this for security
```

### FIRST FLASH (USB Connection)

**Manual Version:**
```bash
esphome run .\dotpwm.yaml
```

**Automatic Version:**
```bash
esphome run .\sensor_pwm_auto.yaml
```

Follow on-screen prompts to select COM port and device.

### OTA UPDATES (Over-The-Air)

Once connected to WiFi:

**Manual:**
```bash
esphome run .\dotpwm.yaml --device <DEVICE_IP>
```

**Automatic:**
```bash
esphome run .\sensor_pwm_auto.yaml --device <DEVICE_IP>
```

Replace `<DEVICE_IP>` with your ESP32's IP address (visible on OLED).

---

## OPERATING LOGIC

### Manual Version
1. Boot sequence: delay 2s for power stabilization
2. Restores last saved fan speed from flash
3. User controls via web UI buttons or CLI tools
4. Speed changes persist across reboots

### Automatic Version
1. Boot sequence: delay 2s, restores last saved mode from flash
2. If mode is AUTO: waits for first PM2.5 sensor reading (up to 30s)
3. In AUTO mode: fan speed adjusts automatically every 30 seconds based on latest PM2.5 reading
4. User can override by changing mode via:
   - Physical button (GPIO14) — cycles through all modes
   - Web UI dropdown — instant mode switch
5. Mode and speed persist across reboots
6. RGB LED updates in real-time as air quality changes

---

## CONNECTIVITY & ACCESS

### Local Web UI
- **URL:** `http://<device-ip>/`
- **Port:** 80

Find device IP:
- Check OLED display (shown in real-time)
- Check WiFi router DHCP list
- Use: `esphome logs` to see serial output

### WiFi Fallback AP
If main WiFi is unavailable:
- **SSID:** "Dotpwm Fallback"
- **Password:** "82jNkVOU8RN1" (configure in YAML)

---

## TROUBLESHOOTING

### Fan doesn't turn on
1. Check GPIO25 wiring (PWM output)
2. Verify 12V supply to fan (should be direct from PSU, not ESP32)
3. Check tach connection (GPIO27)

### ESP32 fails to boot
1. Ensure GPIO12 is unused (boot configuration pin)
2. Install 1000µF capacitor close to ESP32
3. Check power supply (minimum 3A)

### OLED display blank
1. Verify I²C wiring (GPIO21 SDA, GPIO22 SCL)
2. Check I²C address: should be 0x3C
3. Ensure 3.3V power to OLED

### PM2.5 sensor not reading (Automatic version)
1. Check UART wiring (GPIO16 RX, GPIO17 TX)
2. Verify 5V power to sensor (not 3.3V)
3. Check baud rate: 9600
4. Look for "Start character mismatch" warnings in logs — indicates pin swap

### Web UI dropdown out of sync
1. Reload the webpage in browser (hard refresh: Ctrl+F5)
2. Physical button press syncs automatically
3. Power cycle if still stuck

### Uptime resets unexpectedly
1. Indicates unintended reboot (voltage drop, brown-out)
2. Check power supply capacity
3. Verify capacitor connection

---

## LOGGING & DEBUGGING

### View Logs
```bash
esphome logs .\dotpwm.yaml
# or
esphome logs .\sensor_pwm_auto.yaml
```

### Log Levels
- Manual version: Standard INFO level
- Automatic version: ERROR level for UART (suppresses non-critical sync warnings)

Adjust in YAML:
```yaml
logger:
  level: INFO
```

---

## FILE STRUCTURE

```
esphome/
├── dotpwm.yaml              # Manual firmware (no sensor)
├── sensor_pwm_auto.yaml     # Automatic firmware with PM2.5 sensor (recommended)
├── roboto.ttf               # Font for OLED display
├── README.md                # This file
└── .esphome/                # ESPHome cache (auto-generated)
```

---

## DESIGN PHILOSOPHY

**Manual Version:** Deterministic behavior, simple state management, no sensor surprises.

**Automatic Version:** Sensor-driven automation, real-time air quality response, user control always available, no cloud dependency.

---

## USE CASES

| Version | Best For |
|:--------|:---------|
| **Manual** | Quiet, always-on operation; predictable noise levels; simplicity matters |
| **Automatic** | Health-conscious users; real-time air quality awareness; auto-optimize energy use |

---

## ADVANCED: Custom Fan Curve

To modify the auto fan speed curve in `sensor_pwm_auto.yaml`, edit the PM2.5 threshold logic:

Find section: **"PM2.5 - Primary sensor for auto mode control"**

Locate this code:
```cpp
if (pm <= 3) speed = 10;
else if (pm <= 6) speed = 20;
else if (pm <= 9) speed = 30;
// ... etc
```

Adjust thresholds and speeds to your preference. Save and re-flash.

---

## NOTES

- Both versions tested and stable
- No Home Assistant required
- Requires ESPHome CLI (desktop/laptop with Python)
- Local control only (no cloud)
- WiFi configuration in YAML required before flashing

---

## SUPPORT

For issues:
1. Check TROUBLESHOOTING section above
2. Review ESPHome logs (`esphome logs` command)
3. Verify all GPIO connections match documentation
4. Ensure correct YAML file is being flashed

