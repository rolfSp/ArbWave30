<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->


<!-- Assisted-by: DeepSeek - document framework -->

# ArbWave30 - Technical Specification

| Document ID | ArbWave30-SRS-001 | Version | V1.2 |
|---------|--------------|------|------|
| Project | ArbWave30 | Date | 2026-04-06 |
| Author | EERNINUO | Status | Draft |

## 1. Overview
This device is a digitally synthesized signal source based on a Gowin FPGA and a high-speed DAC. The project was started to address the author's lack of a high-speed signal source for learning and practical work, and aims to provide high-precision, wide-bandwidth programmable waveform output. It supports USB remote control and can be extended with an external reference clock input, making it suitable for laboratory testing, teaching demonstrations, and embedded-system debugging.

## 2. Referenced Documents and Standards

| Document/standard | Identifier | Description |
|---------|------|-----|
| SCPI standard | SCPI-1999 | Standard Commands for Programmable Instruments |
| USB device class | USB TMC 1.0 | USB-TMC protocol |
| IEEE 488.2 | IEEE Std 488.2-1992 | Instrument response format |

## 3. Electrical Parameters (Electrical Specifications)

### 3.1 Output Characteristics (Main Channel)

| Parameter | Symbol | Minimum | Typical | Maximum | Unit | Conditions/notes |
|------|------|--------|--------|--------|------|------------|
| **Waveform types** | – | Sine, square, triangle, noise, DC | – | – | – | Arbitrary waveforms supported by a future firmware upgrade |
| **Maximum sine-wave frequency** | f_sine_max | – | – | 30 | MHz | – |
| **Maximum square-wave frequency** | f_sqr_max | – | – | 5 | MHz | Rise/fall time < 50ns |
| **Maximum frequency of other waveforms** | f_other_max | – | – | 5 | MHz | Triangle, noise |
| **Frequency resolution[^1]** | Δf | – | 1 | – | μHz | |
| **Frequency accuracy** | – | – | ±1 | – | ppm | Over 24 hours, using a temperature-compensated crystal oscillator (TCXO) as the reference |
| **Phase noise** | L(f) | – | -100 | – | dBc/Hz | @10kHz offset, 30MHz carrier, target |
| **Total harmonic distortion** | THD | – | -55 | -50 | dBc | f_sine ≤ 10MHz, full-amplitude output |
| **Spurious-free dynamic range** | SFDR | – | 65 | – | dBc | f_sine ≤ 30MHz, < 60dBc above 1MHz |
| **RMS output noise** | V_noise | – | < 3 | – | mV RMS | 0V output, DC~30MHz, 50Ω load |

[^1]: Frequency resolution is the minimum frequency step in DDS mode. Actual frequency accuracy depends on clock stability.

### 3.2 Amplitude and Resolution (High-Impedance Output Mode)

| Frequency range | Amplitude range[^2] (Vpp) | Maximum offset[^3] | Resolution | Absolute accuracy |
|----------|----------------|------------|--------|----------|
| DC ~ 10 MHz | 0 ~20 | ±10 V | 2 mV (typical, 5 mV worst case) | ±(1% + 2mV) @1kHz |
| 10 MHz ~ 20 MHz | 0 ~ 14 | ±7 V | 2 mV (typical, 5 mV worst case) | ±(2% + 2mV) |
| 20 MHz ~ 30 MHz | 0 ~ 10 | ±5 V | 2 mV (typical, 5 mV worst case) | ±(2% + 2mV) |

[^2]: With a 50Ω load, the amplitude range is 1/2 of that in the high-impedance state.
[^3]: The offset voltage plus the amplitude must be less than the maximum amplitude at zero offset for the corresponding frequency.

### 3.3 Square-Wave Characteristics (High Impedance, 5MHz)

| Parameter | Target | Unit |
|------|--------|------|
| Rise/fall time (10% ~ 90%) | < 50 | ns |
| Overshoot (target) | < 5 | % |
| Jitter (RMS) (target) | < 500 | ps |

### 3.4 Sweep and Modulation (Future Implementation)

| Function | Parameter | Target |
|------|------|------|
| Linear/logarithmic frequency sweep | Start/stop frequency | 1μHz ~ 30MHz |
| AM modulation | Modulation frequency | 1Hz ~ 100kHz, modulation depth 0~100% |

### 3.5 Clock Characteristics

| Parameter | Typical value | Amplitude | Notes |
|------|-------|------|------|
| Internal reference frequency | 10 MHz | \ | TCXO, initial accuracy ±1ppm, aging ±1ppm/year |
| External reference input | 10 MHz | 3.3V | ±10ppm, 0~10dBm, SMA connector |
| Reference output | 10 MHz | 3.3V | LVCMOS33, SMA |

### 3.6 Trigger Input/Output

| Parameter | Amplitude | Notes |
|------|-------|------|
| Multifunction input | 3.3V/5V compatible | SMA |
| Multifunction output | 3.3V | SMA |

### 3.7 Power and Environment

| Parameter | Specification |
|------|------|
| DC input voltage | 24 V DC |
| Power consumption | < 20 W (estimated) |
| Operating temperature | 0 ~ 40 ℃ |
| Storage temperature | -20 ~ 70 ℃ |
| ESD protection | ±4 kV contact discharge (TVS at the output BNC) |

## 4. Functional Requirements

### 4.1 Basic Functions
- Generate sine, square, triangle, noise, and DC waveforms.
- Configurable frequency, amplitude, offset, and phase.
- Output switching (ON/OFF); restore the last settings after power loss (stored in EEPROM).

### 4.2 Modulation and Sweep (V2 Implementation)
- AM: internal/external modulation, configurable modulation frequency/depth.
- Linear/logarithmic frequency sweep.

### 4.3 Arbitrary Waveforms (V2 Implementation)
- User-defined waveform length: up to 16kpts (14bit resolution).
- Download waveform data using host software or SCPI commands.

### 4.4 External Interfaces
- USB 2.0 (USBTMC or CDC class) for remote control and waveform data transfer.
- Waveform output: SMA connector, switchable 50Ω/high impedance.
- External reference clock input (10 MHz, SMA connector).
- Trigger input/output (SMA).
- Power input (DC2.1mm, 24~36 V).
- Optional: LAN interface (future implementation).

<a id="remote-control"></a>

### 4.5 Remote Control
- Interface: USB 2.0 (USB-CDC class).
- Command set: compliant with SCPI-1999, including at least the following subsystems:
  - `*IDN?`, `*RST`, `*CLS`
  - `SOURce`: FREQuency, VOLTage, VOLTage:OFFSet, FUNCtion, PHASe
  - `OUTPut`: STATe, IMPedance, LOAD
  - `TRIGger`: SOURce, SLOPe
  - `SYSTem`: ERRor?
  - `SOURce`:TRACe:DATA?
- Response format: `<header><value><terminator>`; supports queries and settings.

### 4.6 User Interface (Long-Term Expansion)
- Optional: a simple GUI running on a USB-connected PC (Python+PyQt).
- Optional: later port to a Linux board (running a local GUI).

## 5. Hardware Architecture Requirements

### 5.1 Main IC Selection (Preliminary)

| Module | Part | Key parameters | Function |
|------|------|----------|----------|
| FPGA | Gowin GW2AR-LV18 | Logic resources > 20k LUTs, PLL support | DDS core, clock management, data processing |
| MCU | STM32F411 | 32-bit ARM Cortex-M4 processor, 96MHz core clock | System control, USB communication, user interface |
| Clock generator | CDCI6214 | 4 outputs, external reference support | DAC clock and system clock |
| DAC | AD9747 | 16bit, up to 250MSPS, dual channel | Waveform generation core, high-precision output |
| Reconstruction filter | 7th-order elliptic LPF | 60MHz cutoff, passband ripple <0.5dB | Remove DAC sampling-frequency components and improve waveform quality |
| Differential amplifier | OPA695 | Current-feedback operational amplifier, bandwidth >500MHz | Differential-to-single-ended conversion |
| Attenuation network | Precision resistors, signal relays | Resistor tolerance <0.1% | Automatically adjust gain according to frequency for segmented amplitude control |
| Output power stage | THS3095 | | Drive ±10V into a 1MΩ load |

### 5.2 PCB Requirements
- Layers: ≥4 (signal-ground-power-signal).
- Separate analog board (FPGA+DAC) and control board (STM32), connected over SPI.
- DAC clock traces as a differential pair (or single-ended with ground shielding), with tightly controlled impedance (50Ω).
- Separate LDO supply for analog power.

## 6. Development Plan

| Stage | Function | Description |
|------|------|------|
| V1.0 | Single channel, basic waveforms (sine, square, triangle, noise, DC) | Continuously adjustable frequency, amplitude, offset, and phase; SCPI support (a small core command set) |
| V2.0 | Dual channel | Add a second output with frequency/phase synchronization and adjustable phase difference |
| | AM modulation | Internal/external modulation source, modulation depth 0~100% |
| V3.0 | Embedded GUI | Linux + LVGL, direct control through an LCD touchscreen |
| | Arbitrary waveforms | User-defined waveform tables, storage depth ≥1Mpts |

## 7. Acceptance Test Methods

| Parameter | Test equipment | Method summary[^4] | Tolerance |
|------|----------|--------------|------|
| Maximum frequency | Siglent SDS824X | 30MHz sine wave, measure the period | ±0.1% |
| Amplitude accuracy | Siglent SDS824X | 1kHz sine wave, compare the setpoint with the measured value | See Section 3.2 |
| THD | SDS824X (FFT mode) | 10MHz sine wave, calculate harmonic power | ≤ -50dBc |
| Rise time | Siglent SDS824X | 5MHz square wave, measure 10%-90% | < 50ns |
| Frequency accuracy | Siglent SDS824X | 10MHz output, drift over 24 hours | ±1ppm |
| Output flatness | Siglent SDS824X | 10MHz sine wave, measure amplitude variation over 0.1MHz~10MHz | < ±0.5dB |
| Jitter (RMS) | SDS824X + long acquisition (>1000 edges) | 5MHz square wave, measure standard deviation of edge timing deviations | < 200 ps RMS |
| SCPI compatibility | PC + PyVISA | Send `*IDN?` and verify the response format | Must pass |
| Phase noise | Low-phase-noise spectrum analyzer required | - | Cannot currently be characterized due to test equipment limitations |

[^4]: Use a short SMA-to-BNC cable and a 50Ω impedance-matching adapter to connect the oscilloscope, with its full bandwidth enabled.

## 8. References

1. SDG1000X datasheet (Siglent Technologies)
2. AD9747 datasheet (ADI)
3. THS3091 datasheet (TI)
4. Gowin Semiconductor GW2A series datasheet
5. *Signal Integrity and Power Integrity* (Joe Hogue)
6. *Digital Signal Processing* (John G. Proakis)
7. *Op Amps for Everyone* (TI)

## 9. Revision History

| Version | Date | Changes | Author |
|------|------|----------|--------|
| V1.0 | 2026-03-01 | Initial version | EERNINUO |
| V1.1 | 2026-04-05 | Update technical specifications and adjust test methods | EERNINUO |
| v1.2 | 2026-04-06 | Change power specifications | EERNINUO |
| v1.3 | 2026-04-23 | Add multifunction I/O characteristics | EERNINUO |
| v1.4 | 2026-04-30 | Replace the DAC | EERNINUO |
