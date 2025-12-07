---
title: Component Selection
---

# Component Selection

This page explains the major parts used in my Rotary Encoder Subsystem and the 5 V Voltage Regulation Subsystem.  
Each part includes three options with cost links, pros and cons.  
Each rationale also explains why the part supports the project requirements.

---

# 5 V Voltage Regulator (Voltage Regulation Subsystem)

The 5 V regulator powers the PIC18F57Q43 and the rotary encoder.  
It needs to give stable 5 V with low noise so the PIC reads signals clean.  
A stable 5 V rail is required for accurate angle detection and microcontroller reliability.

---

## Option 1: LM340MPX-5.0/NOPB
<img src="image1.png" width="500px">

- **$1.43 each**  
- [LM340MPX-5.0/NOPB – Texas Instruments](https://www.digikey.com/en/products/detail/texas-instruments/LM340MPX-5-0-NOPB/367021)

**Pros**  
* Small SMD size  
* Good regulation  
* Well documented  

**Cons**  
* Not good for breadboards  
* Needs copper area for heat  

**Why this matters**  
A clean 5 V output keeps PIC inputs stable and prevents encoder misreads.  
This supports the requirement for accurate door angle feedback.

---

## Option 2: LM7805CT/NOPB
<img src="image2.png" width="500px">

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

**Why this matters**  
This regulator keeps the 5 V rail stable even with load changes.  
Stable voltage is required so the PIC can measure encoder signals consistently.

---

## Option 3: LM7805SX/NOPB
<img src="image3.png" width="500px">

- **$1.80 each**  
- [LM7805SX/NOPB – Texas Instruments](https://www.digikey.com/en/products/detail/texas-instruments/LM7805SX-NOPB/6110585)

**Pros**  
* Small SMD part  
* Good thermal performance  

**Cons**  
* Needs thermal pad  
* Harder to solder  

**Why this matters**  
Good thermal handling helps keep the 5 V rail stable during continuous operation.  
This supports system reliability over repeated door cycles.

---

> # Final Choice  
> **LM7805CT/NOPB**  
>  
> The LM7805CT gives consistent 5 V and works well with the PIC.  
> The TO-220 body makes testing simple and gives space for a heatsink.  
> The 5 V output stays stable even under load which helps encoder accuracy.  
>  
> **Why this fits the requirements**  
> A stable 5 V rail is required for the PIC, encoder, and communication lines.  
> This regulator meets the reliability and noise control requirements of the subsystem.

---

# Rotary Sensor (Rotary Encoder)

The encoder gives angle feedback so the system knows when the door is at 0° and 90°.  
It must send clean signals and last through repeated cycles.  
Accurate angle detection is a requirement for proper door motion control.

---

## Option 1: Bourns EM14R0D-R20-L064S (FINAL CHOICE)
<img src="image9.png" width="500px">

- **$31.31 each**  
- [EM14R0D-R20-L064S – Bourns 64-Position Encoder](https://www.digikey.com/en/products/detail/bourns-inc/EM14R0D-R20-L064S/2538006)

**Pros**  
* 64 resolution steps  
* Gold plated contacts  
* Long shaft  
* High durability  
* Smooth rotation  
* Clean quadrature signals  

**Cons**  
* Higher price  
* Larger size  

**Why this matters**  
High resolution and clean quadrature output are needed so the PIC can detect angles accurately.  
This meets the requirement for precise door position tracking.

---

## Option 2: Bourns PEC11R-4215F-S0024
<img src="image8.png" width="500px">

- **$2.2 each**  
- [PEC11R-4215F-S0024 – Bourns](https://www.digikey.com/en/products/detail/bourns-inc/PEC11R-4215F-S0024/4499665)

**Pros**  
* Common encoder choice  
* 24 detents  
* Clean quadrature  

**Cons**  
* Lower resolution  
* Shorter shaft  
* Not as durable  

**Why this matters**  
Lower resolution may reduce accuracy when detecting door endpoints.  
This does not fully support the precision needed for 0° and 90° detection.

---

## Option 3: KY-040 Rotary Encoder Module
<img src="image7.png" width="500px">

- **~$1.85 each**  
- [KY-040 Module](https://www.amazon.com/Cylewet-Encoder-15%C3%9716-5-Arduino-CYT1062/dp/B06XQTHDRR)

**Pros**  
* Easy to test  
* 5 V logic  
* Has small PCB  

**Cons**  
* Noisy signal  
* Low accuracy  
* Not good for final systems  

**Why this matters**  
Noisy output leads to inaccurate angle readings.  
This fails the requirement for consistent and reliable door position feedback.

---

> # Final Choice  
> **Bourns EM14R0D-R20-L064S**  
>  
> The system needs accurate angle detection and the EM14 provides clean quadrature transitions with 64 steps.  
> It handles repeated door cycles without wearing out and the long shaft helps with mounting.  
> Signal quality stays stable so the PIC reads angles correctly.  
>  
> **Why this fits the requirements**  
> The project requires reliable angle detection for 0° and 90° stopping.  
> The EM14 provides the accuracy and durability required for long term operation.

---

# External Power Supply

The project needs a 9 V regulated supply to feed the 5 V regulator.  
It must be safe, reliable, and give enough current for all subsystems.  
A stable 9 V input keeps the 5 V rail from dropping during motor movement or PIC activity.

---

## Option 1: Tri-Mag L6R12-090
<img src="image4.png" width="500px">

- **$8.76**  
- [L6R12-090 – Tri-Mag LLC](https://www.digikey.com/en/products/detail/tri-mag-llc/L6R12-090/7682630)  
- Regulated 9 V output, **1.33 A max**

**Pros**  
* Clean 9 V output  
* Widely stocked  
* UL listed  
* Enough current for basic microcontroller systems  

**Cons**  
* Only 1.33 A headroom  
* Lower margin for motors or added subsystems  

**Why this matters**  
A stable 9 V source is required to keep the 5 V rail clean.  
With only 1.33 A, this option may limit current during door movement or peak loads.

---

## Option 2: Triad WSU090-1300
<img src="image5.png" width="500px">

- **$8.86**  
- [WSU090-1300 – Triad Magnetics](https://www.digikey.com/en/products/detail/triad-magnetics/WSU090-1300/3094977)  
- Regulated 9 V output, **1.3 A max**

**Pros**  
* Reliable regulation  
* Similar size to standard wall adapters  
* Good long term performance  

**Cons**  
* Same current limit as Option 1  
* Fewer units in stock  

**Why this matters**  
1.3 A is enough for the PIC and encoder subsystem but gives limited headroom for motor surges.  
This may not fully support stable operation under load.

---

## Option 3: Triad WSU090-2000-R (FINAL CHOICE)
<img src="image6.png" width="500px">

- **$12.15**  
- [WSU090-2000-R – Triad Magnetics](https://www.digikey.com/en/products/detail/triad-magnetics/WSU090-2000-R/3094952)  
- Regulated 9 V output, **2 A max**

**Pros**  
* Highest current output of all options  
* Good regulation and long term stability  
* Safe wall mount design  
* Enough current for PIC, encoder, motor, and future expansion  

**Cons**  
* Slightly higher cost  
* Fixed cable length  

**Why this matters**  
2 A ensures the regulator always gets a stable input voltage.  
This prevents brownouts, voltage dips, and unstable encoder readings during door motion.  
It directly supports the requirement for reliable and safe subsystem operation.

---

> # Final Choice  
> **Triad WSU090-2000-R**  
>  
> This power supply gives the most current headroom, which keeps the 5 V regulator stable and supports reliable subsystem operation even under heavier loads.  
> Its regulated output meets the project requirements for safe, consistent system power.  
>  
> **Why this fits the requirements**  
> The project requires stable power for the motor, PIC, and sensors.  
> This adapter gives enough current and regulation to meet those requirements reliably.
