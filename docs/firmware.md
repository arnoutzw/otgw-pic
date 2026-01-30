# Firmware Architecture

## Source Files

### gateway.asm

The main OpenTherm Gateway firmware (version 6.6). This is the primary firmware for normal operation.

**Features:**
- OpenTherm message interception and modification
- Serial command interface
- Temperature override capabilities
- GPIO configuration
- LED status indicators
- DS18x20 temperature sensor support
- Self-programming bootloader support

**Operating Modes:**
- **Gateway Mode** (default): Firmware controls message passing, allowing modifications
- **Monitor Mode**: Hardware passes messages directly (fallback on watchdog timeout)

### diagnose.asm

Diagnostic firmware (version 2.1.1) for hardware testing.

**Available Tests:**
1. LED test - Scanning effect across all LEDs
2. Master interface pulse width measurement
3. Slave interface pulse width measurement
4. Loop delay test (requires interfaces connected together)
5. Voltage measurement on OpenTherm interfaces
6. Line idle period measurement

### interface.asm

Simple interface firmware (version 2.0) for basic pass-through operation.

- Reports all received OpenTherm messages via serial
- Transmits messages received on serial as OpenTherm
- Minimal processing overhead

### ds1820.asm

Driver for Dallas/Maxim 1-Wire temperature sensors.

**Supported Sensors:**
- DS18S20 (family code 0x10)
- DS18B20 (family code 0x28)
- DS1822 (family code 0x22)

**Features:**
- CRC verification (configurable)
- Non-blocking operation (state machine with short steps)
- Automatic sensor type detection

### selfprog.asm

Self-programming bootloader based on Microchip AN851.

**Differences from AN851:**
- Fixed 9600 baud (no auto-detection)
- Simplified protocol (dropped first STX, third address byte)
- Transmits ETX and waits 1 second for STX to enter boot mode
- Located in high memory (0x1F00) to avoid interrupt vector conflicts
- Can update itself (no code protection)
- Erase Flash command implemented
- Extended version command reports bootloader memory boundaries

## Memory Layout

The PIC16F1847 has:
- 8K words of program memory (0x0000-0x1FFF)
- 1024 bytes of RAM
- 256 bytes of EEPROM

```
Program Memory:
0x0000          Reset vector
0x0004          Interrupt vector
0x0005-0x1EFF   Main firmware
0x1F00-0x1FFF   Bootloader (selfprog.asm)
```

## Configuration Bits

```
CONFIG1 (0x8007): 0x2FFC
- FOSC: INTOSC (internal oscillator)
- WDTE: WDT enabled, can be disabled by SWDTEN
- PWRTE: Power-up timer disabled
- MCLRE: MCLR/VPP pin is MCLR
- CP: Code protection off
- BOREN: Brown-out reset enabled
- CLKOUTEN: CLKOUT disabled

CONFIG2 (0x8008): 0xEBFF
- WRT: Write protection off
- STVREN: Stack overflow causes reset
- BORV: Brown-out voltage 1.9V
- LPBOR: Low-power brown-out disabled
- LVP: Low-voltage programming enabled
```

## Timing Constants

| Constant | Value | Description |
|----------|-------|-------------|
| BAUD | 25 | SPBRG for 9600 baud at 4 MHz |
| PERIOD | 249 | Timer 2 period for 250μs |
| IDLETIME | 10000 | Line idle time (μs) before considering idle |
| ONESEC | 50 | Timer 4 overflows per second |
| TWOSEC | 122 | Timer 0 overflows in ~2 seconds |

## OpenTherm Protocol

### Message Types

| Code | Type | Direction |
|------|------|-----------|
| 0x00 | Read-Data | Master → Slave |
| 0x10 | Write-Data | Master → Slave |
| 0x20 | Invalid-Data | Master → Slave |
| 0x40 | Read-Ack | Slave → Master |
| 0x50 | Write-Ack | Slave → Master |
| 0x60 | Data-Invalid | Slave → Master |
| 0x70 | Unknown-DataId | Slave → Master |

### Key Data IDs

| ID | Name | Description |
|----|------|-------------|
| 0 | Status | Master/Slave status flags |
| 1 | Control Setpoint | CH water temperature setpoint |
| 9 | Remote Override | Room setpoint override |
| 14 | Max Modulation | Maximum modulation level |
| 16 | Room Setpoint | Desired room temperature |
| 17 | Modulation | Current modulation level |
| 25 | Boiler Temp | Boiler water temperature |
| 27 | Outside Temp | Outside temperature |
| 56 | DHW Setpoint | DHW temperature setpoint |
| 57 | Max CH Setpoint | Max CH water temperature |

## Line Voltage Thresholds

Relative to 2.048V fixed voltage reference:

| Threshold | Value | Actual Voltage |
|-----------|-------|----------------|
| V_SHORT | 6 | ~0.8V (shorted line) |
| V_LOW | 86 | ~11V (logical low) |
| V_OPEN | 156 | ~20V (open line) |
