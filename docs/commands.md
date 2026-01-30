# Serial Command Reference

The OpenTherm Gateway communicates via serial interface at 9600 baud, 8 bits, no parity.

## Message Format

### Incoming Messages (from OTGW)

Messages are reported as a letter followed by 8 hexadecimal digits:

| Prefix | Description |
|--------|-------------|
| `T` | Message received from thermostat |
| `B` | Message received from boiler |
| `R` | Modified request sent to boiler |
| `A` | Modified answer sent to thermostat |

Example: `T10000000` - Thermostat sent a read request for Data ID 0 (Status)

### Commands (to OTGW)

Commands consist of two letters, an equals sign, and an argument, terminated by carriage return (0x0D).

Format: `XX=value<CR>`

---

## Temperature Commands

### OT - Outside Temperature
```
OT=+15.4
```
Specify the outside temperature to inject into OpenTherm messages.

### TT - Temporary Temperature Setpoint
```
TT=18.5
```
Temporarily change the room temperature setpoint. Use `TT=0` or `TT=+0.0` to return to normal programming.

### TC - Constant Temperature Setpoint
```
TC=+16.0
```
Continuously override the room temperature setpoint. Use `TC=+0.0` to cancel.

### SB - Setback Temperature
```
SB=15.5
```
Configure the setback temperature for use with GPIO HOME/AWAY functions.

### CS - Control Setpoint Override
```
CS=10.0
```
Override the control setpoint (CH water temperature) from the thermostat.

### SH - Max CH Setpoint
```
SH=72.0
```
Set the maximum central heating water temperature setpoint.

### SW - DHW Setpoint
```
SW=60.0
```
Set the domestic hot water temperature setpoint.

---

## Control Commands

### CH - CH Enable Override
```
CH=1
```
Override the CH enable bit when CS is non-zero.

### HW - DHW Control
```
HW=1   # Enable
HW=0   # Disable
HW=P   # Start manual DHW push
HW=A   # Thermostat control
HW=T   # Thermostat control
```
Control domestic hot water.

### VS - Ventilation Setpoint
```
VS=25
```
Override the ventilation setpoint from the thermostat.

### MM - Max Modulation
```
MM=40
```
Override the maximum relative modulation from the thermostat.

### CL - Cooling Control
```
CL=62.5
```
Override the cooling control signal from the thermostat.

### CE - Cooling Enable
```
CE=1
```
Override the cooling enable bit when CL is non-zero.

---

## Clock Commands

### SC - Set Clock
```
SC=14:45/6
```
Set the time and day of week (1=Monday, 7=Sunday).

---

## Configuration Commands

### VR - Reference Voltage
```
VR=5
```
Adjust the reference voltage between 0.8V and 2.0V.

### GW - Gateway Mode
```
GW=0   # Monitor mode
GW=1   # Gateway mode
GW=R   # Reset
```
Switch between monitor and gateway modes.

### IT - Ignore Transitions
```
IT=1
```
Control whether multiple mid-bit line transitions are ignored (normally causes Error 01).

### OH - Override High Byte
```
OH=0
```
Copy low byte of MsgID 100 to high byte (workaround for Honeywell Vision thermostats).

### FT - Force Thermostat
```
FT=C   # Remeha Celcia 20
FT=I   # iSense
FT=D   # Default/auto-detect
```
Force thermostat model detection.

---

## LED Configuration

### LA-LF - LED Functions
```
LA=F   # LED A = Flame (default)
LB=X   # LED B = Transmitting (default)
LC=O   # LED C = Setpoint Override (default)
LD=M   # LED D = Maintenance (default)
LE=P   # LED E = Raised Power (default)
LF=C   # LED F = Comfort Mode (default)
```

**Function Codes:**

| Code | Function |
|------|----------|
| R | Receiving OpenTherm message |
| X | Transmitting OpenTherm message |
| T | Activity on master interface |
| B | Activity on slave interface |
| O | Remote setpoint override active |
| F | Flame on |
| H | Flame on for central heating |
| W | Flame on for hot water |
| C | Comfort mode (DHW enable) |
| E | Communication error detected |
| M | Boiler requires maintenance |
| P | Raised power mode |

---

## GPIO Configuration

### GA, GB - GPIO Functions
```
GA=3   # GPIO A = LEDE output (default: no function)
GB=4   # GPIO B = LEDF output (default: no function)
```

**GPIO Functions:**

| Code | I/O | Function |
|------|-----|----------|
| 0 | Input | No function |
| 1 | Output | Constant GND |
| 2 | Output | Constant VCC |
| 3 | Output | LED E function |
| 4 | Output | LED F function |
| 5 | Input | HOME (High=cancel override, Low=setback) |
| 6 | Input | AWAY (High=setback, Low=cancel override) |
| 7 | I/O | DS18x20 temperature sensor |

### TS - Temperature Sensor Function
```
TS=O   # Outside temperature
TS=R   # Return water temperature
```

---

## Data ID Commands

### AA - Add Alternative
```
AA=42
```
Add alternative message to send instead of unsupported messages.

### DA - Delete Alternative
```
DA=42
```
Delete a single occurrence from the alternative message list.

### UI - Unsupported ID
```
UI=9
```
Mark a Data ID as unsupported by the boiler (gateway will inject its own message).

### KI - Known ID
```
KI=9
```
Remove a Data ID from the unsupported list.

### SR - Set Response
```
SR=18:1,230
```
Set the response for a Data ID (one or two data bytes).

### CR - Clear Response
```
CR=18
```
Clear the configured response for a Data ID.

---

## Remote Override Modes

### MW - DHW Override Mode
```
MW=3
```
Set Remote Override Operating Mode for DHW.

### MH - CH1 Override Mode
```
MH=4
```
Set Remote Override Operating Mode for CH1.

### M2 - CH2 Override Mode
```
M2=5
```
Set Remote Override Operating Mode for CH2.

---

## Remote Request

### RR - Remote Request
```
RR=2
```
Issue a remote request to the boiler.

---

## Counter Reset

### RS - Reset Counter
```
RS=HBS   # CH burner starts
RS=HBH   # CH burner operation hours
RS=HPS   # CH pump starts
RS=HPH   # CH pump operation hours
RS=WBS   # DHW burner starts
RS=WBH   # DHW burner operation hours
RS=WPS   # DHW pump starts
RS=WPH   # DHW pump operation hours
```

---

## Print/Query Commands

### PS - Print Summary
```
PS=0   # Print every message
PS=1   # Print summary only
```

### PR - Print Information
```
PR=A   # About (welcome message)
PR=B   # Build date and time
PR=C   # Clock speed
PR=G   # GPIO configuration
PR=I   # GPIO input states
PR=L   # LED functions (e.g., "FXOMPC")
PR=M   # Gateway mode
PR=O   # Setpoint override
PR=P   # Smart-Power mode
PR=R   # Remeha detection state
PR=S   # Setback temperature
PR=T   # Tweak settings
PR=V   # Reference voltage
PR=W   # DHW setting
```

---

## Debug Commands

### DP - Debug Pointer
```
DP=115
```
Set debug pointer to specified file register (development use).
