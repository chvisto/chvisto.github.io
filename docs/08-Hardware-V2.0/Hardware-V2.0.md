---
Title : Hardware V2.0
---

# Alterations for Version 2.0

No project, no matter how long you spend on it, is ever perfect. There is always room for improvement. For my Version 2.0, I would improve the project so it moves closer to an industry level standard.

I would focus on these areas:  
- Schematic  
- PCB design  
- Software  

---

## Schematic Improvements
Even though the schematic is accurate and functional, I feel like I could make it more organized for Version 2.0. I looked at a lot of project schematics made by engineers in the industry, and I realized that company level schematics are extremely tidy and structured while still being functional. I know I can improve the neatness of my schematic and get it looking closer to the ones used in top companies.

---

## PCB Improvements
For the PCB design, I really wanted to use vias, but I didn’t have enough time to learn them properly and add them into my project. For Version 2.0, I would definitely use vias so the routing is cleaner and more organized.

I would also make the PCB more compact. Right now it’s at the full 100×100 mm limit with a lot of wasted space. For the next version, I want it to be smaller but still able to hold all the components without any issues.

---

## Software Improvements
I’m honestly disappointed that I didn’t know about interrupts when coding for my team’s subsystem. That means we could’ve been losing precise encoder values, which is not ideal at all. The door has to open exactly to 90 degrees. Losing values because of `sprintf` in UART means the door might not fully open or fully close.

Even though we assumed 4 degrees per encoder count for the motor, and it worked for the demo, real-life situations could be totally different. I would hate to get negative customer feedback just because the door didn’t open or close all the way.

For Version 2.0, I would integrate interrupts across the entire subsystem so we always get accurate readings from the rotary encoder, making the product much more reliable.

---

# Conclusion
This has been a really great journey, going from not even knowing what a schematic was to building a full PCB that connected to three other subsystems and actually worked. Version 2.0 will fix the mistakes I made in Version 1.0 and move my team closer to the project we originally imagined.
