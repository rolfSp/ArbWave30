<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->

<!-- Assisted-by: DeepSeek - Document framework -->

# ADR-006: Returning to a CFB Topology for Differential-to-Single-Ended Conversion (Supersedes ADR-005)

## Status
Accepted (2026-09-26), supersedes ADR-005

## Context
Based on impedance calculations, ADR-005 selected the LMH6609 (VFB) to avoid the
50Ω matching difficulty caused by the low inverting-input impedance of the OPA695 (CFB).
However, analog-board MVP measurements showed that the VFB solution could not meet flatness requirements
over the target frequency range (up to 30MHz):

- Approximately -9.5dB attenuation at 30MHz (approximately 1/3 of the original amplitude)
- -1.2dB attenuation at 10MHz, with the output lagging the non-inverting input by approximately 38°
- A phase difference approaching 90° between the inverting input (V-) and non-inverting input (V+), with an abnormally high V- peak-to-peak voltage
- Repeating the measurements with a 10:1 probe produced the same results, ruling out measurement artifacts caused by loading from a direct 50Ω coaxial connection

## Root-Cause Analysis
The high input impedance at the V- node, together with substantial parasitic capacitance, creates a pole at approximately 25MHz. This pole invalidates the VFB virtual-short approximation at high frequencies:

- 10MHz: Magnitude -1.2dB, phase -38°
- 30MHz: Cumulative roll-off reaches -9.5dB

Topological cause: The inverting input of a VFB amplifier is a high-impedance node, sensitive to stray/parasitic capacitance
(see ADI MT-040).

## Decision Drivers (in Priority Order)
1. Amplitude flatness and phase linearity up to 30MHz (newly added; omitted in the original ADR)
2. Retaining the existing analog-board MVP without another fabrication run
3. Cost
4. A 50Ω input load (the design load of the 7th-order elliptic filter)

## Options Considered
### Option One: Return to OPA695 (CFB), Rf = 560Ω
- Advantages: The CFB inverting input is a low-impedance node with low sensitivity to parasitics; CFB bandwidth BW ≈ 1/(2π·Rf·Cc) is independent of gain, allowing gain and matching to be decoupled.
- Disadvantages: Rf is approximately 100Ω higher than the recommended value (approximately 460Ω) → slightly lower bandwidth and slightly higher thermal noise and offset.
- Measurements: Attenuation at 30MHz improved from -9.5dB to -1.2dB.

### Option Two: Retain LMH6609 (VFB) + Preceding Buffer
- Advantages: Adding a follower reduces the source impedance at V- from hundreds of Ω to a few Ω, moving the pole to a higher frequency.
- Disadvantages: Requires PCB changes; the additional op amp introduces noise and offset.
- Measurements: Not verified (Option One already meets requirements).

### Option Three: OPA695 (CFB) + Preceding Buffer
- Advantages: Low and symmetric input capacitance, good high-frequency CMRR, independent matching and gain, and the ability to use the datasheet-recommended Rf value.
- Disadvantages: Requires PCB changes, component reselection, and validation, with higher cost and a longer schedule.
- Conclusion: Use Option One and Option Three on the two channels of the next MVP revision to evaluate performance.

## Decision
Return to OPA695 (CFB), Rf = 560Ω.

Rationale: The CFB inverting input is a low-impedance node with low parasitic sensitivity, eliminating the VFB solution's high-frequency bottleneck at the topology level. CFB bandwidth is also independent of gain, allowing gain and input matching to be decoupled. Setting Rf approximately 100Ω above the recommended value is a deliberate trade-off: approximately 20% of bandwidth is exchanged for matching the input impedance to the filter output. Since 30MHz is only approximately 5% of the available bandwidth, ample margin remains.

## Consequences
### Positive
- Attenuation at 30MHz improves from -9.5dB to -1.2dB, meeting the target
- No new fabrication run is needed, leaving the MVP schedule unaffected
- Clarifies the VFB/CFB trade-off in parasitic-capacitance sensitivity and provides design guidance for subsequent versions

### Negative
- Rf deviates from the recommended value: Slightly lower bandwidth and slightly higher thermal noise and DC offset require recalibration
- Additional CFB constraints: No compensation capacitor may be placed across Rf; control capacitance at the inverting input and output;
  recheck DC offset and input matching

## Related Documents
- ADR-005 (superseded by this ADR)
- /docs/hardware-architecture.md
- /docs/hardware-design/analog-board.md
