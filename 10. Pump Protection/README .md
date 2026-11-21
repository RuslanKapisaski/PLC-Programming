# Project 10 — PLC Pump Control System

## Overview

This project implements a pump-control program in **RSLogix 500** using ladder logic. It provides HAND, OFF, and AUTO operating modes, a timed AUTO duty cycle, and fault monitoring for loss of flow and excessive pressure.

The program was developed and tested using a PLC simulation environment. The descriptions below reflect the intended control behavior discussed during development; verify the final ladder file and I/O assignments before commissioning.

## Operating Modes

| Mode     | Intended behavior                                                                              |
| -------- | ---------------------------------------------------------------------------------------------- |
| **OFF**  | Pump is stopped.                                                                               |
| **HAND** | Pump runs on a manual request, provided no latched fault is blocking it.                       |
| **AUTO** | Pump alternates between **30 seconds ON** and **10 seconds OFF**, provided no fault is active. |

The AUTO cycle uses a 30-second timer (`T4:3`) and a 10-second timer (`T4:2`). A separate rest-phase bit may be used to ensure that the full 10-second pause is maintained before the next ON period.

## Fault Monitoring

### Flow Fault

A flow fault occurs when the pump is running but the flow switch does not detect flow for **5 continuous seconds**.

1. The pump starts.
2. If the flow switch is OFF, the flow-fault timer (`T4:1`) starts.
3. If flow returns before 5 seconds, the timer resets.
4. If no flow is detected for 5 seconds, the Flow Fault bit is latched, the pump stops, and the Flow Fault indicator turns on.
5. The fault remains latched until an authorized reset; switching the flow input ON alone does not clear it.

No-flow detection should not trigger a fault while the pump is intentionally stopped, including during the AUTO rest period.

### Pressure Fault

The pressure input is scaled from a raw analog value into engineering units. If pressure remains at or above the configured **30 PSI** threshold for **5 continuous seconds**, the Pressure Fault bit is latched and the pump is stopped.

The pressure-fault timer is `T4:0`. The raw analog value and scaled pressure must use **different registers** to avoid repeatedly scaling an already scaled value.

## Key Addresses

These addresses were used during development; confirm them against the final `.RSS` file.

| Address   | Description                                 |
| --------- | ------------------------------------------- |
| `N7:1`    | Mode selection: `0` OFF, `1` HAND, `2` AUTO |
| `B3:0/9`  | Latched Flow Fault                          |
| `B3:0/10` | Latched Pressure Fault                      |
| `B3:0/11` | Pump command bit                            |
| `B3:0/12` | Flow Alarm Silenced                         |
| `B3:0/13` | Pressure Alarm Silenced                     |
| `O:0/1`   | Flow Fault indicator                        |
| `O:0/2`   | Pressure Fault indicator                    |
| `I:0/2`   | Alarm Silence input                         |
| `T4:0`    | Pressure fault delay — 5 s                  |
| `T4:1`    | Flow fault delay — 5 s                      |
| `T4:2`    | AUTO rest timer — 10 s                      |
| `T4:3`    | AUTO run timer — 30 s                       |

The exact Alarm Reset input, flow-switch address, timer time bases, and analog-scaling registers should be checked in the final ladder program.
