---
title: PCB Design
---

## Overview
This page shows the Printed Circuit Board design for the Rotary Encoder subsystem. This board takes the schematic and lays it out in a physical form that can be manufactured and assembled. It is one of the final steps before soldering and testing the hardware. The board uses two copper layers, top and bottom. The components are placed on the top side, with plated through-holes and pads for each pin. The top layer carries the regulated 5V where needed, while the bottom layer is used as the ground reference for the circuit. Signals between the encoder, microcontroller header, buffer components, and indicators are routed using copper traces that stay isolated from other nets. Once the layout is completed and verified, the design is exported into manufacturing files (Gerber file).

---
## PCB Top and Bottom
![PCB-Design](PCB-top.png)
**Figure 01:** Top Layer of the PCB

![PCB-Design](PCB-bottom.png)
**Figure 02:** Bottom Layer of the PCB


---


## Resouces

The Top and Bottom Layer as a PDF download is available [*here*](PCB-top-bottom-CJJ.pdf), and the updated ECAD project .zip file [*here*](rotary-encoder.zip).

The Gerber Files for the PCB can be downloaded [*here*](Rotary Encoder Gerber Files.zip)