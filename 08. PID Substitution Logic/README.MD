# Vacuum Control System — RSLogix 500

## Project Overview

This project implements a vacuum control system using PLC Ladder Logic in **RSLogix 500**.

The system monitors the measured vacuum, compares it with a user-defined setpoint, and automatically adjusts the vacuum pump output based on the calculated error.

The pump output is adjusted every **10 seconds**, using three correction levels depending on the deviation from the setpoint.


## System Operation

### 1. Vacuum Measurement

The system reads the raw analog input from `N7:0` and converts it into an engineering-unit vacuum measurement stored in `N7:3`.

For the provided test configuration:

- Raw input: `N7:0 = 8192`
- Expected measured vacuum: approximately `15`
- Vacuum setpoint: `N7:2 = 15`

### 2. Error Calculation

The error is calculated as the absolute difference between the measured vacuum and the setpoint.

**Absolute Error:**


Error = ABS(Measured Vacuum - Setpoint)


A zero setpoint must be handled separately to prevent division by zero.

Floating-point registers (`F8`) are recommended when fractional values are required.

### 3. Pump Output Adjustment

The system uses three correction levels based on the absolute error.

For a setpoint of 15, the proposed error thresholds are:

| Absolute Error | Adjustment |
|---|---|
| Below 0.6 | No adjustment |
| 0.6 to less than 2.25 | 20 units |
| 2.25 to less than 4.5 | 60 units |
| 4.5 or greater | 100 units |

These thresholds correspond to 4%, 15%, and 30% of a setpoint of 15.

The correction direction depends on the measured vacuum:

- **Measured > Setpoint:** Increase the pump output.
- **Measured < Setpoint:** Decrease the pump output.
- **Measured = Setpoint:** Keep the pump output unchanged.

### 4. Timer Operation

The control logic executes every 10 seconds using `T4:0`.

At the end of each interval:

1. Reset the temporary adjustment value (`N7:10`).
2. Calculate the absolute error.
3. Determine the correction level.
4. Execute the pump adjustment subroutine.
5. Reset the timer for the next interval.

## 5. Program Structure

### LAD 2 — MAIN

Main program execution and subroutine calls.

### LAD 3 — PRC_LOG

Handles the control logic, error evaluation, adjustment selection, and timer operation.

### LAD 4 — OPERATIONS

Handles pump output adjustments using `ADD` and `SUB` instructions.

A single `JSR` instruction calls the operations subroutine after the adjustment value has been selected.

