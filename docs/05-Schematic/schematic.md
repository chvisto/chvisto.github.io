---
title: HMI Subsystem Schematic
---

## Overview

The HMI subsystem schematic is built around the **ESP32-S3-WROOM-1**. It runs the display, reads button inputs, drives the status LEDs, and handles daisy-chain UART communication with the rest of the rover. Power enters through a 9V DC barrel jack and the LM2596S-3.3 switching regulator steps it down to 3.3V for the ESP32 and all peripherals. There's a USB Micro-B port for programming and debug. The 0.96" OLED connects over I2C (just SDA and SCL), and the four status LEDs are driven directly from GPIO through current-limiting resistors. Fuses on both the input rail and USB line protect against overcurrent faults.

![HMI Subsystem Schematic](Screenshot.png)

**Figure 1:** HMI subsystem schematic

## Resources

The schematic as a PDF download is available [*here*](Hmi-subsystem.pdf), and the KiCad project zip is available [*here*](Hmi-subsystem.zip).
