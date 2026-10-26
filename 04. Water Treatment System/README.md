# Project 4 – Multi-Position Valve Control 


## Description

A modular water treatment system is being integrated into an existing one. When the host system calls for a cycle (cycle call), a servo turns a cam that moves a control valve:

`HOME → FILL (10 s) → DRAIN (20 s) → FLUSH (10 s) → HOME`

The valve position is determined by four microswitches operated by the cam.

## I/O

| Address | Description |
|---------|-------------|
| `I:0/0` | Home microswitch (closed when valve is in HOME position) |
| `I:0/1` | Fill microswitch |
| `I:0/2` | Drain microswitch |
| `I:0/3` | Flush microswitch |
| `I:0/4` | Cycle call from the host system |
| `O:0/0` | Servo (energized = cam rotates) |
| `N7:0` | Bonus 1: position (0 = HOME, 1 = FILL, 2 = DRAIN, 3 = FLUSH, 4 = TRAVELLING) |
| `B3:0/0` | Bonus 2: Reset / cycle interrupt |


## How it works

- **Startup / seeking home:** on the first scan RUN latches and the servo starts immediately. CYCLE is off, so the timers do not stop the servo while it seeks home (test 2).
- **Stopping at home:** RUN is only cleared if the cam has been away from home (AWAY) and home is now closed. This way, a cycle call made while the valve is still at home does not stop the servo right away.
- **Dwell times:** at FILL/DRAIN/FLUSH the matching timer holds the servo off only while it is timing (`TT`); after `DN` the servo starts again even if the switch is still closed.
-**Reset (`B3:0/0`):** clears CYCLE - the timers reset and the servo keeps running without dwell times until it reaches home.
- **Power loss:** the first scan latches RUN again and the valve returns to home.




