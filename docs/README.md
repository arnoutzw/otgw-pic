# OpenTherm Gateway (OTGW) Firmware Documentation

This documentation covers the OpenTherm Gateway PIC16F1847 firmware source code.

## Overview

The OpenTherm Gateway (OTGW) is a device that sits between an OpenTherm thermostat and boiler in a central heating system. It can:

- **Monitor mode**: Pass messages between thermostat and boiler unmodified using hardware functions
- **Gateway mode**: Intercept and modify OpenTherm messages under firmware control
- **Standalone mode**: Generate OpenTherm messages directly when no OpenTherm thermostat is connected

The gateway communicates via a serial interface (9600 baud, 8 bits, no parity) for configuration and monitoring.

## Project Structure

```
otgw-pic/
├── gateway.asm        # Main OTGW firmware (v6.6)
├── diagnose.asm       # Diagnostic firmware (v2.1.1)
├── interface.asm      # Simple interface firmware (v2.0)
├── ds1820.asm         # DS18x20 temperature sensor driver
├── selfprog.asm       # Self-programming bootloader (based on AN851)
├── 16f1847.lkr        # Linker script for PIC16F1847
├── Makefile           # Build configuration
├── build.ctl          # Build control file
├── license.txt        # License information
├── chmodule/          # gpsim simulation module
├── tests/             # Test suite
├── tools/             # Build tools
└── .github/           # CI/CD workflows
```

## Documentation Contents

- [Building the Firmware](building.md) - How to compile the firmware
- [Firmware Architecture](firmware.md) - Source code structure and components
- [Serial Commands](commands.md) - Complete command reference
- [Testing](testing.md) - Running the test suite
- [Simulation Module](chmodule.md) - gpsim module for testing

## Hardware

The firmware targets the **PIC16F1847** microcontroller running at 4 MHz.

### Peripheral Usage

| Peripheral | Function |
|------------|----------|
| Comparator 1 | Thermostat input (requests) |
| Comparator 2 | Boiler input (responses) |
| Timer 0 | Line idle time measurement, error LED timing |
| Timer 1 | Override LED flashing (0.5s period) |
| Timer 2 | 250μs time base for bit timing |
| Timer 4 | 1-second intervals for standalone mode |
| AUSART | Serial communication (9600 baud) |
| ADC | Line voltage measurement |

## Firmware Versions

| Firmware | Version | Description |
|----------|---------|-------------|
| gateway.hex | 6.6 | Full OpenTherm Gateway functionality |
| diagnose.hex | 2.1.1 | Hardware diagnostic tests |
| interface.hex | 2.0 | Simple pass-through interface |

## License

Copyright (c) 2022 - Schelte Bron

This software is provided for non-commercial use. See [license.txt](../license.txt) for full terms.

## Resources

- [OTGW Website](https://otgw.tclcode.com/)
- [gputils](https://gputils.sourceforge.io/) - PIC assembler toolchain
- [gpsim](http://gpsim.sourceforge.net/) - PIC simulator for testing
