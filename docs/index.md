---
title: Welcome to Christo's Datasheet
tags:
- tag1
- tag2
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

### Project Summary

Our team is building an autonomous rover that uses a daisy-chained UART communication network to connect multiple subsystems together. Each team member is responsible for one subsystem board. My board sits at the user-facing end of the system : it's what the operator interacts with to monitor the rover's status, issue commands, and trigger an emergency stop if something goes wrong.

For more details on the full team project, check out the [team report](https://egr314-s-2026-30.github.io/EGR314-S-2026-305.github.io/).

### My Contribution

I'm responsible for the HMI & System Safety Board, which is the primary local interface between the operator and the rover. The board runs on an **ESP32-S3-WROOM-1** microcontroller and handles real-time display output on a **0.96" OLED screen**, reads user input from tactile push buttons, drives four status LEDs, and manages emergency stop and fail-safe behavior. Power comes in at 9V and gets regulated down to 3.3V using the **LM2596S-3.3** switching regulator.

On the communication side, the board integrates into the team's daisy-chain UART bus, forwarding messages it doesn't own and acting on the ones it does — including sending and receiving telemetry from the other subsystems.

The full list of parts used on the board is in the [BOM](04-BOM/BOM.md) section, and the schematic, component selection, and power budget are all documented in their respective sections of this datasheet.
