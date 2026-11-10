<!------------------------------------------------------------------------
SPDX-License-Identifier: CC-BY-SA-4.0
Copyright (C) 2026 [EERNINUO]

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.
------------------------------------------------------------------------->

# ArbWave30 - Control Board Debugging Notes

## DC-DC Output Voltage Test
**Date**: 2026-6-23  
**Hardware version**: v0.1  
**Purpose**: verify that the control board DC-DC output voltages are correct.  
**Equipment**: ZT-219 digital multimeter, RK6006 programmable power supply.  
**Procedure**:
1. Use the ZT-219 in diode mode to measure the voltage drop from the positive power input to ground.
2. Connect the RK6006 set to 24V to the control board, and measure voltages at the power test points (+17V, -17V, +7V, -7V, +3.3V).

**Test data**:
