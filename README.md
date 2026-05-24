# RoboDev

An open-source STM32-based robotics carrier board with integrated WiFi. Plug in power, plug in your motors and sensors — that's it. No breadboard, no stacked breakout boards, no wiring harness.

> **Status:** Schematic substantially complete · PCB layout not yet started

---

## Architecture

RoboDev uses a two-MCU design:

| | |
|---|---|
| **Primary MCU** | STM32G431RBTx · Cortex-M4F 170 MHz · motor control, real-time I/O |
| **WiFi co-processor** | ESP32-C3-MINI-1-N4 · SPI-connected · handles wireless comms |

---

## What's on the board

| | |
|---|---|
| **DC motors** | 4 channels · DRV8874PWPR · up to ~40 V · ~5 A cont. |
| **Servos** | 16 channels · PCA9685PW · 5 V servo rail |
| **Steppers** | 2× NEMA 17 · TMC2209-LA-T · StealthChop2 · silent operation |
| **CAN bus** | SN65HVD230D · ISO 11898-1 · screw terminal + 5.08 mm connector |
| **IMU** | BMI270 · 6-axis accel + gyro · SPI interface |
| **16-bit ADC** | ADS1115IDGST · 4-channel · I²C |
| **Native ADC** | 3× 2.54 mm headers · STM32 ADC |
| **GPIO expander** | PCF8574DW · 8 additional I/O · I²C |
| **Encoders** | 2 channels · 2.54 mm headers |
| **I²C expansion** | 16 independent ports · mix of QWIIC (JST SH) + 2.54 mm · 2× TCA9548APWR mux |
| **USB** | Type-C · USB4085-GF-A |
| **SWD debug** | 2.54 mm header |
| **UART** | 2× headers (debug + general) |
| **Power input** | XT60PW-M · reverse polarity protection · slide switch |

---

## Motor compatibility

| Motor | Works? | Notes |
|---|---|---|
| TT yellow gearmotors (2WD/4WD) | ✓ | Ideal fit |
| Pololu micro/mini gearmotors | ✓ | Well within limits |
| 25–37 mm DC gearmotors | ✓ | Software current limiting recommended near stall |
| Hobby servos | ✓ | Up to 16 channels via PCA9685 |
| NEMA 17 steppers | ✓ | TMC2209 · 2 A RMS / 2.8 A peak |
| CAN motor controllers (VESC, ODrive) | ✓ | Via SN65HVD230D CAN transceiver |
| 775-size / high-current motors | ✗ | Exceeds DRV8874 — use an external driver |

---

## Power

Five independent rails from a single XT60 input with reverse polarity protection (IRF4905 P-channel MOSFET) and a slide switch.

```
XT60  →  IRF4905 (reverse protect)  →  slide switch
           ├── VMOT    adj. buck (XL4016)  →  DC motors
           ├── VSTEP   adj. buck (XL4016)  →  steppers
           ├── +5V     LM2596T-5           →  servo rail + USB
           └── +3V3    AP2112K-3.3 LDO     →  STM32 + ESP32-C3 + all logic
```

VMOT and VSTEP are separate adjustable bucks so DC motors and steppers can run at different voltages simultaneously.

---

## I²C expansion

16 independent I²C ports behind two TCA9548APWR mux ICs — at zero GPIO cost, sharing the existing I²C bus. Each port is an isolated bus segment, so you can connect multiple devices with the same address across different ports without conflict.

3 ports are QWIIC (JST SH), the rest are standard 2.54 mm 4-pin headers. All carry GND / 3.3V / SDA / SCL.

---

## I²C address map

| Address | Device |
|---|---|
| 0x20 | PCF8574DW — digital GPIO expander |
| 0x3C | OLED display (user-supplied, optional) |
| 0x40 | PCA9685PW — 16-channel servo PWM |
| 0x48 | ADS1115IDGST — 16-bit ADC |
| 0x70 | TCA9548A Mux #1 — expansion ports 0–7 |
| 0x71 | TCA9548A Mux #2 — expansion ports 8–15 |

> BMI270 IMU is on SPI (via ESP32-C3), not I²C.

---

## Schematic structure

The KiCad project uses a hierarchical multi-sheet design:

| Sheet | Contents |
|---|---|
| Root | Top-level interconnect, mounting holes |
| `power` | XT60 input, reverse protect, all buck converters, LDO |
| `stm32` | STM32G431RBTx, SN65HVD230D CAN, USB-C, SWD, UART, ADC headers, encoders |
| `spi_bus` | ESP32-C3-MINI-1-N4 WiFi module, BMI270 IMU, 74HC595 shift register |
| `i2c_bus` | TCA9548A ×2 muxes, ADS1115 ADC, PCF8574 GPIO expander, all I²C headers |
| `motor_dc` | DRV8874PWPR ×4, motor output terminals, fault LEDs |
| `motor_stepper` | TMC2209-LA-T ×2, stepper output terminals, UART config |
| `motor_servo` | PCA9685PW, 16-channel servo headers |

---

## Project status

- [x] Design spec complete
- [x] KiCad schematic — substantially complete (all major blocks drawn)
- [ ] PCB layout
- [ ] Bringup firmware (I²C scan, PWM test, CAN loopback, UART echo)
- [ ] TCA9548A reverse-ARP HAL
- [ ] BMI270 driver + sensor fusion
- [ ] TMC2209 UART config
- [ ] Full LCSC/Mouser BOM with pricing

---

## Bill of materials

Key ICs: STM32G431RBTx · ESP32-C3-MINI-1-N4 · DRV8874PWPR ×4 · TMC2209-LA-T ×2 · PCA9685PW · ADS1115IDGST · PCF8574DW · TCA9548APWR ×2 · SN65HVD230D · BMI270 · USB4085-GF-A · XL4016 ×2 · LM2596T-5 · AP2112K-3.3

---

## Repo structure

```
/
├── hardware/
│   └── Robodev/     KiCad project (schematic + PCB)
├── firmware/        ESP-IDF / STM32 firmware (not yet started)
├── docs/
│   ├── Datasheets/  Component datasheets
│   └── RoboDevDesignSpec.docx
└── README.md
```

---

## Contributing

This is an early-stage open hardware project — no PCB has been fabricated yet. If you want to follow along or contribute, the most useful things right now are:

- KiCad schematic review
- Firmware: STM32 HAL bringup, TCA9548A I²C mux driver, TMC2209 UART configuration
- ESP32-C3 SPI bridge firmware
- BMI270 SPI driver + Madgwick/Mahony filter

---

*STM32 + ESP32-C3 Integrated Robotics Carrier Board · May 2026*
