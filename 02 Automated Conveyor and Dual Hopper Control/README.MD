# Automated Conveyor and Dual Hopper Control

## Overview

This project demonstrates an automated conveyor and dual hopper control system developed using **RSLogix 500** and **Allen-Bradley PLC Ladder Logic**.

The system controls a conveyor motor and two product dispensing hoppers (Pecan and Walnut) using proximity sensors, photoelectric sensors, a level switch, timers, and interlocking logic.

The main objective is to automate the filling process while preventing simultaneous hopper operation and ensuring that products are dispensed only when a container is correctly positioned.

## Software and Technologies

- RSLogix 500
- RSLogix Emulate 500
- RSLinx Classic
- Allen-Bradley PLC
- Ladder Logic (LAD)
- PLC Timers (TON)
- XIC / XIO / OTE Instructions
- One-Shot (ONS) Instructions
- Interlocking Logic

## System Components

| Component | Description |
|---|---|
| Conveyor Motor | Moves containers through the filling system |
| Proximity Switch | Detects container presence |
| Level Switch | Detects when the container is full |
| Red Photo Eye | Requests Pecan Hopper operation |
| Blue Photo Eye | Requests Walnut Hopper operation |
| Pecan Hopper | Dispenses pecans |
| Walnut Hopper | Dispenses walnuts |
| Start Delay Timer | Controls the initial motor startup delay |

## PLC Address Mapping

### Inputs

| Address | Description |
|---|---|
| I:0/0 | Proximity Switch |
| I:0/1 | Level Switch |
| I:0/2 | Red Photo Eye |
| I:0/3 | Blue Photo Eye |

### Outputs

| Address | Description |
|---|---|
| O:0/0 | Conveyor Motor |
| O:0/1 | Walnut Hopper |
| O:0/2 | Pecan Hopper |



## Operating Sequence

### 1. System Startup

When the PLC program starts, a TON timer introduces a 2-second startup delay.

Once the timer reaches its preset value, the conveyor motor starts automatically.

Both dispensing hoppers remain de-energized.

### 2. Container Detection

When the proximity switch detects a container:

- The conveyor motor stops.
- Both hoppers remain OFF until a valid dispensing request is received.

### 3. Pecan Hopper Operation

When the proximity switch is active and the red photo eye is triggered:

- The conveyor remains stopped.
- The Pecan Hopper is energized.
- The Walnut Hopper remains de-energized.

### 4. Walnut Hopper Operation

When the proximity switch is active and the blue photo eye is triggered:

- The conveyor remains stopped.
- The Walnut Hopper is energized.
- The Pecan Hopper remains de-energized.

### 5. Hopper Interlocking

Interlocking logic prevents the Pecan and Walnut hoppers from operating simultaneously.

XIO instructions are used to block one hopper while the other is energized.

This ensures controlled product dispensing.

### 6. Container Full Detection

When the level switch is activated:

- Both hoppers are de-energized.
- The conveyor interrupt condition is cleared.
- The conveyor motor restarts to move the filled container forward.

### 7. System Reset Conditions

When the proximity switch, level switch, and photoelectric sensors are inactive:

- Both hoppers remain OFF.
- The conveyor continues running.


## Project Objectives

- Develop practical PLC programming skills.
- Implement automated material handling logic.
- Understand sensor-based control sequences.
- Apply interlocking to prevent conflicting outputs.
- Practice troubleshooting and online monitoring.
- Validate PLC behavior through simulation.

