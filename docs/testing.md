# Testing

The OTGW firmware includes a comprehensive test suite that runs on the gpsim PIC simulator.

## Requirements

### gpsim

The test suite requires gpsim version 0.32.1 or later with PIC16F1847 support.

```bash
# Download and build gpsim
wget -qO- https://sourceforge.net/projects/gpsim/files/gpsim/0.32.0/gpsim-0.32.1.tar.gz | tar xzv
cd gpsim-0.32.1
./configure --disable-gui
make -j4
sudo make install
```

### chmodule (Simulation Module)

The test suite requires a simulated central heating system provided by the chmodule.

```bash
cd chmodule
./autogen.sh
./configure
make -j4
sudo make install
```

### System Packages

**RPM-based systems (Fedora, RHEL, CentOS):**
```bash
sudo dnf install tcl subversion make libtool flex bison \
    popt-devel glib2-devel readline-devel
# Optional: gtk2-devel (GUI), lyx (docs)
```

**Debian-based systems (Ubuntu, Debian):**
```bash
sudo apt install tcl subversion make libtool flex bison \
    libpopt-dev libglib2.0-dev libreadline-dev
# Optional: libgtk2.0-dev (GUI), lyx (docs)
```

**Pacman-based systems (Arch Linux):**
```bash
sudo pacman -S tcl subversion gcc make autoconf automake libtool \
    pkgconfig flex bison popt glib2 readline
# Optional: gtk2 (GUI), lyx (docs)
```

## Running Tests

### Full Test Suite

```bash
make test
```

### Selective Tests

Use `TESTFLAGS` to run specific tests:

```bash
# Run only thermostat tests
make test TESTFLAGS="-match 'thermostat-*'"

# Run thermostat and standalone tests
make test TESTFLAGS="-match 'thermostat-* standalone-*'"

# Run a single test
make test TESTFLAGS="-match 'command-1'"
```

## Test Files

Tests are defined in `.stc` files in the `tests/` directory:

| Category | Tests | Description |
|----------|-------|-------------|
| boiler | boiler.stc | Boiler communication |
| clock | clock.stc | Time/date functions |
| command | command-1 through command-7 | Serial commands |
| control | control-1 through control-5 | Control functions |
| dhw | dhw-1 through dhw-10 | Domestic hot water |
| gpio | gpio-1 through gpio-5 | GPIO functions |
| interval | interval-1, interval-2 | Timing intervals |
| monitor | monitor-1, monitor-2 | Monitor mode |
| opermode | opermode-1 | Operating modes |
| override | override-1 through override-5 | Setpoint overrides |
| reset | reset-1 through reset-5 | Reset behavior |
| sensor | sensor-1 through sensor-12 | Temperature sensors |
| serial | serial-1, serial-2, serial-5 | Serial interface |
| service | service-1, service-2 | Service functions |
| setup | setup.stc, setup33.stc | Initial setup |
| smartpower | smartpower.stc | Smart power mode |
| standalone | standalone-1 through standalone-7 | Standalone operation |
| str | str-1 through str-3 | String handling |
| summary | summary-1 | Summary reports |
| thermostat | thermostat-1 through thermostat-7 | Thermostat communication |
| tsp | tsp-1 through tsp-8 | Transparent slave parameters |
| ventilation | ventilation-1 | Ventilation control |
| version | version-1 through version-3 | Version reporting |

## Test Framework

The test suite uses:
- `tests/all.tcl` - Main test runner
- Tcl-based test scripts
- gpsim for PIC simulation
- chmodule for thermostat/boiler simulation

## Non-Root Installation

If you cannot use root permissions:

```bash
PREFIX=$HOME/.local

# Install gpsim
svn checkout svn://svn.code.sf.net/p/gpsim/code/branches/p16f1847 gpsim
cd gpsim
./autogen.sh
./configure --prefix=$PREFIX --disable-gui
make -j4
make install
cd ..

# Install chmodule
cd chmodule
./autogen.sh
./configure --with-gpsim=$PREFIX/include/gpsim --prefix=$PREFIX
make -j4
make install
cd ..

# Run tests
PATH=$PREFIX/bin:$PATH LD_LIBRARY_PATH=$PREFIX/lib make test
```

Or add to `Makefile-local.mk`:

```makefile
GPSIM = $(HOME)/.local/bin/gpsim
test: export LD_LIBRARY_PATH = $(HOME)/.local/lib
```

## Continuous Integration

The GitHub Actions workflow automatically:
1. Installs dependencies
2. Builds the firmware
3. Builds chmodule
4. Runs the complete test suite
