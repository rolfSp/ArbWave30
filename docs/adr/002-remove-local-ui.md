<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->

<!-- Assisted-by: DeepSeek - Document framework -->

# ADR-002: Remote Control Only in V1.0/V2.0, with No Local User Interface

## Status
Accepted (2026-04-27)

## Context
ArbWave30 is positioned as a 30MHz arbitrary waveform generator whose core value is:
- High signal quality (low THD, low noise, and wide bandwidth)
- Full SCPI command-set support for automated testing

The initial design planned local controls for V1.0 (buttons, a rotary encoder, and a small TFT display) to allow basic settings without a PC connection. However, further design work revealed the following issues:

1. **Limited MCU pins**: After allocating pins on the STM32F411CEU6 (48 pins) to USB, SPI (to the FPGA), I2C (to the EEPROM), debugging, and power management, the remaining GPIOs cannot readily support a matrix keypad + encoder + SPI display. Forcing these additions would severely constrain the PCB layout.
2. **Limited value of a local display**: A small 1.8-inch display cannot clearly show waveforms, frequency, amplitude, and other parameters. A 2.8-inch touchscreen is feasible, but would add substantial UI code debugging work, diverting effort from core modules such as the analog front end, FPGA DDS, and SCPI interpreter.
3. **Remote control is the primary use case**: A PC connection is available in most typical use cases. Tools such as Python/PyVISA can send SCPI commands for automated testing or manual debugging. Local operation is not essential in most cases.
4. **Low-cost iteration**: V1.0 should complete hardware validation and basic functionality as soon as possible. Local interaction can be a V3.0 enhancement, potentially using a Linux system-on-module and a large display for a better experience without delaying the main development schedule.

## Decision

**Remove all local user-interface hardware (buttons, encoders, TFT/LCD displays) entirely from V1.0 and V2.0, retaining only a few status LEDs for indication.**

The control board's core responsibilities are limited to:
- USB communication (USBTMC or CDC class), implementing an SCPI server
- Configuring FPGA DDS parameters (frequency, phase, waveform, amplitude coefficient) over SPI
- Configuring the CDCI6214 clock generator over I2C
- Providing power and control signals to the analog board through the inter-board connectors

All parameter settings, waveform selection, and output control are performed through SCPI commands. Users can control the instrument using any VISA-compatible software (such as Python PyVISA, LabVIEW, or Keysight Command Expert) or the project's simple PyQt PC application.

## Consequences

### Positive

1. **Reduced hardware complexity**: Smaller control-board PCB area, simpler routing, more flexible pin allocation, and lower EMC risk.
2. **Shorter development cycle**: No button-scanning, menu-logic, LCD-driver, or touch-calibration code is required. Firmware can focus on SCPI parsing and SPI/I2C communication.
3. **Improved reliability**: Fewer mechanical components (buttons, encoders) and ribbon-cable connections reduce hardware failure points.

### Negative

1. **No standalone operation without a PC**: Users must have a computer connected over USB to operate the instrument. This limitation is acceptable because the target users (laboratory engineers and developers) have PCs and USB cables are inexpensive.
2. **An additional computer is needed for debugging**: Every parameter change during development requires an SCPI command. A simple PyQt PC application or a serial-terminal utility can mitigate this, and it is already part of automated testing.
3. **Incomplete product appearance**: The instrument (v1.0) has only one SMA output, four special-function SMA ports, one USB port, and one power input, with no display or buttons. This may confuse first-time users, but clear silkscreen labels and documentation can address it.

## Alternatives

| Option | Description | Reason for Rejection |
|------|------|----------|
| Retain encoder + buttons without a display | Blind operation with buttons and an encoder, relying on buzzer or LED feedback | Cannot display current settings, has limited practical value, and still consumes GPIOs |
| Use a larger-package MCU (such as STM32F407VET6) | A 100-pin MCU provides enough pins for a keypad and display | Increases PCB area and cost; local interaction is not a core V1.0 objective, so changing platforms is not worthwhile |
| Use a Linux system-on-module (such as RK3506) from V1.0 | Run Linux directly and drive a large touchscreen for a full local GUI | Requires a substantially different technology stack and an unpredictable development schedule, conflicting with the current priority of meeting hardware specifications first |

## Follow-up Plan

- V1.0 / V2.0: Retain the remote-only design and complete all performance targets and the SCPI command set.
- V3.0 (planned): Evaluate adding a local HMI. A separate Linux system-on-module (such as RK3506) would replace the MCU on the control board. Local display and remote control can coexist.

## References

- Project Technical Specification `docs/technical-specification.md`, Section 4.5 (Remote Control)