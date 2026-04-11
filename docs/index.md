---
title: Welcome to Christo's Datasheet
tags:
- hmi
- esp32
- egr314
---
<center>
<font size= "6">Christo Jomon Joseph's Datasheet</font><br>
as part of<br>
<font size= "8"> EGR 314 </font><br>
for<br>
<font size= "5"> Team 305 </font><br>

**Spring 2026**
</center>

## Introduction

My name is Christo Jomon Joseph and this is my individual datasheet for EGR 314 at Arizona State University. I'm part of Team 305, and my role on the team is designing and building the **HMI (Human Machine Interface) & System Safety subsystem** for our rover project.

For more details on the full team project, check out the [team report](https://egr314-s-2026-30.github.io/EGR314-S-2026-305.github.io/).

### Project Overview

Our team is building an autonomous rover that uses a daisy-chained UART communication network to connect multiple subsystems together. Each team member owns one subsystem board. My board sits at the user-facing end of the system: it's what the operator interacts with to monitor the rover's status, issue commands, and trigger an emergency stop if something goes wrong.

### My Subsystem

I'm responsible for the HMI & System Safety Board, which is the primary local interface between the operator and the rover. Here's what it does:

- **Microcontroller:** ESP32-S3-WROOM-1 running MicroPython v1.23.0
- **Display:** 0.96" SSD1306 OLED over I2C, showing live telemetry from other subsystems
- **Input:** 7 tactile pushbuttons for menu navigation and control
- **Safety:** Dedicated emergency stop button with hardware fail-safe behavior
- **Status LEDs:** 4 LEDs for quick visual system state feedback
- **Power:** 9V input regulated to 3.3V via LM2596S-3.3 switching regulator
- **Communication:** Integrated into the team's daisy-chain UART bus : forwards messages it doesn't own, processes and ACKs the ones it does

The full parts list is in the [BOM](04-BOM/BOM.md) section. Schematic, component selection, and power budget are each documented in their own sections of this datasheet.

## Subsystem API

The [Subsystem API](07-API/api.md) page documents all UART messages my HMI board sends and receives on the team daisy-chain network, including message types, byte layouts, data ranges, ACK behavior, and receiver routing logic.
