# RoboDev

An open-source ESP32-S3 robotics carrier board. Plug in power, plug in your motors and sensors — that's it. No breadboard, no stacked breakout boards, no wiring harness.

> **Status:** v0.2.1 design spec complete · PCB layout not yet started

---

## What's on the board

| | |
|---|---|
| **MCU** | ESP32-S3 DevKitC-1 v1.1 in removable dual-row headers |
| **DC motors** | 4 channels · DRV8874 · up to 37 V · 2.1 A cont. / 6 A peak |
| **Servos** | 16 channels · PCA9685 · adjustable 5–8.4 V rail |
| **Steppers** | 2× NEMA 17 · TMC2209 · StealthChop2 · silent operation |
| **CAN bus** | TWAI hardware · SN65HVD230 · ISO 11898-1 · up to 1 Mbps |
| **IMU** | MPU-6050 GY-521 · 6-axis accel + gyro |
| **Analogue in** | 9 total: 5× native ADC1 (12-bit) + 4× ADS1115 (16-bit I²C) |
| **Digital I/O** | 19 headers: 8 direct GPIO + 8× PCF8574 expander + 3 freed pins |
| **Encoders** | 2 channels · hardware PCNT quadrature decode |
| **I²C expansion** | 16 independent ports · 6× Qwiic + 10× 2.54mm · 2× TCA9548A mux |
| **Power input** | ≥6 V · screw terminal + XT30 · reverse voltage protection |
| **Est. BOM cost** | ~NZ$90–92 (qty 1, excl. assembly/shipping) |

---

## Motor compatibility

| Motor | Works? | Notes |
|---|---|---|
| TT yellow gearmotors (2WD/4WD) | ✓ | Ideal fit |
| Pololu micro/mini gearmotors | ✓ | Well within limits |
| 25–37 mm DC gearmotors | ✓ | Software current limiting recommended near stall |
| Hobby servos | ✓ | Up to 16 channels via PCA9685 |
| NEMA 17 steppers | ✓ | TMC2209 · 2 A RMS / 2.8 A peak |
| CAN motor controllers (VESC, ODrive) | ✓ | Via TWAI hardware CAN |
| 775-size motors | ✗ | Exceeds DRV8874 — use an external driver |

---

## Power

Five independent rails from a single protected VIN input. Vmot-DC and Vstep are separate adjustable bucks so brushed DC motors and steppers can run at different voltages simultaneously.

```
VIN  →  reverse voltage protection  →  slide switch
          ├── Vmot-DC   adj. 4.5–24 V  →  DC motors
          ├── Vstep     adj. 4.5–24 V  →  steppers
          ├── Vservo    adj. 5–8.4 V   →  servos
          └── 5 V buck  →  3.3 V LDO   →  ESP32 + all logic
```

---

## I²C expansion (new in v0.2.1)

16 independent I²C ports behind two TCA9548A mux ICs — at zero GPIO cost, sharing the existing I²C bus. Each port is an isolated bus segment, so you can connect multiple devices with the same address across different ports without conflict.

On startup, the firmware HAL scans all 16 ports and builds a device map automatically. Application code never touches the mux directly — just ask for `i2c_port_N` and the HAL handles the rest.

6 ports are Qwiic (JST SH), 10 are standard 2.54 mm 4-pin headers. All carry GND / 3.3V / SDA / SCL.

---

## GPIO

All 45 ESP32-S3 GPIOs are allocated. Zero spares remain.

| Block | GPIOs used |
|---|---|
| DC motors (4 ch · PWM + DIR) | GPIO10–17 |
| Stepper STEP/DIR/EN (2 ch) | GPIO26–31 |
| Quadrature encoders (2 ch) | GPIO39–42 |
| I²C bus | GPIO8 (SDA), GPIO9 (SCL) |
| CAN bus TX/RX | GPIO7, GPIO32 |
| UART1 header | GPIO47 (TX), GPIO21 (RX) |
| Native ADC1 headers A1–A5 | GPIO1, 2, 4, 5, 6 |
| Direct digital headers | GPIO18, 33, 34 |

GPIO38 and GPIO48 are written off — they're hardwired to the WS2812B RGB LED on the DevKitC-1 v1.1.

---

## I²C address map

| Address | Device |
|---|---|
| 0x20 | PCF8574 — digital GPIO expander |
| 0x3C | SSD1306 OLED (user-supplied) |
| 0x40 | PCA9685 — servo PWM driver |
| 0x48 | ADS1115 — 16-bit ADC |
| 0x68 | MPU-6050 — IMU |
| 0x70 | TCA9548A Mux #1 — expansion ports 0–7 |
| 0x71 | TCA9548A Mux #2 — expansion ports 8–15 |

---

## Project status

- [x] Design spec v0.2.1 complete
- [ ] KiCad schematic — in progress (start with power system)
- [ ] PCB layout
- [ ] Bringup firmware (I²C scan, PWM test, CAN loopback, UART echo)
- [ ] TCA9548A reverse-ARP HAL
- [ ] MPU-6050 Kalman filter
- [ ] Full LCSC/Mouser BOM with pricing

---

## Bill of materials

Full BOM in [`BOM.xlsx`](./BOM.xlsx). Estimated ~NZ$90–92 per board at qty 1, mid-range pricing, excluding assembly, shipping, and duties.

Key ICs: ESP32-S3 DevKitC-1 v1.1 · DRV8874 ×4 · TMC2209 ×2 · PCA9685 · ADS1115 · PCF8574 · TCA9548A ×2 · SN65HVD230 · MPU-6050 GY-521 · AO3401

---

## Repo structure

```
/
├── hardware/        KiCad schematic and PCB files
├── firmware/        ESP-IDF firmware
├── docs/            Design spec, layout notes, datasheets
├── BOM.xlsx         Bill of materials
└── README.md
```

---

## Contributing

This is an early-stage open hardware project — no PCB has been fabricated yet. If you want to follow along or contribute, the most useful things right now are:

- KiCad schematic review once the first draft is up
- Firmware: ESP-IDF bringup routines, TCA9548A HAL, TMC2209 UART config
- Breadboard prototyping of the I²C bus with all 7 devices simultaneously

---

*ESP32-S3 Integrated Robotics Carrier Board · v0.2.1 · March 2026*