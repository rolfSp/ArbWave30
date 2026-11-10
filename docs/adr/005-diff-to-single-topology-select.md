<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->

<!-- Assisted-by: DeepSeek - Document framework -->

# ADR-005: Differential-to-Single-Ended Circuit Topology Selection

## Status
Deprecated (2026-09-26)

## Context
Calculations for the original differential-to-single-ended circuit show that, with the OPA695 at gain G = 6 V/V, the inverting-input impedance is $Z_{Non} \approx 39.4 \Omega$, so a parallel resistor cannot achieve a 50Ω input impedance. The circuit topology therefore needs to change to meet the filter's output-load requirement.

## Decision Drivers
- Ease of PCB modification: The analog-board MVP has already been fabricated; prefer a solution that can be implemented on the existing PCB.
- Cost: Keep total cost as low as possible, preferably below 50.
- Circuit performance: Requirements for gain, stability, noise, bandwidth, slew rate, etc.

## Options Considered
### Option One: Retain the OPA695 and Add a 10.6 Ω Series Resistor at the Inverting Input
- Rationale: A series resistor at the inverting input can decrease/increase the input impedance to achieve a 50Ω input impedance.
- Ease of PCB modification: No additional PCB changes are needed; the new series resistor can be combined with the inverting-input resistor into 83.8 Ω.
- Cost: Only additional resistors need to be purchased.
- Circuit performance:
  - Noise, bandwidth, slew rate: Unaffected because the op amp is unchanged.
  - Stability: Adding series resistance at the differential amplifier's inverting input actually changes the gain, requiring recalculation of the feedback resistor; otherwise stability is likely to be affected.
  - Gain: Series resistance at the inverting input reduces gain.
  - Common-mode rejection ratio (CMRR): Adding series resistance only at the inverting input makes the circuit asymmetric, with unequal inverting and non-inverting gains, affecting CMRR.
  
### Option Two: Replace with a Voltage-Feedback Op Amp
- Rationale: Voltage-feedback op amps allow a wider range of feedback resistances, avoiding the limitation of current-feedback op amps that require a specific resistor for a given gain, preventing a 50Ω input impedance.
- Ease of PCB modification: The op amp must be replaced. An SOIC-8 package allows direct replacement; other packages require PCB redesign.
- Cost: Additional op amps must be purchased.
- Op amp comparison:

| Parameter | OPA695 | OPA690 | OPA820 | LMH6609 | LMH6624 | LMH6629 |
| --- | --- | --- | --- | --- | :--- | --- |
| Topology | Current feedback (CFB) | Voltage feedback (VFB) | Voltage feedback (VFB) | Voltage feedback (VFB) | Voltage feedback (VFB) | Voltage feedback (VFB) |
| -3dB bandwidth | 1900MHz (G=1) | 535MHz (G=1) | 240MHz (G=2) | 900MHz (G=1) | 1.5 GHz | 900 MHz |
| Slew rate (SR) | 5000 V/µs | 1900 V/µs | 240 V/µs | 1400 V/µs | 350 V/µs | 1600 V/µs |
| Output current (Iout) | ±140 mA | ±215 mA | ±110 mA | ±90 mA | - | ±250 mA |
| Voltage noise (eₙ) | 2 nV/√Hz | 4.6 nV/√Hz | 2.5 nV/√Hz | 3.1 nV/√Hz | 0.92 nV/√Hz | 0.69 nV/√Hz |
| Supply voltage | 5V to 12V | 5V to 12V | 5V to 12V | 6.6V to 12V | 5V to 12V| 2.7V to 5.5V |
| Minimum stable gain | Flexible (CFB characteristic) | Unity-gain stable | Unity-gain stable | Unity-gain stable | ≥ 10 | ≥ 4 or 10 |
| Package | SOIC-8 | SOIC-8 | SOIC-8 | SOIC-8 | SOIC-8 | SOT-23 |
| TI website price | $2.274 | $3.520 | $2.245 | $1.657 | $3.128 | $3.300 |
| LCSC price[^1] | ￥11.29 | ￥39.64 | ￥20.14 | ￥34.06 | ￥41.54 | ￥32.02 |

> Note: Taobao prices are lower than those listed above and close to TI's volume pricing, but the parts may be salvaged, so caution is required.

[^1]: LCSC prices as of 2026-08-09.

- Circuit performance:
  - Noise, bandwidth, slew rate: Refer to the table above
  - Stability: Voltage-feedback op amps have better stability and do not require additional feedback-resistor calculations
  - Gain: Voltage-feedback op amps allow a wider range of feedback resistances, avoiding the limitation of current-feedback op amps that require a specific resistor for a given gain, preventing a 50Ω input impedance.
  
### Option Three: Add Voltage Followers Before the Differential-to-Single-Ended Stage
- Rationale: Add voltage followers before the differential-to-single-ended stage, handling 50Ω matching at the followers so the differential amplifier need not account for input impedance.
- Ease of PCB modification: Additional PCB changes are required to add the voltage-follower circuit.
- Cost: Additional op amps must be purchased, and the PCB must be redesigned and fabricated.
- Circuit performance:
  - Noise: The extra op amp stage increases noise to some extent.
  - Bandwidth: The OPA695 is a current-feedback op amp whose bandwidth does not change with gain, so the preceding voltage followers are the bandwidth bottleneck. Since their gain is G = 1 rather than G = 6 V/V in Option Two, the required bandwidth is lower, making op amp selection easier.
  - Slew rate: Unchanged because the output-stage op amp is not replaced.
  - Stability: Feedback remains local to each stage, so the effect on stability is small.
  - Gain: The voltage followers have a gain of 1 and the differential amplifier a gain of 6, giving an overall gain of 6 V/V.
  - CMRR: The voltage-follower op amps are likely to have different offset voltages, reducing CMRR.

## Decision
Since the analog-board MVP has already been fabricated, **temporarily adopt Option Two and evaluate the LMH6609. If measured performance is unsatisfactory, switch to Option Three for analog-board v1.0.**

## Consequences
### Positive
- Direct op amp replacement without PCB modification.
- Meets the 50Ω input-impedance requirement.
- Circuit performance meets requirements.
- Voltage-feedback op amps impose less stringent parasitic requirements and have better stability.

### Negative
- Additional op amps must be purchased.
- High-bandwidth voltage-feedback op amps are more expensive.

### Reasons for Rejecting Alternatives
- Option One: Affects stability, CMRR, and gain.
- Option Three: Requires additional PCB changes and increases cost, while the voltage followers introduce extra noise.

## Related Documents
- [Hardware Architecture Document](/docs/hardware-architecture.md)
- [Hardware Design Document (Analog Board)](/docs/hardware-design/analog-board.md)

> **2026-09-19 revision**: Measurements disproved this decision (the LMH6609 showed approximately -9.5dB attenuation at 30MHz, with a parasitic pole at the inverting input; the CFA's Rf is not a rigid constraint, and a slightly larger Rf affects only bandwidth while actually improving stability).