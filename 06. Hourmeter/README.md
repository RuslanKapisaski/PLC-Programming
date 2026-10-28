# PLC Clock (RSLogix 500)

A software clock with seconds, minutes and hours, driven by a single 1-second timer.

## Registers

| Address | Meaning | Range |
|---------|---------|-------|
| `T4:0` | Timer (Base 1.0, Preset 1) | - |
| `N7:0` | Seconds | 0-59 |
| `N7:1` | Minutes | 0-59 |
| `N7:2` | Hours | 0-999 (or 0-23) |
| `N7:3` | Hours x 1000 (optional) | 0+ |



## How It Works

- `T4:0/DN` is true for exactly one scan, then `RES` resets the timer.
- Each stage is triggered by the rollover of the previous one, not directly by the timer.
- `EQU` is true for only one scan because the register is cleared immediately.

