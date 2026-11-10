<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->


<!-- Assisted-by: DeepSeek - Document framework -->

# ADR-004: Filter Topology Selection

## Status
Accepted (2026-05-09)

## Context
The AWG outputs 30MHz sine waves and 5MHz square waves. The DAC is an AD9747 (150MSPS), requiring a reconstruction filter to suppress sampling images.

## Decision Drivers
1. Passband insertion loss (especially at 30MHz) must be < 0.5dB
2. Suppression at the image frequency (120MHz) must be > 60dB
3. Square-wave overshoot should be minimized (preferably < 10%)
4. Inductors must be commercially available high-Q models to avoid excessive DCR

## Alternatives and Rationale

**Option A: 5th-order elliptic, 35MHz cutoff**
- ✅ Excellent stopband rejection (>100dB@120MHz)
- ❌ Large inductance values required (>300nH); low Q results in 0.7dB insertion loss at 30MHz
- ❌ High inductor DC resistance (>3Ω)


**Option B: 7th-order elliptic, 35MHz cutoff**
- ✅ Excellent stopband rejection (>100dB@75MHz)
- ❌ Large inductance values required (>300nH); low Q results in 0.45dB insertion loss at 30MHz, but actual inductor tolerances may prevent achieving the simulated performance
- ❌ High inductor DC resistance (>3Ω)

**Option C: 7th-order elliptic, 60MHz cutoff (selected)**
- ✅ Small inductance values (~120-180nH), allowing high-Q wirewound inductors (Q@60MHz>70)
- ✅ Simulated insertion loss at 30MHz <0.5dB (using Coilcraft 0805HP models)
- ✅ Rejection at 120MHz >90dB (far exceeding the 60dB requirement)
- ⚠️ Square-wave overshoot ~13% (acceptable, and reducible through software predistortion or rise-time adjustment)
- ✅ Relatively smooth group-delay variation (because the 5MHz harmonics are far from the cutoff region)

**Option D: 7th-order elliptic, 60MHz cutoff, RC snubber**
- ✅ Small inductance values (~120-180nH), allowing high-Q wirewound inductors (Q@60MHz>70)
- ✅ Square-wave overshoot ~8%
- ✅ Rejection at 120MHz >90dB (far exceeding the 60dB requirement)
- ❌ Simulated insertion loss at 30MHz >1.2dB (the RC snubber absorbs too much high-frequency energy, causing excessive passband loss)
- ❌ Increased design complexity; RC values must be adjusted to balance overshoot and insertion loss, with sensitivity to component parameters

**Option E: 7th-order Chebyshev Type II**
- ✅ Flat passband and better group-delay characteristics
- ⚠️ Lower stopband rejection than an elliptic filter of the same order
- ⚠️ Sensitive to component parameters; actual inductor Q and tolerances may significantly degrade performance
- **Decision**: Reserve as a V2.0 upgrade option (pin-to-pin replacement possible)

## Decision
Select a **7th-order elliptic filter with a 60MHz cutoff**. Track square-wave overshoot as a known issue for later optimization through software predistortion, rise-time adjustment, or a separate square-wave channel.

## Rationale
- With commercially available inductors tested (such as the AHW2012 series), Q at 60MHz can reach 60+, with measured/simulated insertion loss at 30MHz <1dB.
- Rejection of 90dB@120MHz far exceeds the requirement (65dB is sufficient).
- Simulation confirms square-wave overshoot of approximately 13%~18%, which is acceptable (and can be reduced through software predistortion or rise-time adjustment).
- Compared with Chebyshev II, an elliptic filter places relatively lower demands on component parameters and PCB layout, making it suitable for the first version.

## Consequences
- Positive: A wide choice of inductors, low passband loss, and excellent image rejection.
- Negative: Approximately 13%~18% square-wave overshoot requires optimization in subsequent designs.

## Related Links
- [Reconstruction Filter Debugging Notes](/docs/design-notes/reconstruction-filter-debugging.md)