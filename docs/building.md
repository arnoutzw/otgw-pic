# Building the Firmware

## Requirements

### gputils (required)

The firmware is assembled using gputils (GNU PIC utilities). Version 1.5.0 or later is required.

- Download: https://gputils.sourceforge.io/
- Many Linux distributions include gputils in their package manager

```bash
# Debian/Ubuntu
sudo apt install gputils

# Or build from source
wget -qO- https://sourceforge.net/projects/gputils/files/gputils/1.5.0/gputils-1.5.2.tar.bz2 | tar xjv
cd gputils-1.5.2
./configure
make -j4
sudo make install
```

### GNU Make (required)

The Makefile simplifies the build process.

### Tcl Interpreter (required)

A Tcl interpreter (`tclsh` or `tclkit`) is needed to generate the `build.asm` file containing build metadata.

## Building

### Gateway Firmware (default)

```bash
make
# or explicitly:
make gateway.hex
```

This produces:
- `gateway.hex` - Firmware for programming
- `gateway.cod` - Debug symbols for gpsim
- `gateway.map` - Memory map
- `gateway.lst` - Assembly listing

### Diagnostic Firmware

```bash
make diagnose.hex
```

### Interface Firmware

```bash
make interface.hex
```

### All Firmwares

```bash
make gateway.hex diagnose.hex interface.hex
```

## Build Process

1. The `tools/build.tcl` script generates `build.asm` with:
   - Build number (auto-incremented)
   - Timestamp

2. Assembly files are compiled to object files:
   ```
   gateway.asm → gateway.o
   ds1820.asm → ds1820.o
   selfprog.asm → selfprog.o
   ```

3. Object files are linked using the `16f1847.lkr` linker script

## Customization

Create a `Makefile-local.mk` file for local customizations without modifying the main Makefile:

```makefile
# Example: Custom tool paths
USERBIN = $(HOME)/usr/local/bin
GPASM = $(USERBIN)/gpasm
GPLINK = $(USERBIN)/gplink
GPSIM = $(USERBIN)/gpsim
TCLSH = $(USERBIN)/tclsh

# For testing with non-standard library paths
test: export LD_LIBRARY_PATH = $(HOME)/usr/local/lib
```

## Alternative Installation (Non-root)

Install tools in `~/.local` instead of system directories:

```bash
PREFIX=$HOME/.local

# gputils
wget -qO- https://sourceforge.net/projects/gputils/files/gputils/1.5.0/gputils-1.5.2.tar.bz2 | tar xjv
cd gputils-1.5.2
./configure --prefix=$PREFIX
make -j4
make install
cd ..

# Build firmware
PATH=$PREFIX/bin:$PATH make
```

## Cleaning

```bash
make clean
```

Removes: `*.o`, `*.lst`, `*.map`, `*.hex`, `*.cod`, `*~`

## CI/CD

The project includes a GitHub Actions workflow (`.github/workflows/firmware.yaml`) that:

1. Installs gputils and gpsim
2. Builds all three firmware variants
3. Builds the chmodule simulation module
4. Runs the test suite
5. Uploads firmware artifacts
