# Simulation Module (chmodule)

The `chmodule` directory contains a gpsim module that simulates OpenTherm thermostat and boiler devices for testing the OTGW firmware.

## Overview

The module provides:
- **Thermostat** - Simulates an OpenTherm master device
- **Boiler** - Simulates an OpenTherm slave device

These allow the OTGW firmware to be tested in gpsim without physical hardware.

## Building

### Prerequisites

- gpsim 0.32.1+ with development headers
- autotools (autoconf, automake, libtool)
- C++ compiler

### Build Steps

```bash
cd chmodule
./autogen.sh
./configure
make -j4
sudo make install
```

### Custom gpsim Location

If gpsim is installed in a non-standard location:

```bash
./configure --with-gpsim=/path/to/gpsim/include/gpsim
```

### Non-Root Installation

```bash
./configure --with-gpsim=$PREFIX/include/gpsim --prefix=$PREFIX
make
make install
```

## Source Files

### opentherm.h / opentherm.cc

Main module implementation providing:

#### Data Types

```cpp
enum OTDataTypes {
    kReadData = 0,      // Master read request
    kWriteData = 1,     // Master write request
    kInvalidData = 2,   // Invalid data
    kReadAck = 4,       // Slave read acknowledgment
    kWriteAck = 5,      // Slave write acknowledgment
    kDataInvalid = 6,   // Data invalid response
    kUnknownDataID = 7  // Unknown data ID response
};
```

#### Status Counters

```cpp
enum StatusCounters {
    kCounterSent,       // Messages sent
    kCounterReceived,   // Messages received
    kCounterStopBit,    // Stop bit errors
    kCounterParity,     // Parity errors
    kCounterCorrupt,    // Data bit errors
    kCounterDirection,  // Wrong direction
    kCounterInvalid,    // Invalid messages
    kCounterMissing,    // Missing response
    kCounterMismatch,   // DataID mismatch
    kCounterUnexpected  // Multiple responses
};
```

#### Classes

**Opentherm** (base class)
- Handles OpenTherm signal transmission and reception
- Manages voltage levels and timing
- Provides message parity calculation

**Thermostat** (derived from Opentherm)
- Simulates an OpenTherm master device
- Supports modes: OpenTherm, On, Off
- Configurable power levels: Low, Medium, High
- Message queue for requests
- Interval-based message transmission

**Boiler** (derived from Opentherm)
- Simulates an OpenTherm slave device
- Configurable responses for each Data ID
- TSP (Transparent Slave Parameters) support
- String data support
- Status tracking (flame, DHW, CH, etc.)

### probe.h / probe.cc

Additional probe/diagnostic functionality for the simulation.

### manager.cc

Module registration and factory functions for gpsim.

## Configuration Files

### Makefile.am

Automake configuration for building the module.

### configure.ac

Autoconf configuration with gpsim detection.

### autogen.sh

Script to regenerate build system files:

```bash
#!/bin/sh
autoreconf -ivf
```

## Usage in Tests

The test suite loads the module and creates simulated devices:

```tcl
# Load the module
module library libchmodule.so

# Create thermostat instance
module load thermostat tstat

# Create boiler instance
module load boiler boiler

# Connect to OTGW
# ... (stimulus configuration)
```

## Thermostat Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| mode | string | "opentherm", "on", or "off" |
| power | string | "low", "medium", or "high" |
| interval | float | Message interval in seconds |
| setpoint | float | Room temperature setpoint |
| roomtemp | float | Current room temperature |
| garbage | bool | Send garbage data |

## Boiler Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| response | int | Set response for Data ID |
| tsp | array | Transparent slave parameters |
| str | string | String data responses |
| garbage | int | Inject garbage responses |

## Voltage Levels

Configurable via `v_low` and `v_high` attributes:

| State | Default Low | Default High |
|-------|-------------|--------------|
| Idle | 15-18V | 15-18V |
| Active | 5-7V | 5-7V |

## Error Injection

Both thermostat and boiler support error injection for testing:
- Parity errors
- Stop bit errors
- Timing violations
- Garbage data
