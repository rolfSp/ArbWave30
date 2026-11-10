<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->


<!-- Assisted-by: DeepSeek - Document framework -->

# ADR-003: Main DAC Selection — AD9747 vs AD9767

**Status**: Accepted
**Decision Maker**: EERNINUO
**Date**: 2026-04-29

## Context and Problem
ArbWave30 requires a high-speed DAC for waveform output at sampling rates up to 125MSPS or higher. The original design used the AD9767 (14-bit, 125MSPS, dual-channel). However, detailed design revealed two issues:
1. Pin layout: The AD9767 clock input is between two groups of parallel data pins, causing high-speed clock and data traces to cross on the PCB, increasing layout difficulty and crosstalk risk.
2. Offset generation: An additional 16-bit DAC (DAC8562) is needed to generate the DC offset voltage (±2V range), increasing the BOM and PCB area.

The alternative AD9747 (16-bit, 250MSPS, dual-channel) has the following features:

- Differential clock input pins on the side of the chip, separate from the data pins, for a cleaner PCB layout.
- Two integrated 10-bit auxiliary DACs that can generate offset voltages, potentially eliminating the separate offset DAC.
- A **data clock output (DCO)** interface: The DAC outputs a synchronous clock (DCO) to the FPGA, which uses it to latch data. The interface supports up to 250MSPS.

However, it also introduces new challenges:
- The Gowin GW2AR FPGA I/O switching limit is approximately 1.3ns (approximately 380MHz), theoretically supporting 250MSPS, but the DCO interface setup/hold requirements (400ps/1200ps) are very stringent.
- DCO propagation delay on the PCB makes FPGA timing constraints more difficult.
- The auxiliary DACs have only 10-bit resolution, below the originally selected 16-bit dedicated DAC, potentially affecting offset accuracy.
- The differential clock input requires an AC-coupled conversion circuit and cannot connect directly to a standard LVDS clock source (such as CDCI6214), imposing routing requirements (such as 100Ω differential impedance).

## Evaluation Criteria
| Criterion | Weight | Description |
|------|------|----------|
| PCB layout friendliness | High | Whether clock and data pins require crossing traces, and how easily trace lengths can be matched |
| Signal-integrity risk | High | Clock jitter, crosstalk, and the need for layer transitions through vias |
| Procurement and cost | Medium | Sample pricing and availability (small project quantities; not highly cost-sensitive) |
| Interface design complexity | Medium | Difficulty of FPGA timing constraints and need for additional logic |
| Performance expansion potential | Low | Support for higher sampling rates for future upgrades |
| Integration / BOM simplification | Low | Reduction of external components (such as an offset DAC) |

## Analysis
### 1. PCB Layout Friendliness
- AD9767: The clock pins are between the DB13P1\~DB0P1 and DB13P2\~DB0P2 buses. Length-matched routing requires data traces to detour around the clock trace on either side, or the clock trace to change layers through vias to avoid the data buses. This increases layout difficulty and signal-integrity risk.

- AD9747: The clock pins are on the side of the device (pins 3 and 4), while the data pins are grouped on two other sides, providing natural separation. The clock traces can exit directly and the data traces can be routed independently, significantly simplifying routing.

Conclusion: AD9747 has a clear advantage.

### 2. Signal-Integrity Risk
- AD9767: Changing clock routing layers to avoid data traces introduces a reference-plane transition. If handled incorrectly on a four-layer board, this can introduce jitter and radiation.

- AD9747: The differential clock input inherently rejects common-mode interference well. Although an AC-coupled conversion circuit is required, it is simple and proven (used on ADI's official evaluation board). Clock traces can exit directly from the side without layer transitions, maintaining an uninterrupted reference plane.

Conclusion: AD9747 is preferred.

### 3. Procurement and Cost
Both are ADI production devices available through authorized channels (LCSC). The AD9767 is moderately priced (approximately CNY 65\~70), while the AD9747 is slightly more expensive (approximately CNY 90\~100), but eliminates the external offset DAC (DAC8562, approximately CNY 30). The net cost increase is approximately CNY 20 and is negligible. In addition, resolving the AD9767 clock-routing issue may require a six-layer board, which costs more than a four-layer board.

Conclusion: AD9747 is preferred.

### 4. Interface Design Complexity
AD9767: A standard parallel interface, with the FPGA providing clock and data. Timing is simple and readily achievable on a Gowin FPGA at 125MHz.

AD9747: Dual-port mode requires the FPGA to receive the DAC's DCO clock and use it to latch output data. Setup/hold requirements are stringent (400ps/1200ps). The FPGA must adjust its internal delay. The resolution of the Gowin FPGA I/O delay elements needs verification.

Conclusion: AD9767 is preferred (lower development risk).

### 5. Performance Expansion Potential
The AD9747 supports up to 250MSPS, versus only 125MSPS for the AD9767. Although the current project uses only 125MSPS (potentially increasing to 150\~200MSPS for better dynamic performance), the AD9747 provides more headroom for future arbitrary-waveform upgrades or higher sine-wave frequencies.

Conclusion: AD9747 is preferred.

### 6. Integration / BOM Simplification
AD9767: Requires an additional 16-bit DAC (such as DAC8562) to generate a ±2V offset voltage, increasing the BOM and PCB area.

AD9747: Integrates two 10-bit auxiliary DACs. Does the 10-bit resolution (approximately 4.88mV steps) fail to meet the ≤2mV resolution requirement over a ±10V offset range? Even so, **coarse adjustment with the auxiliary DAC + fine adjustment with the main DAC** can be used. For stricter requirements, the external offset DAC can still be retained, and the auxiliary DACs used for other functions (such as controlling attenuation relays).

Conclusion: AD9747 is preferred (can eliminate the separate offset DAC).

## Decision
**Select the AD9747 as the main DAC for ArbWave30.**

Despite the increased interface design complexity, its significant advantages in PCB layout, signal integrity, integration, and long-term project expandability make it the better choice. Careful FPGA timing design can address the interface complexity, and official reference designs are available.

## Implementation Plan
1. Clock input circuit: Design the AC-coupled circuit according to Figure 29 of the AD9747 datasheet:
- CDCI6214 LVDS differential output → 0.1µF capacitors → AD9747 CLKP/CLKN
- Use 1kΩ and 287Ω resistors to divide CVDD18 and generate a ~400mV common-mode voltage
- Place a 100Ω differential termination resistor across the pair near the AD9747 pins

2. FPGA interface:
- Use pins supporting high-speed output in the Gowin GW2AR I/O banks
- Use dual-port mode at an initial sampling rate of 125MSPS (increase to 150~200M if timing margin permits)
- Receive DCO and use a PLL or IODELAY to adjust the internal data-output clock phase to meet setup/hold requirements

3. Auxiliary DAC usage:
- Connect auxiliary DAC1 to the offset summing node to generate a ±2V offset
- Reserve auxiliary DAC2 for future use (attenuation-relay control or expansion)
- Retain optional external DAC8562 pads in case offset accuracy is insufficient

3. PCB layout:
- Use a 4-layer analog board (stackup: TOP - GND - PWR - BOT)
- Route the differential clock at 100Ω impedance with strict length matching (error ≤5mil)
- Place CDCI6214 close to AD9747, with the AC-coupling circuit immediately adjacent to the DAC clock pins

## Consequences
### Positive
- Significantly easier PCB layout and more assured signal integrity
- Elimination of the external offset DAC simplifies the BOM and reduces PCB area
- Headroom for future sampling-rate increases

### Negative
- More complex FPGA timing design, requiring careful constraints and validation
- Requires 5 additional passive components (2 capacitors + 4 resistors) for clock conversion
- If the Gowin FPGA cannot operate reliably above 125MSPS, the sampling rate must be reduced or the design reverted to AD9767

## Related Documents
Technical Specification, Section 5.1
Hardware Architecture Document, Section 3.2
Gowin Semiconductor GW2AR Datasheet
AD9767 Datasheet
AD9747 Datasheet
