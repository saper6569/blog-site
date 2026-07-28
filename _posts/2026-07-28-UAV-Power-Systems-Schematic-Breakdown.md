---
layout: post
title: "Power System Schematic Breakdown"
date: 2026-07-28
tags: [Underwater ROV, Project]
---

# High Level Design
Before going into depth on individual circuits it would likely be beneficial to explain the high level design to provide insight into system design ideologies, tradeoffs and decisions that will influence development. The purpose of this is to illustrate how individual modules will work together.

The primary purpose of the power systems is to control, manage, regulate and measure the flow of electrical energy through the battery to components pipeline. This includes converting voltages into the various supply rails required by different components as well as stable voltage regulation, continuous monitoring of battery state and also providing safety precautions for both hardware and users from electrical faults. 

The following diagram presents the high-level architecture of the power subsystem, illustrating how energy flows from the battery through the protection, monitoring, regulation, and distribution stages before reaching the individual electronic components. This overview serves as a big picture guide for the detailed circuit explanations presented in the following sections.

>**Note: GND connections are omitted to save space. **

![high level ]({{ 'assets/images/2026-07-22 high level.png' | relative_url }})

# Battery Array
As stated previously, the battery array consists of fifteen 3.7 V nominal 18650 lithium-ion cells arranged in a 3-series, 5-parallel (3S5P) configuration. These rechargeable cells were selected for their high energy density, long cycle life, and ability to deliver the continuous current required by the system while maintaining reliability. The 3S configuration provides a maximum fully charged voltage of 12.6 V and a nominal operating voltage of 11.1 V, making it well suited for powering the system as it is designed for 12 V. The five parallel cells increase the overall battery capacity while allowing the battery pack to safely supply well over the required 20 A continuous current with reduced heat generation. 

# Battery Management System (BMS)
As stated in a previous post a pre-made module will be used for the battery management system. This decision was made due to already existing modules being quite accessible and cheap as well as reducing design lead time. The chosen BMS is designed for 3s lithium ion with a maximum discharge current of 40A which lies well within the specifications of the battery pack.

The BMS' main functions are to monitors and protects the battery pack through charging and discharge. This is done through the various preventative and active measures such as over-charge protection, over-discharge protection, short circuit protection and cell balancing.

- **Overcharge Protection:** Stops charging immediately if cell voltage gets too high, preventing overheating.
- **Over-Discharge Protection:** Shuts off power output if the battery drains too low, preventing permanent chemical damage.
- **Temperature Control:** Monitors heat levels and blocks charging during temperature conditions that can ruin the internal components.
- **Short Circuit Protection:** Cuts power in  if an accidental fault causes a dangerous current spike.
- **Cell Balancing:** Equalizes charge levels across all individual cells inside the battery so they wear out evenly and hold maximum capacity.

>**I plan on designing my own bms for later iterations. **

# Bulk Power
The BMS is connected to the bulk power stage. This is where the 12V input is initially handled. The main purpose of this section is to house additional safety precautions and buses to all the voltage rails as well as the battery fuel gauge. Below is the schematic for this stage.

![bulk power ]({{ 'assets/images/2026-07-22 input.png' | relative_url }})

## Safety
This stage provides two safety features for both the electronics as well as the user. The first is a high side fuse to protect against high currents that can damage electronics and lie outside of limits set by trace and via widths. This is a slow blow fuse to allow for current spikes on motor startup but still provide required protection. The second is a kill switch which is mostly for testing and prototyping. The switch acts as a last case resort to shut of power. Redundancy in safety measures ensures the system shuts down during a fault even if one safeguard fails.

## Reverse Polarity Protection
Reverse polarity protection circuits prevent damage to electronics in the case of the power source being accidentally connected backwards. Many electronic components can fail and or be permanently damaged by reverse currents. The implemented reverse polarity protection circuit uses a high side P-channel mosfet paired with a zener diode.

Using a P-channel MOSFET in place of a diode provides a very low resistance path when the correct polarity is applied, but turns completely off when the polarity is reversed. The diode is to protect the MOSFET's gate oxide from excessive gate-to-source voltage. This is done by clamping the Vgs to a safer vale when the mosfet is in reverse bias (no current flow).

## Battery Fuel Gauge
To measure battery capacity a current and voltage sensing ic is used. The INA226 is a high-precision digital current shunt and power monitor that measures bus voltage, current flow, and calculated power. The INA226 takes in supply voltage of 2.7V-5.5V and a bus voltage og 0V-36V, both of which fit the specifications of this board. Data is communicated through I2C, to a programmable address. Below is the schematic.

![fg ]({{ 'assets/images/2026-07-22 fuel gauge.png' | relative_url }})

With a 0.002 Ω shunt resistor (R6), the INA226 can measure a maximum theoretical current of 40.96 A. This limit is calculated by dividing the chip's internal shunt voltage limit of 81.92 mV by the resistor value. This allows the required 20A to be monitored with ample headroom. At 20A the resistor will output 0.8W of heat which will have to be considered when choosing parts.

>**Note: Pins A0 and A1 may change depending on final I2C address **

# Buck Converters
To bring voltage down from 12V to the required 3.3V and 5V rails two lmr51430 DC-DC buck converters are used. Each ic is able to drive 3A continuously and works with the 12V input with low loss. Designs below comes directly from datasheet and are meant for the 500kHz variants.

![buck converter ]({{ 'assets/images/2026-07-22 3.3V.png' | relative_url }})

![high level ]({{ 'assets/images/2026-07-22 5V.png' | relative_url }})

# LED Drivers
The submarine requires a light source at the front to allow video in dark environments. For this there are 2 identical LED drivers which use pt4115 ics. The design allows for 1A constant output at 12V per ic. This ic uses constant current regulation but has a pwm connection to allow brightness control.

![led driver ]({{ 'assets/images/2026-07-22 led driver.png' | relative_url }})