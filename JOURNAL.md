---
title: "nRFBell"
author: "Eduardo"
description: "nRF52840 development board with the nPM1100 battery charger, inspired by the nice!nano & Supermini nRF52840"
created_at: "2025-05-26"
---

# May 26, 2025
My only pcb projects have been from Hackpad and Pixeldust. With Hackpad, I completely ruled out the use of passive components, however after using decoupling capacitors during Pixeldust, I had found new motivation to make a development board. Not any normal development board, but an nRF52840 dev board that includes its own battery charger like the nice!nano. I didn't want to copy the exact specifications and schematic, so I chose to use the nPM1100 battery charger from Nordic.

### How I want to configure the nRF52840 so far:
- Normal Voltage supply mode (same power to VDD & VDDH)

### How I want to configure the nPM1100 so far:
- Supplying the nRF52840 with buck regulator (VOUTB to VDD & VDDH) 
- 1.8v from buck regulator (VOUTBSET0/1 to GND)
- Termination voltage at 4.2v (VTERMSET to GND)
- USB port detection for VBUS limit (D+ & D- to USB host, ISET to a GPIO)
- Buck regulator automatic mode (MODE to GND)
- Ship mode available (SHPACT in pinout or onboard button)
- Indication LEDs

### I have this so far:
![May 26 schematic](/journalimages/May26.png)
Im unsure, however, of all the capacitors and resistors needed, as well what specific crystals and their capacitors to use for the nRF52840 chip.

**Total time spent: 3h**

# June 1, 2025
The nPM1100 was a terrible choice for this board.
1. Too complicated
2. The voltage of the GPIOs on the nRF52840, which I want at 3.3v, are tied to the voltage of VDD.
3. This makes the nPM1100's bulk regulator that outputs 1.8v-3.0v useless on this board (to my knowledge)
4. I don't want a large inductor on this board which I believe is required for the nPM1100

After looking at [this post](https://www.reddit.com/r/embedded/comments/1c180l2/which_is_your_favorite_battery_charging_and/), I decided to continue with a Texas Instruments battery charger, landing on the BQ25185.
I chose this over a BQ2407x because it supports LiFePO4 batteries.
Now for my next journal I have to finish wiring this and figure out an LDO.

![June 1 schematic](/journalimages/June1.png)

**Total time spent: 3h**
