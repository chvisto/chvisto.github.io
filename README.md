# EGR 314 — Christo Jomon Joseph's Individual Datasheet

This is my individual datasheet for EGR 314 at Arizona State University, Spring 2026. I'm part of Team 305 and my subsystem is the HMI & System Safety Board for our autonomous rover project.

The board runs on an ESP32-S3-WROOM-1, drives a 0.96" SSD1306 OLED over I2C, reads input from 10 tactile buttons, drives 4 status LEDs, and handles daisy-chain UART communication with the rest of the team's subsystems. Power comes in at 9V and gets regulated down to 3.3V by a LM2596S-3.3 switching regulator.

The site covers requirements, block diagram, component selection, BOM, schematic, PCB, and the full subsystem API.

Live site: [chvisto.github.io](https://chvisto.github.io)
