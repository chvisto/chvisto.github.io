---
title: Module Bill of Materials
tags:
- tag1
- tag2
---

## Overview

This is the full bill of materials for the HMI Subsystem board. It covers everything from the main ESP32-S3 microcontroller and the LM2596S-3.3 switching regulator down to the passives, connectors, fuses, and the 0.96" OLED display. Most of the resistors, capacitors, and test points are sourced from Peralta Labs, while the main ICs and connectors are ordered through DigiKey. The table includes part numbers, quantities, vendor links, and schematic reference designators to make it easy to cross-reference with the schematic.

## Bill of Materials

*Table 1: Bill of materials for the HMI Subsystem*

| **Part Name/Description** | **Qty** | **Unit Cost** | **Total Cost** | **Manufacturer** | **Manufacturer #** | **Vendor Link** | **Datasheet Link** | **Schematic Reference Designators** |
|:---|:---|:---|:---|:---|:---|:---|:---|:---|
| ESP32-S3-WROOM-1 Module, SMD, 4MB Flash | 1 | $0.00 | $0.00 | Espressif | ESP32-S3-WROOM-1-N4 | [DigiKey](https://www.digikey.com/en/products/detail/espressif-systems/ESP32-S3-WROOM-1-N4CT/15297745) | n/a | E1 |
| Buck Switching Regulator, 3.3V Fixed, 3A, TO-263-5 | 1 | $0.00 | $0.00 | Texas Instruments | LM2596S-3.3/NOPB | [DigiKey](https://www.digikey.com/en/products/detail/texas-instruments/LM2596S-3-3-NOPB/363704) | n/a | U1 |
| USB Micro B Receptacle, 5-Pin, SMT Right Angle/THT | 1 | $0.00 | $0.00 | Amphenol | UX60SC-MB-5S8 | [DigiKey](https://www.digikey.com/en/products/detail/amphenol-cs-fci/670-2675-1-ND/2370786) | n/a | J1 |
| Connector Header, Through Hole, 8 Position, 2.54mm | 5 | $0.00 | $0.00 | Wurth Elektronik | 61300811121 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/61300811121/4846907) | n/a | J3, J4, J5, J6, J7 |
| DC Barrel Jack with Switch, 2.1mm ID, PCB Mount | 1 | $1.10 | $1.10 | CUI Devices | PJ-002BH | [DigiKey](https://www.digikey.com/en/products/detail/cui-devices/PJ-002BH/408447) | n/a | J2 |
| 0.96" I2C OLED Display Module, 128x64, SSD1306 | 1 | $0.00 | $0.00 | Songhe | B085WCRS7C | [Amazon](https://www.amazon.com/dp/B085WCRS7C) | n/a | LCD1 |
| Schottky Rectifier Diode, 30V 5A, DO-201AD (1N5824 sub) | 1 | $0.62 | $0.62 | Vishay | SB530-E3/54 | [DigiKey](https://www.digikey.com/en/products/detail/vishay-general-semiconductor-diodes-division/SB530-E3-54/420422) | n/a | D1 |
| ESD Protection TVS Diode, 5V, SOD-923 (x2) | 2 | $0.17 | $0.34 | onsemi | ESD9B5.0ST5G | [DigiKey](https://www.digikey.com/en/products/detail/onsemi/ESD9B5-0ST5G/1646449) | n/a | D6, D7 |
| LED Blue, T-1 3/4 (5mm), 20mA | 1 | $0.00 | $0.00 | Wurth Elektronik | 151051BS04000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/151051BS04000/4489940) | n/a | D2 |
| LED Green, T-1 3/4 (5mm), 20mA | 1 | $0.00 | $0.00 | Wurth Elektronik | 151051VS04000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/151051VS04000/4489944) | n/a | D3 |
| LED White, T-1 3/4 (5mm), 20mA | 1 | $0.00 | $0.00 | Wurth Elektronik | 151051WS04000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/151051WS04000/4489952) | n/a | D4 |
| LED Red, T-1 3/4 (5mm), 20mA | 1 | $0.00 | $0.00 | Wurth Elektronik | 151051RS04000 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/151051RS04000/4489942) | n/a | D5 |
| Inductor, 33uH, 2.05A, Radial (Bourns RLB9012) | 1 | $1.25 | $1.25 | Bourns | RLB9012-330KL | [DigiKey](https://www.digikey.com/en/products/detail/bourns-inc/RLB9012-330KL/705790) | n/a | L1 |
| Fuse, 1A Fast-Blow, 5x20mm Cylindrical | 1 | $0.45 | $0.45 | Littelfuse | 0215001.MXP | [DigiKey](https://www.digikey.com/en/products/detail/littelfuse-inc/0215001-MXP/777524) | n/a | F1 |
| Fuse Holder, 5x20mm, PCB Mount | 1 | $0.94 | $0.94 | Keystone | 3544-2 | [DigiKey](https://www.digikey.com/en/products/detail/keystone-electronics/3544-2/316039) | n/a | F1 holder |
| Electrolytic Capacitor, 100uF 16V, Radial | 1 | $0.20 | $0.20 | Panasonic | ECA-1CM101 | [DigiKey](https://www.digikey.com/en/products/detail/panasonic-electronic-components/ECA-1CM101/244997) | n/a | C1 |
| Electrolytic Capacitor, 220uF 16V, Radial | 1 | $0.25 | $0.25 | Panasonic | ECA-1CM221 | [DigiKey](https://www.digikey.com/en/products/detail/panasonic-electronic-components/ECA-1CM221/245002) | n/a | C2 |
| Ceramic Capacitor, 0.1uF 50V, Through Hole (x3) | 3 | $0.00 | $0.00 | Various | Generic | Peralta Labs | n/a | C3, C4, C5 |
| Resistor, 10k, 2.5mW, Through Hole (x7) | 7 | $0.00 | $0.00 | Various | Generic | Peralta Labs | n/a | R1, R2, R3, R4, R5, R6, R7 |
| Resistor, 10k, Through Hole (x3) | 3 | $0.00 | $0.00 | Various | Generic | Peralta Labs | n/a | R10, R11, R16 |
| Resistor, 27 Ohm, Through Hole (x2) | 2 | $0.00 | $0.00 | Various | Generic | Peralta Labs | n/a | R8, R9 |
| Resistor, 470 Ohm, Through Hole (x2) | 2 | $0.00 | $0.00 | Various | Generic | Peralta Labs | n/a | R12, R13 |
| Resistor, 100 Ohm, Through Hole (x4) — LED current limiting | 4 | $0.00 | $0.00 | Various | Generic | Peralta Labs | n/a | R14, R15, R17, R18 |
| Tactile Push Button Switch, SPST, Through Hole (x10) | 10 | $0.18 | $1.80 | E-Switch | TL1105SPF160Q | [DigiKey](https://www.digikey.com/en/products/detail/e-switch/TL1105SPF160Q/271573) | n/a | SW1, SW2, SW3, SW4, SW5, SW6, SW7, SW8, SW9, SW10 |
| 2-Pin Jumper, 2.54mm (x3) | 3 | $0.05 | $0.15 | Wurth Elektronik | 61000211121 | [DigiKey](https://www.digikey.com/en/products/detail/wurth-elektronik/61000211121/4846851) | n/a | JP1, JP2, JP3 |
| Schottky Diode, 30V 200mA, SOD-323F (BAT54J) | 1 | $0.18 | $0.18 | Nexperia | BAT54J,115 | [DigiKey](https://www.digikey.com/en/products/detail/nxp-semiconductors/BAT54J,115/2531305) | [datasheet](https://assets.nexperia.com/documents/data-sheet/BAT54_SER.pdf) | D8 |
| Fuse, 500mA Fast-Blow, 5x20mm Glass Cartridge | 1 | $0.42 | $0.42 | Littelfuse | 0217.500MXP | [DigiKey](https://www.digikey.com/en/products/detail/littelfuse-inc/0217-500MXP/777536) | [datasheet](https://www.littelfuse.com/~/media/electronics/datasheets/fuses/littelfuse_fuse_217_datasheet.pdf.pdf) | F2 |
| Test Point, Small PCB (x30) | 30 | $0.00 | $0.00 | - | - | Peralta Labs | n/a | TP1-TP30 |


## Resouce
The Bill of Material as a PDF download is available [*here*](HMI-Subsystem-BOM.pdf).
