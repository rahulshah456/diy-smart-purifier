# DIY AIR PURIFIER — MANUAL VERSION

A production-grade DIY air purifier controller using ESP32 + ESPHome (CLI only).
This version is **manual control only** — no air quality sensor.

Designed for stability, predictable behavior, and 24/7 operation.

---

## FEATURES

- Manual fan modes: **Quiet / Normal / Max**
- PWM control of 4-pin BLDC fan (25 kHz)
- Accurate RPM feedback (tachometer)
- Persistent fan speed (survives reboot & power loss)
- OLED (I²C) status screen
- Web UI (local)
- BLE fallback control (optional)
- No Home Assistant dependency

---

## QUICK LINK

For the feature-rich automatic PM2.5 version, see: [README-auto.md](README-auto.md)

---

## HARDWARE REQUIRED

### Core
- ESP32 Dev Board (ESP32-WROOM-32)
- 4-pin PWM BLDC fan (PC fan)
- 12V power supply (**minimum 3A, recommended 5A**)
- DC buck converter (12V → 5V)
- 1000 µF electrolytic capacitor

### Display
- SSD1306 OLED (128×64, I²C)

---

### Schematics
<img width="1564" height="846" alt="schematics default" src="https://github.com/user-attachments/assets/ca60616b-848c-4f2f-a2e9-5eb35d1d18fa" />


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

12V PSU
├── Fan (direct 12V)
├── Buck → 5V → ESP32 VIN/5V
└── 1000µF capacitor (close to ESP32)

All grounds must be common.

---

## OLED WIRING

| OLED Pin | ESP32 |
|:--------:|:-----:|
| GND | GND |
| VCC | 3.3V |
| SDA | GPIO21 |
| SCL | GPIO22 |

---

## GPIO ASSIGNMENT (SAFE)

| GPIO | Purpose |
|:-----:|:--------|
| GPIO25 | PWM output |
| GPIO27 | Fan tach |
| GPIO21 | I²C SDA |
| GPIO22 | I²C SCL |

❌ **Never use GPIO12** (boot pin).

---

## SOFTWARE SETUP (CLI ONLY)

### Install ESPHome
```bash
pip install esphome
```

### Create project
```bash
mkdir purifier
cd purifier
nano dotpwm.yaml
# paste the manual firmware YAML (dotpwm.yaml)
```

### FIRST FLASH (USB)
```bash
esphome run dotpwm.yaml
```

### OTA UPDATES
```bash
esphome run dotpwm.yaml --device <DEVICE_IP>
```

---

## OPERATING LOGIC

- Boot sequence: delay 2s for power stabilization, then restore last saved fan speed.
- Speed persistence: changes are saved to flash and restored after reboot.
- OLED shows IP, fan %, RPM, and uptime.

Example OLED display:

```
DIY AIR PURIFIER
IP: 192.168.1.17
Fan: 60%
RPM: 2450
Up: 00:14:32
```

---

## USE CASE

Always-on purifier where noise predictability matters and maximum reliability is desired.

---

## TROUBLESHOOTING

- Fan speed changes unexpectedly: check uptime (indicates reboot). Persistent state should restore previous speed.
- ESP32 fails to boot: ensure GPIO12 is unused and bulk capacitor is installed.

---

## DESIGN PHILOSOPHY

Deterministic behavior, delayed boot application, persistent state, and no sensor-driven surprises.

