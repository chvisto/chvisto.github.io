---
title: Component Selection
---

# Component Selection

This page explains the major parts used in my Rotary Encoder Subsystem and the 5 V Voltage Regulation Subsystem.  
Each part includes three options with cost links pros and cons.

---

# 5 V Voltage Regulator (Voltage Regulation Subsystem)

The 5 V regulator powers the PIC18F57Q43 and the rotary encoder.  
It needs to give stable 5 V with low noise so the PIC reads signals clean.

---

## Option 1: LM340MPX-5.0/NOPB
<img src="image1.png" width="180px">

- **$1.43 each**  
- [LM340MPX-5.0/NOPB – Texas Instruments](https://www.digikey.com/en/products/detail/texas-instruments/LM340MPX-5-0-NOPB/367021)

**Pros**  
* Small SMD size  
* Good regulation  
* Well documented  

**Cons**  
* Not good for breadboards  
* Needs copper area for heat  

---

## Option 2: LM7805CT/NOPB
<img src="image2.png" width="180px">

- **$1.80 each**  
- [LM7805CT/NOPB – Texas Instruments](https://www.digikey.com/en/products/detail/texas-instruments/LM7805CT-NOPB/3901929)

**Pros**  
* TO-220 package is easy to work with  
* Stable 5 V output  
* Easy to test  
* Low noise helps encoder readings  

**Cons**  
* Bigger body  
* Can get warm under load  

---

## Option 3: LM7805SX/NOPB
<img src="image3.png" width="180px">

- **$1.80 each**  
- [LM7805SX/NOPB – Texas Instruments](https://www.digikey.com/en/products/detail/texas-instruments/LM7805SX-NOPB/6110585)

**Pros**  
* Small SMD part  
* Good thermal performance  

**Cons**  
* Needs thermal pad  
* Harder to solder  

---

> # Final Choice  
> LM7805CT/NOPB  
>  
> The LM7805CT gives consistent 5 V and works well with the PIC.  
> The TO-220 body makes testing simple and gives space for a heatsink.  
> The 5 V output stays stable even under load which helps encoder accuracy.  
> This is the safest and most reliable choice for the rail.

---

# Rotary Sensor (Rotary Encoder)

The encoder gives angle feedback so the system knows when the door is at 0° and 90°.  
It must send clean signals and last through repeated cycles.

---

## Option 1: Bourns EM14R0D-R20-L064S (FINAL CHOICE)
<img src="image9.png" width="220px">

- **$31.31 each**  
- [EM14R0D-R20-L064S – Bourns 64-Position Encoder](https://www.digikey.com/en/products/detail/bourns-inc/EM14R0D-R20-L064S/2538006)

**Pros**  
* 64 resolution steps  
* Gold plated contacts  
* Long shaft  
* High durability  
* Smooth rotation  
* Clean signals  

**Cons**  
* Higher price  
* Larger size  

---

## Option 2: Bourns PEC11R-4215F-S0024
<img src="image8.png" width="180px">

- **$2.18 each**  
- [PEC11R-4215F-S0024 – Bourns](https://www.digikey.com/en/products/detail/bourns-inc/PEC11R-4215F-S0024/4499665)

**Pros**  
* Common encoder choice  
* 24 detents  
* Clean quadrature  

**Cons**  
* Lower resolution  
* Shorter shaft  
* Not as durable  

---

## Option 3: KY-040 Rotary Encoder Module
<img src="image7.png" width="180px">

- **~$1.50 each**  
- [KY-040 Module](https://www.amazon.com/Cylewet-Encoder-15%C3%9716-5-Arduino-CYT1062/dp/B06XQTHDRR)

**Pros**  
* Easy to test  
* 5 V logic  
* Has small PCB  

**Cons**  
* Noisy signal  
* Low accuracy  
* Not good for final systems  

---

> # Final Choice  
> Bourns EM14R0D-R20-L064S  
>  
> The system needs accurate angle detection and the EM14 gives that with 64 steps.  
> It handles repeated door cycles without wearing out and the long shaft helps with mounting.  
> Signal quality stays clean so the PIC reads angles properly.  
> This makes it the best match for this subsystem.

---

# External Power Supply

This supply powers the whole system.  
It must give stable DC and enough current for all parts.

---

## Option 1: Bel Power SWI12-12-N-P5
<img src="image4.png" width="180px">

- **≈$15.16 each**  
- [SWI12-12-N-P5 – Bel Power Solutions](https://www.digikey.com/en/products/detail/bel-power-solutions/SWI12-12-N-P5/5287239)

**Pros**  
* 12 V output  
* Works from 90 to 264 VAC  
* Level VI efficiency  
* Low idle power  

**Cons**  
* Only 1 A  
* US plug only  

---

## Option 2: Mean Well GST25A12-P1J (FINAL CHOICE)
<img src="image5.png" width="180px">

- **≈$13.70 each**  
- [GST25A12-P1J – Mean Well USA Inc.](https://www.digikey.com/en/products/detail/mean-well-usa-inc/GST25A12-P1J/7703648)

**Pros**  
* 12 V  
* Up to 2.08 A  
* Wide input range  
* Built in protections  
* Reliable long term  

**Cons**  
* Needs AC cord  
* Bigger body  

---

## Option 3: Triad WSU090-2000-R
<img src="image6.png" width="180px">

- **≈$12.15 each**  
- [WSU090-2000-R – Triad Magnetics](https://www.digikey.com/en/products/detail/triad-magnetics/WSU090-2000-R/3094952)

**Pros**  
* 9 V at 2 A  
* Good input range  
* Compact  
* Has protections  

**Cons**  
* Less margin for the 5 V regulator  
* Lower efficiency rating  

---

> # Final Choice  
> Mean Well GST25A12-P1J  
>  
> This unit gives 12 V at 2.08 A which is more than enough for my setup.  
> It has protection features and a wide input range so it stays safe.  
> 12 V gives good margin for the regulator and helps keep the system stable.  
> The cost is close to smaller adapters but with better output which fits the project needs.

