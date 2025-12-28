# DIY AIR PURIFIER — AUTO PM2.5 VERSION

Advanced purifier firmware with **PM2.5 sensor and automatic fan control**.
Built on ESP32 + ESPHome (CLI only).

This is a **separate firmware** from the manual version.

---

## FEATURES

Everything from the Manual Version **plus**:

- PMS7003 PM2.5 laser sensor
- AQI calculation (US EPA)
- **Auto fan mode**
- Physical mode button
- OLED shows PM2.5 & AQI
- Persistent mode + speed
- Web UI with mode selection

---

## ADDITIONAL HARDWARE

| Component | Notes |
|:---------|:------|
| PMS7003 | PM2.5 sensor |
| Push button | Mode cycling |

---

## PMS7003 WIRING

| PMS7003 | ESP32 |
|:-------:|:-----:|
| VCC | 5V |
| GND | GND |
| TX | GPIO16 |
| RX | GPIO17 |

---

## MODE BUTTON WIRING

| Button | ESP32 |
|:------:|:------:|
| One side | GPIO18 |
| Other side | GND |

Configured as `INPUT_PULLUP`.

---

## FAN MODES

| Mode | Behavior |
|:----:|:--------:|
| Quiet | 25% speed |
| Normal | 60% speed |
| Max | 100% speed |
| Auto | PM2.5-based |

---

## AUTO FAN CURVE (DEFAULT)

| PM2.5 (µg/m³) | Fan Speed |
|:-------------:|:---------:|
| < 25 | 25% |
| 25–50 | 40% |
| 50–100 | 60% |
| 100–150 | 80% |
| > 150 | 100% |

Evaluated every **30 seconds**.

---

## PHYSICAL BUTTON LOGIC

Each press cycles:

Quiet → Normal → Max → Auto → Quiet

Mode is saved to flash.

---

## OLED DISPLAY (AUTO VERSION)

```
DIY PURIFIER AUTO
IP: 192.168.1.17
Mode: Auto
PM2.5: 82.3
AQI: 164
```

---

## SOFTWARE SETUP (CLI ONLY)

```bash
pip install esphome
mkdir purifier
cd purifier
nano dotpwm_auto.yaml
# paste the auto firmware YAML (dotpwm_auto.yaml)

# FLASHING (USB)
esphome run dotpwm_auto.yaml

# OTA
esphome run dotpwm_auto.yaml --device <DEVICE_IP>
```

---

## OPERATING LOGIC

- Restores last mode at boot and applies corresponding fan behavior.
- Auto mode: fan adjusts based on PM2.5; manual override available anytime.
- Interval check every 30s when in Auto mode.

---

## RELIABILITY NOTES

- PWM pin avoids ESP32 boot conflicts.
- Persistent globals prevent state loss.
- Designed for continuous operation.

---

## DESIGN GOAL

Deterministic control, sensor-driven automation, local diagnostics, and no cloud dependency.

