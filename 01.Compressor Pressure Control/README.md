# Compressor Pressure Control — RSLogix 500

A PLC ladder logic project for controlling a compressor pump and pressure indicator using two digital pressure switches.

## Operation

- **Below 90 PSI:** The pump runs and the indicator is OFF.
- **At 90 PSI:** The low-pressure switch closes; the pump continues running and the indicator turns ON.
- **At 110 PSI:** The high-pressure switch closes and stops the pump.
- **As pressure drops:** The pump remains OFF until pressure falls below 90 PSI, providing hysteresis.

## I/O Mapping

| Address | Description |
|---|---|
| `I:0/0` | Low-pressure switch (closes at 90 PSI and above) |
| `I:0/1` | High-pressure switch (closes at 110 PSI and above) |
| `O:0/0` | Compressor pump |
| `O:0/1` | Pressure indicator light |

## Software

- RSLogix 500 / RSLogix Micro Starter Lite
- RSLinx Classic and RSLogix Emulate 500 (for simulation)

## Files

- `Compressor_Pressure_Control.RSS` — editable PLC project (add your saved file)
- `screenshots/ladder-logic.png` — ladder logic screenshot (add after final testing)

## Notes

This README describes the original digital-switch assignment. If implementing an analog pressure transmitter with `SCP`, document its actual analog input, scaling range, and updated logic separately.
