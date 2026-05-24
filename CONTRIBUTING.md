# Contributing to RoboDev

Thanks for your interest in contributing! RoboDev is an early-stage open hardware project — the schematic is substantially complete but the PCB layout has not yet started. All kinds of contributions are welcome.

---

## Where help is needed right now

- **KiCad schematic review** — feedback on component choices, net naming, hierarchy structure, and layout suggestions
- **STM32 firmware** — HAL bringup routines (I²C scan, PWM test, CAN loopback, UART echo), TCA9548A I²C mux driver, TMC2209 UART multi-node config
- **ESP32-C3 firmware** — SPI bridge to STM32, BMI270 SPI driver, Madgwick/Mahony sensor fusion filter
- **BOM review** — alternative component suggestions, better pricing sources, LCSC part number verification

---

## How to contribute

1. Fork the repo and create a branch: `git checkout -b my-feature`
2. Make your changes
3. Open a pull request with a clear description of what you changed and why

For larger changes (new subsystems, significant redesigns), open an issue first to discuss before investing time in it.

---

## Repo structure

```
/
├── hardware/
│   └── Robodev/     KiCad project (schematic + PCB)
├── firmware/        STM32 HAL + ESP-IDF firmware (not yet started)
├── docs/
│   ├── Datasheets/  Component datasheets
│   ├── RoboDevDesignSpec.tex  Design specification (source)
│   └── RoboDevDesignSpec.pdf  Design specification (rendered)
└── README.md
```

---

## Hardware (KiCad)

- Use **KiCad 8** or later
- The schematic uses a hierarchical multi-sheet structure — see the README for the sheet breakdown
- Follow existing net naming conventions
- Reference designators follow the format `U1`, `R1`, `C1` etc. — don't renumber existing parts in a PR

## Firmware (STM32)

- Target: **STM32G431RBTx** · STM32CubeIDE / STM32CubeMX + HAL
- Keep peripheral drivers modular — one file per peripheral (e.g. `can.c`, `i2c_mux.c`)
- Application code should never touch mux or expander registers directly — go through the HAL abstraction

## Firmware (ESP32-C3)

- Target: **ESP32-C3-MINI-1-N4** · **ESP-IDF v5.x**
- Responsible for: SPI bridge to STM32, BMI270 IMU, wireless comms
- Format code with `clang-format` before submitting

---

## License

By contributing, you agree that:

- Firmware/software contributions are licensed under **MIT**
- Hardware contributions (schematics, PCB, BOM) are licensed under **CERN-OHL-S v2**

---

## Questions?

Open an issue and tag it `question`.
