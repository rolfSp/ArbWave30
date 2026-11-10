<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->

<!-- Assisted-by: DeepSeek - document framework -->

# ArbWave30 Standard Commands for Programmable Instruments (SCPI) Reference

| Document ID | ArbWave30-UM-002 | Version | V1.0 |
| --- | --- | --- | --- |
| Project | ArbWave30 | Date | 2026-08-17 |
| Author | EERNINUO | Status | Trial |

## 1. Overview
This document describes the remote-control command set for the ArbWave30 arbitrary waveform generator. The instrument follows **SCPI (Standard Commands for Programmable Instruments) 1999.0** and **IEEE 488.2-1992**.

**Communication interface**:
- **Physical layer**: USB 2.0 (CDC-class virtual serial port).
- **Baud rate**: 115200 bps (this setting has no practical effect on a virtual serial port).
- **Data bits**: 8, **stop bits**: 1, **parity**: none.
- **Terminator**: `\n` (line feed, hexadecimal 0x0A).

---

## 2. Syntax Conventions

The following notation is used in this manual:

| Symbol | Meaning | Example |
| :--- | :--- | :--- |
| `[ ]` | Optional keyword or parameter; may be omitted. | `OUTPut[:STATe]` means `OUTPut:STATe` may be shortened to `OUTPut` |
| `{ }` | Required parameter group; choose one. | `VOLTage {<amplitude>\|MIN\|MAX}` |
| `\|` | Logical OR separator. | `ON\|OFF` |
| `#` | Channel placeholder. | `SOURce#` represents `SOURce1` or `SOURce2` |
| `< >` | Actual value to be supplied by the user. | `<frequency>` |
| `() )` | Short command form (mnemonic); the SCPI engine accepts both long and short forms. | `FREQuency` (short form: `FREQ`) |

**Queries**: append `?` to a command to query the current setting, for example `FREQuency?`.

---

## 3. IEEE 488.2 Common Commands

### 3.1 `*IDN?` - Query Device Identification
**Function**: return the manufacturer, model, firmware version, and serial number.  
**Format**: `*IDN?`  
**Response**: `<manufacturer>,<model>,<firmware-version>,<serial-number>`  
**Example**:
> `*IDN?` -> `EERNINUO,ArbWave30,V1.0.0,SN123456`  
> \# Note: the actual response may vary with the device and firmware version.

### 3.2 `*RST` - Reset Device
**Function**: reset device parameters to factory defaults.  
**Format**: `*RST`  
**Defaults**:
- Frequency: 0 Hz.
- Waveform: sine.
- Amplitude: 0 Vpp.
- Offset: 0 V.
- Output state: OFF.

### 3.3 `*CLS` - Clear Error Queue
**Function**: clear the system error/event queue.  
**Format**: `*CLS`

## 4. `SOURce` Subsystem (Waveform Generation Parameters)

This subsystem configures the signal generator's core DDS parameters.

### 4.1 Frequency `[SOURce#:]FREQuency`

**Description**: set or query the output frequency of the specified channel.

**Command format**:  
`[SOURce#:]FREQuency {<frequency>|MIN|MAX}`  
`[SOURce#:]FREQuency? [MIN|MAX]`

**Parameters**:
- `<frequency>`: floating-point value in Hz.
- `MIN`: minimum, 0 Hz (0 Hz).
- `MAX`: maximum, 30,000,000 Hz (30 MHz).
- Default: 0 Hz.

**Examples**:
- `SOURce1:FREQuency 1000000` -> set channel 1 frequency to 1 MHz.
- `SOUR:FREQ MAX` -> set the currently selected channel to 30 MHz (short form).
- `FREQuency?` -> return `1000000.000000`.

### 4.2 Waveform Type and Specific Parameters `[SOURce#:]FUNCtion`

These commands select the output waveform and configure its specific parameters, such as square-wave duty cycle.

#### 4.2.1 `[SOURce#:]FUNCtion` - Select Waveform Shape

**Description**: set or query the waveform shape of the current channel.

**Command format**:  
`[SOURce#:]FUNCtion {SINusoid|SQUare|TRIangle|NOIse|DC}`  
`[SOURce#:]FUNCtion?`

**Parameters**:

| Parameter | Short form | Description |
| --- | --- | --- |
| `SINusoid` | `SIN` | Sine (default) |
| `SQUare` | `SQU` | Square (adjustable duty cycle; see 4.2.2) |
| `TRIangle` | `TRI` | Triangle |
| `NOIse` | `NOI` | Noise |
| `DC` | `DC` | DC level |

**Examples**:
- `SOUR:FUNC SIN` -> select a sine wave.
- `SOUR:FUNC?` -> return the current waveform type (for example `"SIN"`).

#### 4.2.2 `[SOURce#:]FUNCtion:SQUare:DCYCle` - Square-Wave Duty Cycle

**Description**: set or query the percentage of each period for which the square wave is high. **Valid only for the `SQUare` waveform**.

**Command format**:  
`[SOURce#:]FUNCtion:SQUare:DCYCle {<percent>|MINimum|MAXimum}`  
`[SOURce#:]FUNCtion:SQUare:DCYCle? [MINimum|MAXimum]`

**Parameters**:
- `<percent>`: floating-point percentage.
- Range: 0.01 % ~ 99.99 % (limited by hardware rise time; the extremes may not reach an ideal 100%).
- Default: 50.00 %.

**Examples**:
- `SOUR:FUNC:SQU:DCYC 30` -> set duty cycle to 30%.
- `SOUR:FUNC:SQU:DCYC?` -> return `"30.00"`.

**Error handling**:
- If the current waveform is not square (for example, sine), this command returns `-221,"Settings conflict"`.

#### 4.2.3 `[SOURce#:]FUNCtion:TRIangle:SYMMetry` - Triangle-Wave Symmetry

**Description**: set or query triangle-wave symmetry. **Valid only for the `TRIangle` waveform**.

**Command format**:  
`[SOURce#:]FUNCtion:TRIangle:SYMMetry {<percent>|MINimum|MAXimum}`  
`[SOURce#:]FUNCtion:TRIangle:SYMMetry?`

**Parameters**:
- `<percent>`: floating-point percentage.
- Range: 0.01 % ~ 99.99 % (limited by hardware rise time; the extremes may not reach an ideal 100%).
- Default: 50.00 %.

**Examples**:
- `SOUR:FUNC:TRI:SYMM 75` -> set symmetry to 75%.
- `SOUR:FUNC:TRI:SYMM?` -> return `"75.00"`.

**Error handling**:
- If the current waveform is not triangle, this command returns `-221,"Settings conflict"`.

### 4.2.4 `[SOURce[#]]:FUNCtion:NOISe:MEAN` - Set/Query Noise Mean

**Description**: set or query the noise signal's mean (DC component). **Valid only for the `NOISe` waveform**.

**Command format**:  
`[SOURce#:]FUNCtion:NOISe:MEAN {<mean>|MINimum|MAXimum}`  
`[SOURce#:]FUNCtion:NOISe:MEAN? [MINimum|MAXimum]`

**Parameters**:
- `<mean>`: floating-point mean (DC component) in V.
- Range:
  - `MIN`: minimum, -10 V.
  - `MAX`: maximum, +10 V.
- Default: 0 V.

**Examples**:
- `SOUR:FUNC:NOIS:MEAN 0.5` -> set noise mean to +0.5V.
- `SOUR:FUNC:NOIS:MEAN?` -> return `"0.50"`.

**Error handling**:
- If the current waveform is not noise, this command returns `-221,"Settings conflict"`.

### 4.2.5 `[SOURce[#]]:FUNCtion:NOISe:SDEViation` - Set/Query Noise Standard Deviation

**Description**: set or query the noise signal's standard deviation (a measure of its amplitude). **Valid only for the `NOISe` waveform**.

**Command format**:  
`[SOURce[#]]:FUNCtion:NOISe:SDEViation {<sdev>|MINimum|MAXimum}`  
`[SOURce[#]]:FUNCtion:NOISe:SDEViation?`

**Parameters**:
- `<sdev>`: floating-point standard deviation (a measure of noise amplitude) in V.
- Range:
  - `MIN`: minimum, 0 V.
  - `MAX`: maximum, 5.770 V.
- Default: 0 V.

**Examples**:
- `SOUR:FUNC:NOIS:SDEV 2.5` -> set noise standard deviation to 2.5V.
- `SOUR:FUNC:NOIS:SDEV?` -> return `"2.50"`.

**Error handling**:
- If the current waveform is not noise, this command returns `-221,"Settings conflict"`.

### 4.3 Amplitude `[SOURce#:]VOLTage`

**Description**: set or query output amplitude (peak-to-peak, Vpp).

**Command format**:  
`[SOURce#:]VOLTage[:AMPLitude] {<amplitude>|MIN|MAX}`  
`[SOURce#:]VOLTage[:AMPLitude]? [MIN|MAX]`

**Parameters**:
- `<amplitude>`: floating-point value in V (high-impedance load).
- `MIN`: ~-10 V.
- `MAX`: ~10 V.

**Example**:
- `SOURce1:VOLTage 2.5` -> set channel 1 output to 2.5 V.

### 4.4 Offset `[SOURce#:]VOLTage:OFFSet`

**Description**: set or query output offset.

**Command format**:  
`[SOURce#:]VOLTage:OFFSet {<offset>|MIN|MAX}`  
`[SOURce#:]VOLTage:OFFSet? [MIN|MAX]`

**Parameters**:
- `<offset>`: floating-point value in V (high-impedance load).
- `MIN`: ~-10 V.
- `MAX`: ~10 V.

**Example**:
- `SOURce1:VOLTage:OFFSet 2.5` -> set channel 1 output offset to 2.5 V.

## 5. `OUTPut` Subsystem (Physical Output Control)

This subsystem controls the physical relays and impedance-matching circuits in the signal generator's front end. **Both channels are controlled independently**, identified by `OUTPut#`.

### 5.1 Output Enable `OUTPut#:STATe`

**Description**: enable or disable the specified channel's signal output relay.

**Command format**:  
`OUTPut#:STATe {ON|OFF|1|0}`  
`OUTPut#:STATe?`

**Parameters**:
- `ON` or `1`: close the relay and output the signal.
- `OFF` or `0`: open the relay; the output is shorted to ground (muted).

**Examples**:
- `OUTPut1:STATe ON` -> enable channel 1.
- `OUTPut2:STATe OFF` -> disable channel 2.
- `OUTPut:STATe?` -> without a channel number, query the currently selected channel (the last operated channel; channel one at power-up). Specifying the channel number explicitly is recommended.

### 5.2 Output Impedance `OUTPut#:IMPedance`

**Description**: set the displayed load impedance and amplitude/offset values. This affects calculation coefficients only, not the physical output impedance.

**Command format**:  
`OUTPut#:IMPedance {50|HIGH}`

**Parameters**:
- `50`: match a 50Ω system; displayed amplitude is the actual load voltage (half the setpoint).
- `HIGH`: high-impedance mode; displayed amplitude is the open-circuit voltage (setpoint equals actual output).

---

## 6. System and Status `SYSTem`

### 6.1 Error Queue Query `SYSTem:ERRor?`

**Description**: read and remove the oldest error from the queue.

**Command format**:  
`SYSTem:ERRor?`

**Response format**:  
`<error-code>,"<error-description>"`

- With no errors: `0,"No error"`.

---

## 7. Common Error Codes

| Code | Description | Resolution |
| :--- | :--- | :--- |
| 0 | No error | - |
| -102 | Syntax error | Check spelling, such as `FRQ` instead of `FREQ` |
| -113 | Undefined header | Unsupported command; check the manual |
| -222 | Data out of range | Frequency exceeds 30MHz or is below 1mHz |
| -224 | Illegal parameter value | Invalid waveform type or Boolean, such as `SQURE` |
| -240 | Hardware error | Analog board ACK error |

---

## 8. Programming Examples (Python + PyVISA)
The USB CDC interface supports communication through either PyVISA or PySerial. The following examples show how to control ArbWave30.

### Method 1: PySerial (Simplest and Most Reliable)

```python
import serial
import time

# 1. Configure the serial port (CDC usually ignores baud rate; retained for compatibility).
# Windows example: 'COM3'; Linux example: '/dev/ttyUSB0' or '/dev/ttyACM0'.
ser = serial.Serial(
    port='COM3',       # Replace with your actual port.
    baudrate=115200,   # Does not affect CDC speed; keep both ends consistent.
    timeout=2,         # Read timeout in seconds to prevent readline from blocking indefinitely.
    write_timeout=2
)

# 2. Send a SCPI command; always append a newline.
ser.write(b'*IDN?\n')

# 3. Read the response (SCPI responses end with '\n').
response = ser.readline().decode('utf-8').strip()
print(f"Device Info: {response}")

# 4. Example sequence.
ser.write(b'*RST\n')             # Reset.
time.sleep(0.1)                  # Allow MCU processing time (optional).
ser.write(b'SOUR1:FREQ 1000\n')  # Set 1kHz.
ser.write(b'SOUR1:FUNC SIN\n')   # Sine wave.
ser.write(b'OUTP1:STAT ON\n')    # Enable output.

# 5. Query frequency.
ser.write(b'OUTP1:FREQ?\n')
freq = ser.readline().decode('utf-8').strip()
print(f"Current Frequency: {freq} Hz")

# Close the serial port.
ser.close()
```

### Method 2: PyVISA (More Powerful)

```python
import pyvisa

# 1. Select the pure-Python backend (no NI-VISA driver required).
rm = pyvisa.ResourceManager('@py')

# 2. List resources to find the COM port (typically in a format such as 'ASRL3::INSTR').
# print(rm.list_resources())

# 3. Open the serial resource and configure terminators.
inst = rm.open_resource('ASRL3::INSTR')  # Replace with the actual ASRL port number.
inst.baud_rate = 115200
inst.read_termination = '\n'   # Important: declare explicitly even though '\n' is the default.
inst.write_termination = '\n'  # Automatically append a newline when sending.

# 4. Use query as with a USBTMC instrument.
print(inst.query('*IDN?'))
inst.write('SOUR:FREQ 1000')
inst.write('SOUR:STAT ON')

# 5. Query.
freq = inst.query('SOUR:FREQ?')
print(f"Freq: {freq}")

inst.close()
```

## 9. Revision History

| Version | Date | Changes | Author |
|------|------|----------|--------|
| V1.0 | 2026-08-17 | Initial structure | EERNINUO |
| V1.1 | 2026-08-18 | Add commands | EERNINUO |
| V1.2 | 2026-09-25 | Draft MVP command set | EERNINUO |
