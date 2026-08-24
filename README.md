# Opensprinkler for ESP32 and 74HC595 shift register

Opensprinkler for ESP32 adapted to 74HC595 shift register. This is normally a board like this:

![ESP32 and 74HC595 shift register board](docs/images/esp32-74HC595.png)

This firmware targets boards such as the **LCTech / ChinaLCTech ESP32 16-channel relay module** (two cascaded 74HC595 shift registers). The board exposes a dedicated UART programming header and `IO0` / `EN` buttons for serial flashing.

## Hardware setup

### FT232RL USB-to-serial adapter

Use a **3.3 V** USB-to-TTL adapter (FT232RL or equivalent). The ESP32 uses 3.3 V logic; do not connect 5 V to the board’s `3.3V` pin.

#### FT232RL jumper

Most FT232RL breakout boards have a small **3.3 V / 5 V jumper** (sometimes labeled `VCCIO`, `VCC`, or `3V3/5V`):

- Set the jumper to **3.3 V** before connecting the adapter to the ESP32 board.
- This sets both the adapter’s I/O voltage and (on many modules) the voltage on the `VCC` pin.

Using **5 V** on the jumper can damage the ESP32 or cause unreliable uploads.

#### Wiring (FT232RL ↔ ESP32-74HC595 board)

Connect the adapter to the board’s **serial programming header** (or to UART0: `TX` = GPIO1, `RX` = GPIO3, plus `GND` and `3.3V`):

| FT232RL | ESP32 board |
|---------|-------------|
| **GND** | **GND** |
| **TX**  | **RX** |
| **RX**  | **TX** |
| **VCC** | **3.3V** |

Cross **TX** and **RX** as shown above (adapter TX goes to board RX, adapter RX goes to board TX).

**Power notes:**

- For flashing only, you can power the board from the FT232RL **3.3 V** pin (jumper set to 3.3 V). Current is limited; if uploads fail or the board resets, power the relays board from its **5 V DC input** and keep **GND** common between the adapter and the board (connect only GND, TX, and RX from the FT232RL).
- Relay coils still need the board’s **5 V** supply when testing outputs.

#### Enter download (bootloader) mode

The board does not auto-reset into bootloader mode over serial. Before uploading firmware:

1. Connect the FT232RL as above and plug it into the PC.
2. **Hold** the **IO0** button (GPIO0).
3. While holding **IO0**, press and release the **EN** (reset) button once.
4. Release **IO0**.
5. Start the firmware upload immediately.

After a successful flash, press **EN** once (or power-cycle the board) to run the new firmware.

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

If upload fails with “Failed to connect” or “Timed out”, check the FT232RL **3.3 V jumper**, TX/RX wiring, common **GND**, and that the board is in download mode (**IO0** held during reset).

### After flashing

1. Reset or power-cycle the board.
2. Connect to the OpenSprinkler web UI (WiFi setup on first boot) or use the serial monitor at **115200** baud for debug output.
3. Later firmware updates can also be done **over the air (OTA)** from the web interface once the device is on your network.

## Configuration

Pin mappings for the 74HC595 relay board (shift register data/clock/latch/OE, 16 stations) are defined in **`esp32.h`**. See **`README.txt`** for additional hardware notes and limitations.

### View Full Documentation

[![MkDocs](https://img.shields.io/badge/docs%20by-MkDocs-1f77b4?style=for-the-badge&logo=readthedocs)](https://opensprinkler.github.io/OpenSprinkler-Firmware/)
