# Project 9 – Mode/Function Cycling with DDE Data Logging to Excel

A continuous process written in RSLogix 500 and run on RSLogix Emulate 500. The machine cycles between two modes, each running two functions. The current mode and function names are stored as strings, every cycle is counted, and all values can be pulled into Excel on demand through RSLinx DDE.

## Process Description

The process runs continuously. Each function lasts **10 seconds**, so one full cycle takes **40 seconds**:

| Step | Time window | Mode | Function | Counter |
|------|-------------|------|----------|---------|
| 1 | 0–10 s | Mode-1 | F1 | N7:0 |
| 2 | 10–20 s | Mode-1 | F2 | N7:1 |
| 3 | 20–30 s | Mode-2 | F1 | N7:2 |
| 4 | 30–40 s | Mode-2 | F2 | N7:3 |

After step 4 the cycle restarts. Each counter increases by 1 when its step begins, so the counters read (0,0,0,0) → (1,0,0,0) → (1,1,0,0) → (1,1,1,0) → (1,1,1,1) → (2,1,1,1) and so on.

## Memory Map

| Address | Description |
|---------|-------------|
| `ST9:0` | Current mode |
| `ST9:1` | Current function |
| `ST9:2` | Mode + Function (concatenated) |
| `ST10:0`–`ST10:3` | String constants: Mode-1, Mode-2, F1, F2 |
| `N7:0` | M1F1 cycles |
| `N7:1` | M1F2 cycles |
| `N7:2` | M2F1 cycles |
| `N7:3` | M2F2 cycles |
| `T4:0` | 1 s timer (time base 1.0, preset 1) |
| `C5:0` | Cycle counter, preset 40 (seconds in one full cycle) |
| `B3:0/0`–`B3:0/3` | One-shot (ONS) bits, one per step |


## Setup

### 1. Run the program on the emulator

1. Start RSLinx Classic and add the **RSLogix Emulator 500** driver (for example `EMU500-1`).
2. Start Emulate 500 and keep it open.
3. In RSLogix 500 open the project, go to **Comms → System Comms**, select the emulator station, and **Download**.
4. **Go Online** and switch to **Run**.
5. Verify in a Data Monitor that `ST9:0`–`ST9:2` change every 10 seconds and the counters `N7:0`–`N7:3` increase as described above.

### 2. Configure the DDE topic in RSLinx

1. **DDE/OPC → Topic Configuration → New**, name it `PROJ9`.
2. **Data Source** tab: select the emulator station that contains the downloaded program (not an `UNTITLED` one), then **Apply**.
3. **Data Collection** tab: Processor Type = **SLC-500** (MicroLogix is listed under SLC-500), Polled Messages enabled, then **Apply**.
4. Click **Done** and save when prompted.

### 3. Prepare Excel

1. **File → Options → Advanced → General:** make sure *Ignore other applications that use Dynamic Data Exchange (DDE)* is **unchecked**.
2. If DDE is blocked (`#BLOCKED!` or run-time error 13 on `DDEInitiate`), allow it in **Trust Center → External Content**, and in the registry under `HKEY_CURRENT_USER\Software\Microsoft\Office\16.0\Excel\Security\External Content` set `DisableDDEServerLookup` and `DisableDDEServerLaunch` to `0`. Restart Excel afterwards.
3. Enable the **Developer** tab (**File → Options → Customize Ribbon**).
4. Lay out the sheet:

| Cell | Content |
|------|---------|
| A1:A3 | Mode, Function, Mode+Function |
| B1:B3 | Values written by the macro |
| A5:A8 | M1F1, M1F2, M2F1, M2F2 |
| B5:B8 | Values written by the macro |

5. Press **Alt+F11 → Insert → Module** and paste the macro below.
6. **Developer → Insert → Button (Form Control)**, draw it on the sheet, and assign the macro `UpdatePLC`.
7. Save the file as `.xlsm`.

### Excel Macro

```vba
Sub UpdatePLC()
    Dim ch As Long
    Dim v As Variant
    Dim i As Integer
    Dim s As String

    ch = DDEInitiate("RSLinx", "PROJ9")

    For i = 0 To 2
        v = DDERequest(ch, "ST9:" & i)
        If IsArray(v) Then s = CStr(v(1)) Else s = CStr(v)
        Cells(1 + i, 2).Value = Trim(Replace(Replace(s, vbCr, ""), vbLf, ""))
    Next i

    For i = 0 To 3
        v = DDERequest(ch, "N7:" & i)
        If IsArray(v) Then s = CStr(v(1)) Else s = CStr(v)
        Cells(5 + i, 2).Value = Val(s)
    Next i

    DDETerminate ch
End Sub
```

## Testing

1. Run the program on the emulator. Modes and functions should change every 10 seconds, and `ST9:0`–`ST9:2` should follow.
2. Reset `N7:0`–`N7:3` to 0 and confirm they count (1,0,0,0), (1,1,0,0), (1,1,1,0), (1,1,1,1), (2,1,1,1)…
3. Open the Excel workbook and press the button. The sheet should show the current strings and all four counters.
4. Wait 10 seconds and press the button again. The strings should have changed and one counter should have increased by 1.

## Troubleshooting

| Problem | Likely cause / fix |
|---------|--------------------|
| `#BLOCKED!` in a DDE formula | Excel blocks DDE. See the Trust Center and registry steps above. |
| Run-time error 13 on `DDEInitiate` | Excel cannot reach RSLinx. Check DDE settings, make sure RSLinx is running as a normal application (not only as a service), and that Excel and RSLinx run with the same privileges. |
| "Cannot run RSLINX.EXE" | Excel does not find a running DDE server. Start RSLinx manually first, without "Run as administrator". |
| Values are 0 or empty | The topic points to an `UNTITLED` station, or the emulator is not in Run. |
| Topic name error | The topic name in the macro must match the RSLinx topic exactly (`PROJ9`). |
| All counters increase at the same time | The window conditions overlap or the ONS bits are shared. Use distinct ONS bits and `GEQ`/`LES` windows. |
| Counters show large numbers | The program has been running for a long time. Reset `N7:0`–`N7:3` before testing. |
