<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->


<!-- Assisted-by: DeepSeek - Document framework -->

# ADR-001: Selecting the Gowin GW2AR-18 LQFP144 as the Main FPGA

## Status
Accepted (2026-04-05)

## Context
The ArbWave30 project requires an FPGA to implement:
- A 48-bit DDS phase accumulator (1μHz frequency resolution)
- Waveform memory (sine, square, triangle, sawtooth, and staircase lookup tables, with future arbitrary-waveform support)
- A 16-bit parallel interface driving the AD9747 DAC (150MSPS)
- An SPI slave interface receiving configuration commands from the STM32
We need an FPGA with a hand-solderable package, sufficient logic resources, and a reasonable cost.

## Decision Drivers
- Solderability: This is a personal DIY project; the package must be solderable with a hot-air gun or a hand soldering iron (no BGA).
- Logic resources: At least approximately 10k~15k LUTs are required (DDS + waveform tables + SPI slave + parallel output).
- Memory: Waveform tables require Block RAM or external SDRAM. At least 4 waveforms must be stored (1024 points × 14bit each ≈ 7kB); future arbitrary waveforms may require more capacity (>64kB).
- Cost: The price per FPGA should be kept below CNY 70.
- Development tools: Good IDE support is required (Gowin software is free and supports a logic analyzer).

## Options Considered
### Option A: Gowin GW2A-18 LQFP144
- Logic resources: 18k LUTs
- Package: LQFP144 (hand-solderable)
- Internal BSRAM: Approximately 828kbit (82kB)
- No integrated SDRAM; external memory is required
- Relatively extensive tutorials and community support; suitable for beginners
- Unit price: Approximately CNY 45~55 (not in stock on Taobao/LCSC)

### Option B: Gowin GW2AR-18 LQFP144
- Logic resources: 18k LUTs
- Package: LQFP144 (hand-solderable)
- Internal BSRAM: Approximately 828kbit (82kB)
- Integrated PSRAM/SDRAM: 64Mbit (8MB)
- Relatively fewer tutorials and less community support
- Unit price: Approximately CNY 45~55

### Option C: Gowin GW2A-55 BGA
- Logic resources: 55k LUTs (excessive)
- Package: BGA (difficult to hand-solder; requires machine assembly)
- Abundant integrated resources, but the PCB requires 4 or more layers and precision routing
- Relatively extensive tutorials and community support; suitable for beginners
- Unit price: Approximately CNY 45~55

### Option D: Xilinx Artix-7 XC7A35T BGA
- Logic resources: 33k LUTs (excessive)
- Package: BGA (difficult to hand-solder)
- Abundant integrated resources, but a higher price and a complex development environment
- The most extensive tutorials and community support; a classic entry-level FPGA
- Unit price: >CNY 170

### Option E: ZYNQ-7000 Series SoC
- Logic resources: 28k logic cells (excessive)
- Package: BGA (difficult to hand-solder)
- Integrated ARM processor; high development complexity and cost
- Relatively extensive tutorials and community support, primarily focused on embedded Linux development
- Unit price: Approximately CNY 200 (SoC only; currently high memory prices must also be considered) (ex-mining boards may be cheaper, but their reliability is unknown)
- **Too complex and expensive for the current project requirements, but potentially suitable for a more advanced future version.**

## Decision
Select Option B: Gowin GW2AR-18 LQFP144.

### Rationale
1. The LQFP144 package can be soldered directly using a hot-air gun or a knife-tip soldering iron, making it suitable for personal DIY work; routing requires only a 4-layer PCB.
2. The integrated 64Mbit SDRAM is the decisive advantage:
   - No external SDRAM chip is needed, saving PCB area and reducing routing complexity.
   - The 8MB storage capacity is sufficient for extensive arbitrary-waveform tables (the future V3.0 is planned to support ≥1Mpts).
   - Internal SDRAM access is fast enough for waveform reads at 125MSPS (with a controller built into the FPGA).
3. The 18k LUTs are sufficient for the DDS core (approximately 6k), parallel DAC interface (approximately 2k), SPI slave (approximately 1k), and future digital modulation (approximately 3k), leaving approximately 30% headroom without resource pressure.
4. It costs only approximately CNY 5 more than the GW2A, while eliminating external SDRAM and its routing costs, making it the better overall choice.
5. The Gowin development environment has mature GW2AR support and provides free SDRAM controller IP (or a controller can be written independently).
6. This is a Chinese-made FPGA with a relatively stable supply chain and very competitive pricing.

## Consequences
### Positive
- Simplified PCB design: Only one FPGA power-supply layer (1.0V, 3.3V) is needed, with no external-memory routing.
- No capacity pressure for future arbitrary-waveform expansion.
- Reduced soldering and debugging difficulty.
- Free Gowin software facilitates development and debugging.

### Negative
- Approximately CNY 5~15 more expensive than a GW2A-only solution (acceptable).
- The integrated-SDRAM variant has fewer pins, and some die resources are used for SDRAM control.
- This integrated-SDRAM feature may not be available if switching to another FPGA vendor in the future.

### Reasons for Rejecting Alternatives
**GW2A-18**:
1. Cheaper if arbitrary waveforms are unnecessary or only small storage capacity is needed. However, the V3.0 plan requires large arbitrary-waveform capacity. External SDRAM would increase PCB area and debugging time, and high-speed SDRAM routing is prone to signal-integrity problems. The trade-off is not worthwhile.
2. The GW2A-18 LQFP144 is not in stock on Taobao/LCSC.

**GW2A-55 BGA**:
- Not hand-solderable, with significantly excessive resources and a high price; unsuitable for the prototype stage.

**Xilinx Artix-7**:
1. Although resources are sufficient, the high price and complex development environment do not suit the current project requirements.
2. The BGA package increases soldering difficulty and PCB design complexity.
3. Xilinx development tools (Vivado) are not particularly friendly to individual DIY users and have a steep learning curve.
  
**ZYNQ-7000 SoC**:
1. Too complex, expensive, and difficult to develop for the current project requirements.
2. Although the integrated ARM processor provides substantial processing power, it is overkill for the current DDS and waveform-generation requirements and increases development complexity and cost.
3. Potentially suitable for a more advanced future version, but not recommended at this stage.

## Related Documents
Technical Specification, Section 5.1
Hardware Architecture Document, Section 3.2
Gowin Semiconductor GW2AR Datasheet
Gowin Semiconductor SDRAM Controller IP Documentation
