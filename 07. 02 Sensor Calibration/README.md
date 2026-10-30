# O2 Sensor Calibration

Ladder logic for an O2 concentration monitor installed at the bottom of a coal mine. The sensor drifts over time, so the program includes an automatic two-point calibration cycle that re-tunes its own scaling parameters.

Target: Allen-Bradley MicroLogix (RSLogix Micro Starter Lite, tested with RSLogix Emulate).

## 1. Overview

The machine works in two modes:

- **Sampling:** continuously measures the O2 concentration of the air passing the sensor.
- **Calibration:** the sensor reads 0-40% O2. It is checked against two known gases, 0% and 30% O2, and the averaged readings are used to recompute the SCP block's input limits.

## Calibration Cycle

1. The calibration button starts the cycle (one-shot).
2. The **0% gas valve** opens and the raw sensor value is sampled once per second for 30 s. The average is stored as `O2_Zero_Average`.
3. The **30% gas valve** opens and the sampling repeats for another 30 s. The average is stored as `O2_Test_Gas_Average`.
4. The new scaling limits are calculated and the cycle ends.

### Calibration formulas

```
Input Min = O2_Zero_Average
Input Max = ((O2_Maximum_Concentration / O2_Calibration_Gas_Concentration)
             x (O2_Test_Gas_Average - O2_Zero_Average)) + O2_Zero_Average

O2_Maximum_Concentration         = 40 (%)
O2_Calibration_Gas_Concentration = 30 (%)
```

The averages must be taken from the **raw** sensor value (`N7:0`), not from the scaled output, because the scaling is exactly what the calibration changes.

## 2. Memory Map

| Address            | Description                            |
| ------------------ | -------------------------------------- |
| `B3:0/0`           | Calibrate button                       |
| `B3:0/4`           | Calibrate bit                          |
| `B3:0/3`           | Cycle trigger bit (calibration active) |
| `B3:0/7`           | 0% gas phase active                    |
| `B3:0/8`           | 30% gas phase active                   |
| `B3:0/9`           | One-shot storage bit                   |
| `B3:0/1`, `B3:0/2` | Valve bits for 0% and 30% gas          |
| `O:0/0`, `O:0/1`   | Physical outputs: 0% and 30% O2 valves |
| `N7:0`             | Raw sensor value (input only)          |
| `N7:7`             | Measurement count                      |
| `F8:0`             | O2_Zero_Average (raw)                  |
| `F8:1`             | O2_Test_Gas_Average (raw)              |
| `F8:2`             | Input Min (calibrated)                 |
| `F8:3`             | Input Max (calibrated)                 |
| `F8:4`             | Temporary calculation value            |
| `F8:5`             | Scaled O2 concentration (%)            |
| `F8:6`             | Sum of raw samples                     |
| `T4:0`             | 0% phase timer (30 s)                  |
| `T4:1`             | 30% phase timer (30 s)                 |
| `T4:2`             | 1 s sample timer                       |

## 3. Setup and Testing

1. Set the initial values in the data table: `F8:2 = 0`, `F8:3 = 16383`, and `F8:6`, `N7:7` = 0.
2. Make sure MAIN calls IO, SAMPL_CYCL and CALC_CYCL with JSR.
3. Run the program on the emulator and toggle `B3:0/0` to start a cycle.
4. During the 0% phase set `N7:0` to about 350, and during the 30% phase to about 12100.

Expected result with these example inputs:

| Address | Value       |
| ------- | ----------- |
| `F8:0`  | about 350   |
| `F8:1`  | about 12100 |
| `F8:2`  | about 350   |
| `F8:3`  | about 16017 |

`B3:0/7` should stay on for 30 s, then `B3:0/8` for another 30 s, and both should end off.
