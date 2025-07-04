---
title: "nRFBell"
author: "Eduardo"
description: "nRF52840 development board with the ???? battery charger, inspired by the nice!nano & Supermini nRF52840"
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
![May 26 schematic](journalimages\May26.png)
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

![June 1 schematic](journalimages\June1.png)

**Total time spent: 3h**

# June 2, 2025
I finally finished with the battery management (I think) but I am unsure about how I want to have the charging current (ISET) and input voltage & VBATREG (ILIM/VSET) configurable on the board as they may take up too much space.

Circling back to my uncertainty on decoupling capacitors and such, Nordic recommends using the exact decoupling capacitors from example schematics they give on the datasheet.

I also made a pinout inspired by the nice!nano and the Liatris controller. From the nice!nano, I took the extra 2 pins but made 1 VBUS so that its possible to charge the battery without the USB-C port.

Onto the LDO, I am now realizing that I need a boost converter or a charge pump if I want to use a LiFePO battery?? I may be approaching this the wrong way.

![June 2 schematic](journalimages\June2.png)

**Total time spent: 7h😭**

# June 10, 2025
Changed out the battery charger for 2nd and 3rd time 🏴‍☠️
I wired the CN3063 battery charger with its beautiful datasheets only to realize it didn't have power-path which is something I wanted.
Then I had to face the BQ24232 and its datasheet. Texas Instruments has to have the worst datasheets ever made. A literal hour was spent on solely the ITERM pin (probably my inexperience but still).

Anyways, I finished placing decoupling capacitors for the nRF52840 and also editted the pinout.

![June 10 schematic](journalimages\June10.png)

**Total time spent: 5h**

# June 14, 2025

The next thing I wanted to add and learn about were diodes, specifically a Schottky diode on the VBUS of the USB port. Because I put the VBUS or now 5V on the pinout, I needed one to prevent whatever happens when VBUS and 5V are powered diodeless. 
I am still unsure on what Schottky diode to choose as I didn't want much voltage from the USB to be lost.

I moved onto the LDO where I chose the MP20051, an 3.3v LDO with a very low dropout. I realized that I should be using VDDH on the nRF52840 with the output from the battery charger so that it would last longer. I was skeptical when I read "In High Voltage mode, the configured output voltage for REG0 must not be greater than REG0 input voltage minus the voltage drop in REG0" without any clarification but then I saw it would just lower VDD/REG0 output so that would work in making the battery get used up more.

![June 14 schematic](journalimages\June14.png)

**Total time spent: 4h**

# June 25, 2024

I very UNFORTUNATELY had to go back and focus on the battery charger which I changed out for the BQ24230. It, of course, had the same datasheet making the switch out easy.

I then circled back on how the nRF52 will be powered. For more clarification, if I powered everything through the 3.3v LDO, every component would no longer be powered whenever the battery goes below the dropout voltage. If I were to configure the nRF52 in high voltage mode and power it through the battery charger, I could extend how much of the battery can be used to power the dev board itself drastically. At that point, the voltage of the nRF52's pins wouldn't matter as the LDO wouldn't be able to power at 3.3v.
![June 25 MS Paint](journalimages\June25MSPaint.png)

Afterwards, I focused on how I would make the battery charger's charging current selectable which most dev boards just use a jumper. However, I was confused on how to achieve this until I realized I can use parallel resistors. After jumping back and forth between a resistor calculator and JLCPCB basic parts, I landed on a 10k and 2.2k resistor. Then I had to switch out the 31.25k on the LDO because that resistor probably doesn't even exist. I remembered that I could use series resistors to add up to 31.25k, thanks physics regents. However, adding up 31.25k would take too many resistors. After an even longer time jumping back and forth between Desmos and JLCPCB basic parts, I chose 100k and 32k resistors. I hope that works.

![June 25 schematic of LDO](journalimages\June25a.png)
This probably isn't gonna work. Back to the drawing board(jlcpcb's laggy website) yaaaaaaay

From here I found out that JLCPCB charges an ENTIRE 3 DOLLARS for each extended part so I was probably going to have to replace a component anyway. I then decided to switch out the LDO for the NCP186AMX330TAG, an LDO a few dimes cheaper. 

On the nRF, I saw that the nRF's main regulator stage, REG0, uses an LDO which I have seen has a higher dropout than the LDO. Fortunately, the nRF has its own DC/DC regulator with much higher efficiency which looked easy to setup in the schematic.

I also saw that the battery charger became out of stock, amazing. Dealing with that tomrorrow. I'll also figure out a chip antenna which I have to use over a PCB trace antenna since I'm placing 5 pins at the end of the board. This may be the most complicated part of the PCB.

![June 25 complete schematic](journalimages\June25b.png)

**Total time spent: 8h**