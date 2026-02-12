---
title: Microcontroller Selection
---

## Microcontroller Selection - ESP32

### What We Need

For the HMI & System Safety Board we need a microcontroller that can handle:
- 1x SPI bus for the TFT LCD display
- 2x UART (one for daisy-chain in, one for daisy-chain out)
- GPIO for 4 pushbuttons (or touchscreen if we use that instead)
- GPIO for E-Stop input
- GPIO for status LEDs
- Power monitoring (ADC input for voltage sensing)

### ESP32-S3-WROOM-1 vs Alternatives

| Microcontroller | Pros | Cons | Price |
|----------------|------|------|-------|
| ESP32-S3-WROOM-1 | WiFi/Bluetooth built in, tons of GPIO, multiple UART and I2C | Bigger footprint, uses more power | ~$5.49 |
| PIC18F47K42 | Cheaper, lower power, good for simple tasks | No WiFi, fewer libraries | ~$2.79 |
| ESP8266 | WiFi, cheap, small | Limited GPIO, older, less powerful | ~$5 |

### Why ESP32

The ESP32-S3 has everything we need and more. It's got multiple UART ports so the daisy-chain setup is easy, hardware SPI for the TFT display, plenty of GPIO for all the buttons and LEDs and has built-in ADC for monitoring the 9V power rail. Plus if we ever want to add WiFi telemetry later it's already there.

### ESP32-S3 Pinout

![ESP32-S3 Pinout](esp32-pinout.png)

### Pin Allocation Table

| Function | ESP32 Pin | Notes |
|----------|-----------|-------|
| UART TX (to next board) | GPIO43 | Daisy-chain output |
| UART RX (from prev board) | GPIO44 | Daisy-chain input |
| SPI MOSI (TFT Data) | GPIO11 | TFT LCD data out |
| SPI MISO (TFT Data) | GPIO13 | TFT LCD data in (for touchscreen) |
| SPI SCK (TFT Clock) | GPIO12 | TFT LCD clock |
| TFT CS (Chip Select) | GPIO10 | TFT LCD select |
| TFT DC (Data/Command) | GPIO9 | TFT LCD data/command |
| Button 1 (Up) | GPIO1 | Menu navigation (if not using touch) |
| Button 2 (Down) | GPIO2 | Menu navigation (if not using touch) |
| Button 3 (Select) | GPIO3 | Menu selection (if not using touch) |
| Button 4 (Back) | GPIO4 | Menu back (if not using touch) |
| E-Stop Input | GPIO5 | Hardware emergency stop |
| Status LED | GPIO21 | System status indicator |
| Voltage Monitor | GPIO6 (ADC) | 9V rail monitoring |
| 3.3V Power | 3V3 | From LM2596 regulator |
| Ground | GND | Common ground |

### Communication with Other Subsystems

Our HMI board talks to the other boards through the daisy-chain UART network:

| Connection | Protocol | What It Does |
|------------|----------|---------|
| Motor Control Board | UART (daisy-chain) | Gets motor status and sends commands |
| Sensor Board | UART (daisy-chain) | Pulls sensor data to show on display |
| Power Board | UART (daisy-chain) | Checks battery and power levels |
| Emergency Stop | GPIO wire | Direct hardware E-Stop to everything |

Messages show up on GPIO44. If they're for us we handle them, if not we just pass them along to GPIO43 for the next board.

**Choice:** ESP32-S3-WROOM-1

> **Rationale:** The ESP32-S3 is perfect for what we're doing right now. The dual UART makes the daisy-chain network easy to handle without any problems. Hardware SPI means the TFT LCD updates fast and smooth without bogging down the CPU. It uses more power than a PIC but we got a 3A regulator so that's not an issue.
