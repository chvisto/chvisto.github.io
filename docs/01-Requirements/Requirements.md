---
title: Module's Requirements
---

## Module Requirements

The HMI & System Safety Board is the primary local human machine interface for the rover and a safety critical subsystem. It displays real time system status, reads user input, monitors power health, and enforces emergency shutdown and fail safe behavior when the rover needs to reach a safe state. On the communication side, it sits in the rover’s daisy chained UART network and must not disrupt message flow under any operating condition: commands and telemetry need to keep moving between all subsystems regardless of what this board is doing.

### HMI & System Safety Requirements

| Requirement | Description | Measure of Threshold (Minimum) | Target Measure / Stretch | Stretch (Y/N) |
|-------------|-------------|--------------------------------|--------------------------|---------------|
| Standalone PCB | Implemented as a single modular board | Custom standalone PCB | Daisy chain compatible design | No |
| Visual Interface | Displays rover and subsystem status | Monochrome OLED | ≥1.3" high contrast display | No |
| User Input | Manual control and menu navigation | ≥4 tactile pushbuttons | Debounced multi button UI | No |
| Emergency Stop | Immediate rover shutdown capability | Hard wired E Stop input | Hardware latched E Stop with status LED | No |
| Fail Safe Control | Safe state on fault or E Stop | Software based halt command | Hardware + software redundancy | No |
| Power Monitoring | Monitor rover power health | Voltage sensing on 9 V rail | Voltage + current monitoring | Yes |
| Microcontroller | Central control and communication | SMD MCU with UART and I2C | ESP32 or PIC with dual UART | No |
| UART Communication | Daisy chain data handling | UART TX/RX operational | Full command + telemetry handling | No |
| Message Forwarding | Maintain bus integrity | Non addressed messages forwarded | Reliable forwarding under load | No |
| Status Telemetry | Local system reporting | Display local board status | Display telemetry from other subsystems | Yes |
| Alert System | Notify user of faults | Visual alerts on display | Audible buzzer for critical states | Yes |
| Power Input | External power acceptance | 9 V input supported | Reverse polarity protected input | No |
| Regulation | Local voltage regulation | Regulated 3.3 V output | Stable 3.3 V switching regulator ≥500 mA | No |
| Operating Conditions | Indoor lab operation | Room temperature | 0–40 °C operation | No |
| Programming Interface | Firmware loading and debug | ICSP or USB interface | Onboard USB debug | No |
