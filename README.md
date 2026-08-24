# Opensprinkler for ESP32 and 74HC595 shift register

Opensprinkler for ESP32 adapted to 74HC595 shift register. This is normally a board like this:

![ESP32 and 74HC595 shift register board](docs/images/esp32-74HC595.png)

This firmware targets boards such as the **LCTech / ChinaLCTech ESP32 16-channel relay module** (two cascaded 74HC595 shift registers). The board exposes a dedicated UART programming header and `IO0` / `EN` buttons for serial flashing.

## Hardware setup

### FT232RL USB-to-serial adapter

Use a USB-to-TTL adapter (FT232RL or equivalent). The ESP32 UART pins (`GPIO1` / `GPIO3`) use **3.3 V logic** — do not drive them with 5 V.

#### Wiring (FT232RL ↔ ESP32-74HC595 board)

Connect the adapter to the board’s **serial programming header** (UART0: `TX` = GPIO1, `RX` = GPIO3):

| FT232RL | ESP32 board |
|---------|-------------|
| **GND** | **GND** |
| **VCC / 5V** | **VIN / 5V** |
| **TX** | **RX** (GPIO3) |
| **RX** | **TX** (GPIO1) |

Cross **TX** and **RX** as shown above (adapter TX goes to board RX, adapter RX goes to board TX).

![ESP32 74HC595 pinout](docs/images/esp32-74HC595-pinout.jpeg)
![FT232RL pinout](docs/images/FT232RL-pinout.jpeg)

**Power notes:**

- Power the board from the FT232RL **5 V** pin via **VIN / 5V**. If uploads fail or the board resets, use the board’s **5 V DC input** instead and keep only **GND**, **TX**, and **RX** connected from the FT232RL.
- Relay coils still need the board’s **5 V** supply when testing outputs.

#### Enter download (bootloader) mode

The board does not auto-reset into bootloader mode over serial. Before uploading firmware:

1. Connect the FT232RL as above and plug it into the PC.
2. Connect **GPIO0** to **GND** (jumper wire, or hold the **IO0** button if your board has one).
3. Press **EN / RST** once, or power-cycle the board.
4. Start the firmware upload in PlatformIO immediately.

After a successful upload, **disconnect GPIO0 from GND** and press **EN / RST** again (or power-cycle) to run the new firmware.

## Flashing the firmware

### Prerequisites

- [VS Code](https://code.visualstudio.com/) with the [PlatformIO](https://platformio.org/) extension, **or** the [PlatformIO Core CLI](https://docs.platformio.org/en/latest/core/installation.html) (`pio`).
- USB drivers for the FT232RL (usually built in on Linux; on Windows/macOS install FTDI drivers if needed).
- Review **`esp32.h`** and adjust options for your hardware (flash size, LCD type, sensors, etc.). By default **`ESP32_FLASH_4MB`** is enabled, matching the 4 MB flash on these boards.

### Build and upload (VS Code)

1. Clone this repository and open the project folder in VS Code.
2. Wait for PlatformIO to install the ESP32 platform and libraries.
3. Connect the FT232RL and put the board in download mode (see above).
4. Select the **`esp32_sprinkler`** environment (default in `platformio.ini`).
5. Click **PlatformIO: Upload** (arrow icon on the bottom toolbar), or run **PlatformIO: Upload and Monitor**.

### Build and upload (command line)

```bash
# Install PlatformIO Core if needed: https://docs.platformio.org/en/latest/core/installation.html

# Put the board in download mode, then:
pio run -e esp32_sprinkler -t upload

# Optional: specify the serial port (Linux example)
pio run -e esp32_sprinkler -t upload --upload-port /dev/ttyUSB0

# Serial monitor (115200 baud)
pio device monitor -e esp32_sprinkler
```

On Windows, use a port such as `COM3`; on macOS, `/dev/cu.usbserial-*`.

### What happens during upload

- The **`esp32_sprinkler`** environment builds the OpenSprinkler firmware for ESP32 with LittleFS.
- **`esp32_partition_setup.py`** picks the partition table and flash size from `esp32.h`.
- **`merge_firmware.py`** merges bootloader, partition table, and application into a single image and flashes it at address `0x0`.

If upload fails with “Failed to connect” or “Timed out”, check TX/RX wiring, common **GND**, power on **VIN / 5V**, and that **GPIO0** is connected to **GND** during reset.

### After flashing

1. Disconnect **GPIO0** from **GND**, then reset or power-cycle the board.
2. Connect to the OpenSprinkler web UI (WiFi setup on first boot) or use the serial monitor at **115200** baud for debug output.
3. Later firmware updates can also be done **over the air (OTA)** from the web interface once the device is on your network.

## Configuration

Pin mappings for the 74HC595 relay board (shift register data/clock/latch/OE, 16 stations) are defined in **`esp32.h`**. See **`README.txt`** for additional hardware notes and limitations.

### View Full Documentation

[![MkDocs](https://img.shields.io/badge/docs%20by-MkDocs-1f77b4?style=for-the-badge&logo=readthedocs)](https://opensprinkler.github.io/OpenSprinkler-Firmware/)
