<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->

<!-- Assisted-by: DeepSeek - Document framework -->

# ArbWave30 – Inter-Board Interface Design Document

| Document ID | ArbWave30-BIID-001 | Version | V1.2  |
| --------| ----------------- | ---- | ----- |
| Project Name | ArbWave30 | Date | 2026-05-26 |
| Author   | EERNINUO | Status | Draft |

## 1. Overview
This document defines all physical and electrical interfaces between the **control board** and **analog board** in the ArbWave30 project. The control board handles power input, primary power conversion, USB communication, and SCPI parsing; the analog board contains the FPGA, DAC, and analog front end.

The two boards are interconnected through **two types of connectors**:
- **Main power connector J_PWR**: Carries the ±18V power supplies (subsequently converted to ±15V and other rails by LDOs).
- **Signal/auxiliary power connector J_SIG**: Carries digital control signals (SPI, I2C, reset, etc.) and the low-voltage auxiliary supply (+3.3V_D).

Design, placement, routing, and firmware development for both boards must use this document as the sole reference.

## 2. Connector Selection and Mechanical Requirements

### 2.1 Main Power Connector (J_PWR)

| Item | Specification |
|------|------|
| **Type** | VH3.96 pitch, wire-to-board, through-hole |
| **Pin count** | 8 pins (with spare capacity) |
| **Rated current** | 3A/pin (actual requirement < 2A/pin, providing ample margin) |
| **Contact resistance** | ≤10mΩ |
| **Withstand voltage** | ≥250V |
| **Operating temperature** | -40℃ \~ +105℃ |
| **Selection rationale** | VH connectors are widely used and inexpensive, with an integral latch to prevent disconnection. |

### 2.2 Signal Connector (J_SIG)

| Item | Specification |
|------|------|
| **Type** | DC3 shrouded header, 2.54mm pitch |
| **Pin count** | 2×8P (16P)  |
| **Rated current** | 1A per pin (actual < 200mA/pin, with ample derating) |
| **Contact resistance** | ≤20mΩ |
| **Operating temperature** | -40℃ \~ +105℃ |
| **Selection rationale** | Low vibration in a laboratory makes disconnection unlikely. Gold-plated contacts ensure signal integrity; low-voltage power can also be carried. |

### 2.3 Mechanical Assembly Requirements

- **J_PWR location**: Near the board edge for convenient cable-harness routing.
- **J_SIG location**: At least 10mm from J_PWR to avoid harness interference.
- **Ribbon-cable length**: A J_SIG ribbon-cable length of ≤10cm is recommended to maintain SPI signal integrity.
- **Polarization**: Both the shrouded header and XH connectors have keyed notches to ensure correct insertion orientation.

## 3. Pin Definitions

### 3.1 Main Power Connector (J_PWR)

**Pin arrangement**: Notch facing up, pin 1 on the left

| Pin Number | Signal Name | Direction | Rated Current | Description |
|--------|--------|------|----------|------|
| 1 | +17V_M | Control board → Analog board | 1A | Positive supply rail for generating +15V_A |
| 2 | GND_PWR | - | - | Power ground return |
| 3 | -17V_M | Control board → Analog board | 1A | In parallel with pin 3 to share current |
| 4 | +7V_M | Control board → Analog board | 0.6A | Negative supply rail for generating -15V_A |
| 5 | GND_PWR | - | - | Power ground return |
| 6 | -7V_M | Control board → Analog board | 0.6A | In parallel with pin 4 to share current |
| 7 | GND_PWR | - | - | - | Power ground return |
| 8 | 3.3V_D | Control board → Analog board | 1A | Digital supply for FPGA IO, the DAC digital section, and analog-front-end relays |

<a id="signal-connector-j-sig"></a>

### 3.2 Signal Connector (J_SIG)

**Pin arrangement** (2×8P, shrouded-header notch facing left, pin 1 at the upper left, numbered counterclockwise)

| Pin | Signal Name | Direction | Level | Description |
|------|-------|------|------|-----|
| 1 | CS | Control board → Analog board | 3.3V | SPI chip select (active low) |
| 2 | CLK | Control board → Analog board | 3.3V | SPI clock |
| 3 | MISO | Analog board → Control board | 3.3V | SPI master in, slave out |
| 4 | MOSI | Control board → Analog board | 3.3V | SPI master out, slave in |
| 5 | NC | - | - | Reserved |
| 6 | FPGA_NRST | Control board → Analog board | 3.3V | FPGA reset (active low)|
| 7 | FPGA_INT | Analog board → Control board | 3.3V | FPGA status/interrupt output. High = interrupt request or FPGA not ready; low = normal operation. Weak pull-up (10kΩ) at the MCU end. |
| 8 | GPIO2 | Bidirectional | 3.3V | Spare GPIO |
| 9 | GND | - | - | Ground |
| 10 | GND | - | - | Ground |
| 11 | GND | - | - | Ground |
| 12 | GND | - | - | Ground |
| 13 | GND | - | - | Ground |
| 14 | GND | - | - | Ground |
| 15 | GND | - | - | Ground |
| 16 | GND | - | - | Ground |

## 4. Electrical Specifications

### 4.1 Digital Signal Levels

| Parameter | Minimum | Typical | Maximum | Unit |
|------|--------|--------|--------|------|
| VOH | 2.4 | 3.3 | - | V |
| VOL | - | 0 | 0.4 | V |
| VIH | 2.0 | - | 3.6 | V |
| VIL | -0.3 | - | 0.8 | V |
| Rise/fall time | - | - | 10 | ns |

### 4.2 Supply-Rail Ripple Requirements

| Supply Rail | Maximum Ripple (Peak-to-Peak) | Test Conditions |
|--------|-------------------|----------|
| +3.3V_D | 50mV | 10Hz \~ 20MHz |
| +18V_M | 100mV | 10Hz \~ 20MHz |
| -18V_M | 100mV | 10Hz \~ 20MHz |

### 4.3 Timing Requirements

- **SPI**: Clock frequency ≤10MHz, data setup time ≥10ns, hold time ≥10ns.
- **FPGA_RST**: Low pulse width ≥100ns.
- **Power-up sequence**: +3.3V_D is recommended before +1.0V_D (the FPGA core requires the IO supply to power up first).

## 5. Power Decoupling and Filtering Responsibilities

| Location | Decoupling Requirements |
|------|----------|
| **Control board (output side)** | Place 22μF + 0.1μF capacitors near each J_PWR power pin. Also place 22μF + 0.1μF on J_SIG's +3.3V_D. |
| **Analog board (input side)** | Add 10μF + 0.1μF at each power pin at the connector entry. For load-side design details, see [Analog-Board Power Design](./analog-board.md#analog-board-power-design) |

## 6. Grounding and Signal-Integrity Strategy

- **Ground planes**: Both boards use **continuous ground planes**, without splitting digital and analog ground.
- **Ground-pin allocation**: At least one GND pin for every 2\~3 signal pins in J_SIG (already satisfied).
- **Power return**: ±18V return current flows back to the control board through J_PWR GND pins; these must not share pins with signal ground.
- **Clock signals**: The 150MHz DAC clock is generated on the analog board and does not cross the connectors. SPI_CLK on J_SIG is relatively low-frequency (≤10MHz) and requires no special impedance control, but traces should be as short as possible.

## 7. Test and Verification Methods

| Test | Method | Pass Criterion |
|--------|------|----------|
| Continuity | Multimeter continuity mode | Contact resistance < 1Ω for each pin |
| Supply ripple | Oscilloscope with ground spring at the analog-board input | Meets Section 4.2 requirements |
| Signal quality | Oscilloscope measurement of SPI_CLK rising edge (analog-board side) | Overshoot < 0.5V, ringing < 20% |
| Communication test | Control board sends SPI commands; FPGA readback | No bit errors |
| Temperature-rise test | Run at full load for 30 minutes and measure connector housing temperature | < 50℃ (25℃ ambient) |

## 8. Revision History

| Version | Date | Changes | Author |
|------|------|----------|--------|
| V1.0 | 2026-04-25 | Initial version | EERNINUO |
| v1.1 | 2026-05-05 | Revised power pin assignment, moved 3.3V_D to the power connector, and changed power connector selection | EERNINUO |
| v1.2 | 2026-05-26 | Revised power pin assignment and moved ±7V generation to the control board | EERNINUO |
| v1.3 | 2026-09-25 | Corrected signal connector selection | EERNINUO |