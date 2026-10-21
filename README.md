# ODIN — Open Droid INterface

**An open-source robot brick that works with LEGO® MINDSTORMS® EV3 and NXT motors and sensors.**

ODIN is a programmable controller ("brick") built to stand in for the EV3. Plug in the
motors, sensors and cables you already own, write your program in Python from a web
browser, and run it. Nothing to install. The design is open: anyone can study it, build
it, improve it, and contribute back.

> **Status: early development.** The prototype hardware is designed and being brought up.
> The production board is still at the schematic stage, and most of the firmware has not
> been written yet. This is a good time to get involved, because the big decisions are
> being made now.

---

## Who it's for

ODIN is for the people who used the EV3: **classrooms, teachers, robotics teams and
hobbyists.** It is not aimed at embedded developers. That choice shapes the whole design:

- **No toolchain to install.** You program it from Chrome over Web Bluetooth or Web Serial,
  so it works everywhere.
- **Works offline.** USB and Bluetooth need no internet connection and no cloud account.
- **Built to be handled.** Ports survive being plugged and unplugged while powered, cables
  in the wrong socket and shorted motor outputs.
- **Runs on AA batteries.** Six AA cells, the same as the EV3. Rechargeable NiMH is
  recommended and alkaline works. There is no lithium pack, so there is no charging
  circuit to fail and no battery-safety certification to carry.
- **Affordable in quantity.** The target bill of materials is **under $100**.
  We want the board to be affordable for the people using it. 

## Two boards

| | **Dev carrier** (now) | **Production brick** (planned) |
|---|---|---|
| Sensor input ports | 2 | 4 (EV3 ports 1–4) |
| Motor output ports | 2 | 4 (EV3 ports A–D) |
| Expansion header (CAN) | ✓ | ✓ |
| Processor | Waveshare ESP32-P4-WIFI6 dev board | ESP32-P4 on a custom PCB |
| Display | 2.4" 320×240 colour TFT | 2.4" 320×240 colour TFT |
| Purpose | Prove out every circuit and protocol | The brick you put in a robot |

**The dev carrier** is a board that plugs into an off-the-shelf ESP32-P4 dev board. With
two sensors and two motors it is already a working differential-drive robot. It uses the
*same* port circuits as the production brick, generated from the same source, so testing
the dev carrier tests the real design. It has 2 + 2 ports because the dev board only
exposes enough pins for that many.

**The production brick** (planned) has the full EV3 port count, 4 in and 4 out, in a
LEGO-grid-aligned case about the size of an EV3 (15 × 9 studs × 5 bricks). It has Technic
pin holes on both sides so it mounts where an EV3 did.

Both boards have an **external expansion header**, so you can add more I/O beyond the
built-in ports.

## What it works with

| Family | Examples | Built-in ports |
|---|---|---|
| EV3 sensors | Color, Ultrasonic, Gyro, Touch, Infrared | ✓ |
| EV3 motors | Large, Medium | ✓ |
| NXT sensors | Touch, Light, Color, Sound, Ultrasonic | ✓ |
| NXT motors | NXT servo motor | ✓ |
| Third-party I²C sensors | HiTechnic, Mindsensors, etc. | ✓ |
| Powered Up / SPIKE Prime | SPIKE motors and sensors | Planned, via an expansion board |

Devices are **detected automatically** when you plug them in, using the same scheme as
the EV3. The port circuits follow LEGO's own published EV3 Hardware Developer Kit
schematics.

You still need EV3-style RJ12 cables, the ones with the offset latch.

## How it works (Goals of the project)

```
                        [ ESP32-P4 ]
                  dual-core RISC-V, 360 MHz
                             |
     +---------------+-------+-------+-------------------+
     |               |               |                   |
 Sensor ports    Motor ports    ESP32-C6 radio     Expansion header
 UART/I2C/ADC    DRV8876 drivers  Wi-Fi 6 + BLE    CAN bus (JST-GH)
```

- **Processor:** Espressif **ESP32-P4**. **ESP32-C6** companion chip for Wi-Fi 6 and
  Bluetooth LE.
- **Motor drivers:** TI **DRV8876** H-bridges, one per port. They have per-motor current
  sensing and limit current to EV3-equivalent levels, so ODIN never drives a LEGO motor
  harder than an EV3 would.
- **Analog:** an external SPI ADC reads every port's analog and detection signals, as the
  EV3 does.
- **Expansion:** a **CAN bus** on a 4-pin JST-GH connector (5 V, GND, CAN_H, CAN_L), using
  the [Cyphal](https://opencyphal.org/) protocol. The brick powers each daughterboard's
  logic. High-power boards bring their own battery. Daughterboards chain, so you can add
  several.
- **Firmware:** ESP-IDF (C), running a **Pybricks-style MicroPython** VM.
- **Programming:** **Python only**, from a browser-based editor over Bluetooth or USB.
  Several programs can be stored on the brick and picked from its menu, as on the EV3.

## Expansion boards

The expansion header is how ODIN grows past 4 + 4 ports. A daughterboard is just another
node on the CAN bus. Its ports appear to your program exactly like built-in ones, and its
messages are defined in a published, versioned protocol (`odin.*` Cyphal DSDL types).
That means **anyone can design an expansion board** without needing permission from us or
reverse-engineering anything.

The first planned daughterboard adds **LEGO Powered Up / SPIKE Prime** ports. Those
devices speak almost the same serial protocol as EV3 sensors, so most of the software
carries over.

## Repository layout

| Path | What's there |
|---|---|
| [`hardware/`](hardware/) | KiCad 9 schematics for the dev carrier. Open `odin_dev.kicad_pro`. Also has the breadboard layout PDF |

More will be published as it matures: production-brick schematics, the firmware, the
Cyphal DSDL definitions and build guides.

## Roadmap

- [x] Architecture and port design, checked against LEGO's EV3 Hardware Developer Kit
- [x] Dev carrier schematics (2 sensor + 2 motor ports)
- [x] Production brick schematics drafted (4 + 4 ports)
- [x] `odin.*` Cyphal protocol types defined
- [ ] Dev carrier bring-up: motors, encoders, sensor detection, UART sensors
- [ ] Firmware port drivers on ESP-IDF
- [ ] MicroPython VM and browser-based editor
- [ ] Production PCB layout and enclosure
- [ ] Powered Up / SPIKE Prime expansion board

## Contributing

ODIN is an open project and contributions of every kind are welcome. You don't need to be
an electrical engineer:

- **Hardware:** review schematics, check the port circuits against the EV3, build a dev
  carrier and report what happens.
- **Firmware:** sensor and motor drivers, the CAN expansion stack, the MicroPython port.
- **Software:** the browser-based editor and tooling.
- **Education:** lesson ideas, documentation, error messages a twelve-year-old can
  understand.
- **Testing:** try your LEGO sensors and motors, including obscure and third-party ones.

How to take part:

- **[Issues](../../issues)** are for bugs, hardware errata and concrete proposals.
- **[Discussions](../../discussions)** are for questions, ideas and design debate.
- **Pull requests** are welcome. For anything large, please open an issue or discussion
  first so we can agree on the approach before you put in the work.

## License

- **Hardware** (schematics, PCB and enclosure designs, and the scripts that generate them)
  is licensed under the
  [CERN Open Hardware Licence v2 — Strongly Reciprocal](LICENSE-HARDWARE)
  (CERN-OHL-S-2.0).
- **Software and firmware** are licensed under the [MIT License](LICENSE-SOFTWARE).

See [`LICENSE`](LICENSE) for exactly which files fall under each licence.

## Trademarks

LEGO®, MINDSTORMS®, EV3, NXT, Powered Up and SPIKE™ are trademarks of the LEGO Group,
which does not sponsor, authorize or endorse this project. ODIN is an independent,
community-built project designed for compatibility with LEGO peripherals.
