# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RC HeadTracker is a head tracking system for FPV (First-Person View) RC enthusiasts. The project consists of firmware for various microcontroller boards, a Qt-based GUI configuration tool, and a web-based configurator. The system allows users to control their camera view by moving their head.

## Architecture

The project has three main components:

1. **Firmware** (`firmware/`) - Zephyr RTOS-based C++ firmware supporting multiple hardware platforms:
   - NRF52-based boards (Arduino Nano 33 BLE, XIAO BLE)
   - ESP32C3 and ESP32-based boards (M5Stick C Plus)
   - Raspberry Pi Pico (RP2040)
   - Custom DTQ Systems HT board

2. **GUI Configuration Tool** (`gui/`) - Qt5/6 desktop application for configuring and calibrating the head tracker

3. **Web Configurator** (`web_configurator/`) - Web-based configuration interface using JavaScript and Web Bluetooth

## Build System

The firmware uses the Zephyr RTOS build system based on CMake and the West tool.

### Prerequisites
- Zephyr SDK (v0.17.0) and Zephyr RTOS (v3.7.1)
- West tool and Python virtual environment
- Docker container available with complete build environment

### Building Firmware

From the `firmware/src/` directory:

```bash
# Build for specific board
west build -p -b arduino_nano_33_ble

# Build all supported boards
./build_all.sh
```

Supported boards:
- `arduino_nano_33_ble` (including Rev2 variant)
- `xiao_ble/nrf52840`
- `xiao_ble/nrf52840/sense`
- `esp32c3_devkitm`
- `m5stickc_plus/esp32/procpu`
- `rpi_pico/rp2040/w`
- `dtqsys_ht`

### Building GUI

The GUI uses Qt5/6 and can be built with qmake:

```bash
cd gui/src/
qmake HeadTracker.pro
make
```

### Settings System

The project uses a centralized settings system defined in `settings/settings.csv`. Python scripts in the `settings/` directory generate header files and configuration data from this CSV:

- `buildsettings.py` - Main settings generator
- `buildfwsettings.py` - Firmware-specific settings
- `buildguisettings.py` - GUI settings
- `buildwebblebtsettings.py` - Web Bluetooth settings

After modifying `settings.csv`, run the appropriate Python script to regenerate header files.

## Key Firmware Components

### Sensor Libraries
Located in `firmware/src/src/` with support for multiple IMU sensors:
- `BMI270/` - Bosch BMI270 IMU
- `LSM6DS3/` - STMicroelectronics LSM6DS3
- `LSM9DS1/` - STMicroelectronics LSM9DS1
- `MPU6xxx/` - InvenSense MPU6000/MPU6500 series
- `MPU6886/` - InvenSense MPU6886
- `BMM150/` - Bosch BMM150 magnetometer
- `APDS9960/` - Avago APDS9960 gesture sensor

### Communication Protocols
- `CRSF/` - Crossfire Serial Protocol implementation
- `PPMIn.cpp` / `PPMOut.cpp` - Standard PPM signal handling
- Bluetooth Low Energy (BLE) support

### Core System
- `main.cpp` - Entry point (calls C++ start() function)
- `htmain.cpp` - Main application logic
- `io.cpp` - Input/output handling
- `MadgwickAHRS/` - Attitude and heading reference system

## Board Configuration

Each supported board has a device tree overlay file:
- `arduino_nano_33_ble.overlay`
- `esp32c3_devkitm.overlay`
- `xiao_ble_nrf52840.overlay`
- etc.

Configuration files are in `firmware/src/zephyr/`:
- `zephyr/nrf_prj.conf` - NRF52 configuration
- `zephyr/esp32c3_prj.conf` - ESP32C3 configuration
- `zephyr/m5stickc_plus_prj.conf` - M5Stick configuration
- `zephyr/rpi_pico.conf` - Raspberry Pi Pico configuration

## Development Environment

### Docker Setup
The project includes a Dockerfile with a complete Zephyr development environment:

```bash
docker build -t headtracker-build .
docker run -it -v $(pwd):/workspace headtracker-build
```

### VS Code
The project includes VS Code configuration in `.devcontainer/` and `firmware/src/.vscode/`.

## Testing

The build system includes automated builds for all supported platforms via GitHub Actions (`.github/workflows/`).

## Common Development Tasks

1. **Add new sensor support**: Create sensor driver in appropriate subdirectory, update `htmain.cpp` integration
2. **Add new board support**: Create device tree overlay, add board to `CMakeLists.txt`, create configuration file
3. **Modify settings**: Edit `settings/settings.csv`, run relevant Python build scripts
4. **Update firmware**: Modify core logic in `firmware/src/src/`, rebuild with West
5. **Update GUI**: Modify Qt components in `gui/src/`, rebuild with qmake

## Documentation

Additional documentation is available at: https://headtracker.gitbook.io