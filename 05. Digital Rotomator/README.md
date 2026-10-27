# 5. Digital Rotameter Flow Calculation 

A ladder logic exercise that calculates oil flow rate (gpm) from a digital rotameter pulse signal.

## Overview

A digital rotameter on an oil pipeline sends one pulse for every **6.3 gallons** that pass through it (its *k-factor*). Instead of a physical input, the rotameter is simulated by a bit address that is pulsed by a timer. The program counts pulses over a fixed 12-second window and converts them into a flow rate, which is updated every 12 seconds.


## How It Works

1. A timer pulses `B3:0/0` every 12 seconds to simulate the rotameter.
2. A counter accumulates the pulses received during the sampling window.
3. When the window ends, the flow rate is calculated and stored in `F8:0`:

   ```
   Flow (gpm) = pulses × k-factor × (60 / window in seconds)
   ```

4. The counter is reset and the next window begins.

