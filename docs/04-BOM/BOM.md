---
title: HMI Subsystem Bill of Materials (SMD/Mixed)
---

## Overview

Full bill of materials for the HMI Subsystem board. Covers the ESP32-S3 microcontroller, LM2596S-3.3 switching regulator, passives, connectors, fuses, and 0.96" OLED display. Resistors, capacitors, and test points come from Peralta Labs. Main ICs, diodes, inductors, and connectors are ordered through DigiKey. All DigiKey links have been verified live.

## Bill of Materials

*Table 1: Bill of Materials for the HMI Subsystem*

| **Part Name/Description** | **Qty** | **Unit Cost** | **Total Cost** | **Manufacturer** | **Manufacturer #** | **Vendor Link** | **Package** | **Schematic Reference Designators** |
|:---|:---|:---|:---|:---|:---|:---|:---|:---|
| Buck Switching Regulator, 3.3V Fixed, 3A, TO-263-5 | 1 | $6.97 | $6.97 | Texas Instruments | LM2596S-3.3/NOPB | [DigiKey](https://www.digikey.com/en/products/detail/texas-instruments/LM2596S-3-3-NOPB/363704) | TO-263-5 (D2PAK) SMD | U1 |
| Schottky Rectifier Diode, 5A, 40V, DO-214AB SMD (1N5824 sub) | 1 | $0.42 | $0.42 | Vishay | SSC54-E3/57T | [DigiKey](https://www.digikey.com/en/products/detail/vishay-general-semiconductor-diodes-division/SSC54-E3-57T/1091564) | DO-214AB (SMC) | D1 |
| ESD TVS Diode, 5V Clamp, SOD-923 (x2) | 2 | $0.10 | $0.20 | onsemi | ESD9B5.0ST5G | [DigiKey](https://www.digikey.com/en/products/detail/onsemi/ESD9B5-0ST5G/1646449) | SOD-923 | D6, D7 |
| Schottky Diode, 30V 200mA, SOD-323 | 1 | $0.14 | $0.14 | onsemi | BAT54HT1G | [DigiKey](https://www.digikey.com/en/products/detail/onsemi/BAT54HT1G/920184) | SOD-323 | D8 |
| LED Blue, 0805 SMD, 20mA | 1 | $0.29 | $0.29 | Wurth Elektronik | 150080BS75000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/150080BS75000/4489901) | 0805 | D2 |
| LED Green, 0805 SMD, 20mA | 1 | $0.29 | $0.29 | Wurth Elektronik | 150080VS75000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/150080VS75000/4489913) | 0805 | D3 |
| LED White, 0805 SMD, 20mA | 1 | $0.29 | $0.29 | Wurth Elektronik | 150080WS75000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/150080WS75000/4489929) | 0805 | D4 |
| LED Red, 0805 SMD, 20mA | 1 | $0.29 | $0.29 | Wurth Elektronik | 150080RS75000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/150080RS75000/4489907) | 0805 | D5 |
| Fixed Inductor, 33uH, 3A, Shielded SMD | 1 | $1.01 | $1.01 | Bourns | SRR1260-330M | [DigiKey](https://www.digikey.com/en/products/detail/bourns-inc/SRR1260-330M/2127415) | 12.5x12.5x6mm SMD | L1 |
| Fuse, 1A Fast-Blow, Glass Cartridge, 5x20mm | 1 | $0.53 | $0.53 | Littelfuse | 0217001.MXP | [DigiKey](https://www.digikey.com/en/products/detail/littelfuse-inc/0217001-MXP/777543) | 5x20mm THT* | F1 |
| Fuse Holder, 5x20mm Cartridge, PCB Mount THT | 1 | $0.62 | $0.62 | Wurth Elektronik | 696108003002 | [DigiKey](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/696108003002/7244560) | THT* | F1 holder |
| PTC Resettable Fuse, 500mA Hold, 1A Trip, 6V, 1206 SMD | 1 | $0.56 | $0.56 | Littelfuse | 1206L050YR | [DigiKey](https://www.digikey.com/en/products/detail/littelfuse-inc/1206L050YR/455721) | 1206 SMD | F2 |
| Electrolytic Capacitor, 100uF 16V, Radial Can SMD | 1 | $0.45 | $0.45 | Nichicon | UWX1C101MCL1GB | [DigiKey](https://www.digikey.com/en/products/detail/nichicon/UWX1C101MCL1GB/589846) | Radial Can SMD | C1 |
| Electrolytic Capacitor, 220uF 16V, Radial Can SMD | 1 | $0.54 | $0.54 | Nichicon | UWX1C221MCL1GB | [DigiKey](https://www.digikey.com/en/products/detail/nichicon/UWX1C221MCL1GB/589849) | Radial Can SMD | C2 |
| Ceramic Capacitor, 0.1uF 50V, 0805 SMD (x3) | 3 | $0.10 | $0.30 | Yageo | CC0805KRX7R9BB104 | [DigiKey](https://www.digikey.com/en/products/detail/yageo/CC0805KRX7R9BB104/302874) | 0805 | C3, C4, C5 |
| Resistor, 10k 1% 0.25W, 1206 SMD (x10) | 10 | $0.10 | $1.00 | Yageo | RC1206FR-0710KL | [DigiKey](https://www.digikey.com/en/products/detail/yageo/RC1206FR-0710KL/728483) | 1206 | R1–R7, R10, R11, R16 |
| Resistor, 27 Ohm 1% 0.25W, 1206 SMD (x2) | 2 | $0.10 | $0.20 | Yageo | RC1206FR-0727RL | [DigiKey](https://www.digikey.com/en/products/detail/yageo/RC1206FR-0727RL/728510) | 1206 | R8, R9 |
| Resistor, 470 Ohm 1% 0.25W, 1206 SMD (x2) | 2 | $0.10 | $0.20 | Yageo | RC1206FR-07470RL | [DigiKey](https://www.digikey.com/en/products/detail/yageo/RC1206FR-07470RL/728586) | 1206 | R12, R13 |
| Resistor, 100 Ohm 1% 0.25W, 1206 SMD (x4) | 4 | $0.10 | $0.40 | Yageo | RC1206FR-07100RL | [DigiKey](https://www.digikey.com/en/products/detail/yageo/RC1206FR-07100RL/728439) | 1206 | R14, R15, R17, R18 |
| ESP32-S3-WROOM-1 Module, 4MB Flash, SMD | 1 | $0.00 | $0.00 | Espressif | ESP32-S3-WROOM-1-N4 | [DigiKey](https://www.digikey.com/en/products/detail/espressif-systems/ESP32-S3-WROOM-1-N4/16162639) | SMD Castellated | E1 |
| USB Mini B Receptacle, 5-Pin, SMT Right Angle | 1 | $0.00 | $0.00 | Hirose | UX60SC-MB-5S8 | [DigiKey](https://www.digikey.com/en/products/detail/hirose-electric-co-ltd/UX60SC-MB-5S8/1949202) | SMD | J1 |
| 0.96" I2C OLED Display, 128x64, SSD1306 | 1 | $0.00 | $0.00 | Songhe | B085WCRS7C | [Amazon](https://www.amazon.com/dp/B085WCRS7C) | Module (THT pins) | LCD1 |
| DC Barrel Jack with Switch, 2.5mm ID, PCB Mount | 1 | $0.64 | $0.64 | Same Sky (CUI) | PJ-002BH | [DigiKey](https://www.digikey.com/en/products/detail/cui-devices/PJ-002BH/408447) | THT* | J2 |
| Connector Header, 8-Position, 2.54mm THT (x5) | 5 | $0.00 | $0.00 | Wurth Elektronik | 61300811121 | [DigiKey](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/61300811121/4846839) | THT* | J3, J4, J5, J6, J7 |
| Tactile Push Button Switch, SPST, THT (x10) | 10 | $0.18 | $1.80 | E-Switch | TL1105SPF160Q | [DigiKey](https://www.digikey.com/en/products/detail/e-switch/TL1105SPF160Q/271573) | THT* | SW1–SW10 |
| 2-Pin Jumper, 2.54mm (x3) | 3 | $0.05 | $0.15 | Wurth Elektronik | 61000211121 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/61000211121/4846851) | THT* | JP1, JP2, JP3 |
| Test Point, Small PCB (x30) | 30 | $0.00 | $0.00 | — | — | Peralta Labs | SMD | TP1–TP30 |



## Resouce
The Bill of Material as a PDF download is available [*here*](HMI-Subsystem-BOM.pdf).
