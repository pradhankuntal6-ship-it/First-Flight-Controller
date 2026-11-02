# drone bystm32 — Custom STM32F405 Flight Controller

A compact, custom-designed flight controller board for multirotor drones, built around the **STM32F405RGT6** (ARM Cortex-M4F, 168 MHz, 1 MB Flash, 192 KB RAM). Designed in **EasyEDA**.

> **Status:** Work in progress / prototype. Hardware revision **V1.0**. Please read the [Known Issues](#known-issues--review-notes) section before ordering or flashing a board.

<!-- Add a photo or render of the board here -->
<!-- ![Board](docs/board.png) -->

---

## Features

- **MCU:** STM32F405RGT6 (LQFP-64)
- **IMU:** MPU-6050 (3-axis gyro + 3-axis accelerometer) over I²C
- **Barometer:** BMP280 over I²C
- **USB:** USB Type-C (16-pin) connector for power, DFU flashing and USB data
- **Motor outputs:** 6 channels (M1–M6) on a 2×9 header
- **Servo / auxiliary outputs:** 4 channels (S1–S4) on a 2×6 right-angle header (H6) with 5 V
- **UARTs:** 4 broken-out ports (UART1, UART2, UART4, UART6)
- **I²C expansion header** (SDA/SCL, 3.3 V, GND) for GPS compass, OLED, etc.
- **SPI expansion header** (SCK, NSS, MISO, MOSI) for external SPI devices
- **Buzzer** output header
- **Crystals:** 8 MHz HSE + 32.768 kHz LSE
- **Power:** 5 V in (USB) → MS1117-3.3V LDO → 3.3 V rail, with ferrite-bead filtered analog supply (BLM18PG121SN1D on VDDA)
- **Buttons:** BOOT0 (SW1) and NRST (SW2) tactile switches

---

## Repository Structure

```
.
├── hardware/
│   ├── SCH_Schematic1_2026-11-03.pdf        # Schematic (PDF)
│   ├── PCB_PCB1_2026-11-02.pdf              # PCB layout / BOM (PDF)
│   └── easyeda/                             # EasyEDA project source (add if available)
├── firmware/                                # Firmware source (add)
├── docs/                                    # Photos, notes
└── README.md
```

---

## Hardware Overview

### Block Diagram

```
 USB-C (5V, D+/D-) ──► MS1117-3.3V ──► +3V ──► STM32F405, MPU-6050, BMP280
        │                                         │
        └──► +5V (servo header, buzzer, etc.)     ├── I2C2 (PB10/PB11) ── MPU-6050, BMP280, H1
                                                  ├── SPI2 (PB12–PB13, PC2–PC3) ── H9
                                                  ├── UART1/2/4/6 ── H4/H5/H3/H2
                                                  └── PWM ── Motors M1–M6 (H7), Servos S1–S4 (H6)
```

### Main Components

| Ref | Part | Function |
|-----|------|----------|
| U1 | STM32F405RGT6 | Main microcontroller |
| U2 | MS1117-3.3V (SOT-223) | 5 V → 3.3 V LDO |
| U3 | MPU-6050 | 6-axis IMU |
| U4 | BMP280 | Barometric pressure sensor |
| USB1 | TYPE-C 16PIN 2MD(073) | USB-C connector |
| X1 | 8 MHz crystal (HC-49US) | HSE clock |
| X2 | 32.768 kHz crystal | LSE / RTC clock |
| L1 | BLM18PG121SN1D | Ferrite bead for VDDA |
| SW1 | K2-1817UQC4SW-01 | BOOT0 select |
| SW2 | K2-1817UQC4SW-01 | NRST reset |

### Pin Mapping

**Motor / servo outputs**

| Signal | MCU Pin | Header |
|--------|---------|--------|
| M1 | PB8 | H7 |
| M2 | PB5 | H7 |
| M3 | PB4 | H7 |
| M4 | PB3 | H7 |
| M5 | PA10 | H7 |
| M6 | PA9 | H7 |
| S1 | PB15 | H6 |
| S2 | PB14 | H6 |
| S3 | PA7 | H6 |
| S4 | PA6 | H6 |

**Communication**

| Interface | Pins | Header |
|-----------|------|--------|
| UART1 | PB6 (TX), PB7 (RX) | H4 |
| UART2 | PA2 (TX), PA3 (RX) | H5 |
| UART4 | PA0 (TX), PA1 (RX) | H3 |
| UART6 | PC6 (TX), PC7 (RX) | H2 |
| I²C2 | PB10 (SCL), PB11 (SDA) | H1, U3, U4 |
| SPI2 | PB13 (SCK), PB12 (NSS), PC2 (MISO), PC3 (MOSI) | H9 |
| USB FS | PA11 (D−), PA12 (D+) | USB1 |
| Buzzer | PB9 | H8 |

**Sensor I²C addresses (as wired)**

| Device | Address | Notes |
|--------|---------|-------|
| MPU-6050 | `0x68` | AD0 pulled low via R7 (10 kΩ) |
| BMP280 | `0x76` | SDO pulled low, CSB tied high (I²C mode) |

> Please verify these against the schematic when writing drivers.

### Power

- Input: 5 V from USB-C VBUS
- 3.3 V: MS1117-3.3V with 10 µF capacitors on the input and output (C7, C8) and 100 nF on the 5 V side (C9)
- VDDA is filtered through ferrite bead L1
- 100 nF decoupling on the VDD pins (C1–C5), 1 µF on VBAT/VDDA net (C6)
- VCAP_1 / VCAP_2 have C10 / C11 (shown as 2.2 on the schematic, see Known Issues)
- USB-C CC1/CC2 use 5.1 kΩ pull-downs (R3, R4) so the board is detected as a USB device

### Boot & Programming

- **DFU over USB:** hold **SW1 (BOOT0)** while plugging in USB-C (or while pressing SW2/NRST), then flash with [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) or `dfu-util`.
- **SWD:** PA13 / PA14 are **not** broken out to a header on this revision — see Known Issues.

---

## Getting Started

### Requirements

- STM32CubeIDE / PlatformIO / ARM GCC toolchain
- STM32CubeProgrammer or `dfu-util`
- USB-C cable

### Flashing

1. Hold the BOOT0 button and connect the board to your PC over USB-C.
2. Open STM32CubeProgrammer, choose **USB**, and click *Connect*.
3. Load your `.bin` / `.hex` and flash.
4. Release BOOT0 and press NRST to run the firmware.

### Firmware

Firmware is not yet included. Candidate options:
- Custom bare-metal / HAL firmware
- Porting an existing open-source flight stack (e.g. Betaflight / ArduPilot) with a custom target

---

## Bill of Materials

The BOM exported from EasyEDA (with LCSC part numbers) is in `hardware/PCB_PCB1_2026-11-02.pdf`. **That BOM predates the latest schematic (2026-11-03) and still lists the old capacitor and resistor values, so re-export it before ordering.** The table below follows the latest schematic:

| Qty | Designators | Value / Part | Footprint |
|----:|-------------|--------------|-----------|
| 5 | C1–C5 | 100 nF | 0201 |
| 1 | C6 | 1 µF | 0201 |
| 2 | C10, C11 | 2.2 µF (VCAP, label reads "2.2pF", see Known Issues) | 0201 |
| 2 | C7, C8 | 10 µF (LDO input/output) | 0603 |
| 3 | C9, C12, C13 | 100 nF | 0603 |
| 2 | C20, C21 | 10 pF (8 MHz crystal load) | 0603 |
| 2 | C22, C23 | 12 pF (32.768 kHz crystal load) | 0603 |
| 6 | C14–C19 | 0.1 µF | 0603 |
| 1 | U1 | STM32F405RGT6 | LQFP-64 |
| 1 | U2 | MS1117-3.3V | SOT-223 |
| 1 | U3 | MPU-6050 | QFN-24 |
| 1 | U4 | BMP280 | LGA-8 |
| 1 | USB1 | TYPE-C 16PIN 2MD(073) | USB-C SMD |
| 1 | X1 | 8 MHz crystal (X49SD8MSB2SI) | HC-49US |
| 1 | X2 | 32.768 kHz crystal (SF32WK32768D31T002) | SMD 3.2×1.5 |
| 1 | L1 | BLM18PG121SN1D ferrite bead | 0603 |
| 2 | R1, R2 | 470 Ω (per BOM) | 0603 |
| 2 | R3, R4 | 5.1 kΩ (USB-C CC) | 0603 |
| 1 | R7 | 10 kΩ | 0603 |
| 2 | R5, R6 | 22 Ω | 0603 |
| 6 | R8–R13 | 1 kΩ | 0603 |
| 2 | SW1, SW2 | HX 3x4x2.5-PA-4N push button | SMD 4P |
| 5 | H1–H5 | PZ254V-11-04P (1×4, 2.54 mm) | TH |
| 1 | H6 | TSW-104-08-F-T-RA (2×6, right-angle) | TH |
| 1 | H7 | 2011-3X06G00RB (2×9) | TH |
| 1 | H8 | PZ254V-11-02P (1×2) | TH |
| 1 | H9 | 2.54-1x6P (1×6) | TH |

---

## Known Issues / Review Notes

Several problems from the earlier revision have been fixed in the 2026-11-03 schematic: the USB-C CC resistors are now 5.1 kΩ, the LDO capacitors are 10 µF, and the crystal load capacitors are now in the pF range. Remaining items to check **before fabricating or powering a board**:

1. **VCAP capacitor labels (C10, C11):** the schematic text reads **2.2pF**, but the STM32F405 needs about **2.2 µF** (low-ESR) on each VCAP pin. This looks like a unit typo; correct it to 2.2 µF and make sure the footprint/part can actually be a 2.2 µF capacitor.
2. **Crystal load capacitors:** C20/C21 (10 pF) and C22/C23 (12 pF) are now in a sensible range, but the right values depend on each crystal's datasheet load capacitance and the PCB stray capacitance. Please verify them.
3. **BOM out of date:** the BOM PDF (2026-11-02) is older than the schematic (2026-11-03) and still shows 100 nF for C7–C10, C20–C23 and 10 kΩ for R3/R4. Re-export it, and confirm the PCB layout was updated with the new footprints.
4. **No SWD header:** PA13/PA14 are not broken out; flashing is only possible over USB DFU.
5. **No battery voltage / current sensing** or dedicated power input on this revision.

---

## Roadmap

- [x] Fix USB-C CC resistors, LDO capacitors and crystal load capacitors
- [ ] Correct VCAP capacitor labels to 2.2 µF and re-export the BOM
- [ ] Add SWD header
- [ ] Add battery voltage sensing and power input
- [ ] Write / port firmware
- [ ] Add 3D render and assembly photos

---

## Contributing

Issues and pull requests are welcome. If you spot a schematic or layout error, please open an issue with the reference designator and a description.

## License

Choose a license and add it here (for example **MIT** for firmware and **CERN-OHL-P / CC BY-SA 4.0** for hardware).

## Disclaimer

This is an experimental hobby design provided as-is without warranty. Use caution when working with LiPo batteries and spinning propellers — remove props during bench testing.
