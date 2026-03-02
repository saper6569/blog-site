---
layout: post
title: "Power System Changes"
date: 2026-03-01
tags: [Underwater ROV, Project]
---

# UAV Power Systems Changes
Recently majority of my effort has been going into designing and implementing the power systems for the sub. The battery array we opted for is a 3s5p system. from previous calculations we found that it would be able to run for 30 minutes at a high energy consumption. The system has to be rated for upwards of 40A to ensure safety. 
![PCB ]({{ 'assets/images/2026-03-01 3d.png' | relative_url }})

## Main Systems 
- Battery Array (3s5p)
- Reverse Polarity Protection
- Battery Management system
- Safety System (Fuse, kill switch)
- Battery Fuel Gauge
- Buck Step Down Converters
- Power Rails
![Schematic ]({{ 'assets/images/2026-03-01 schem.jpg' | relative_url }})

## Design Considerations
**High Current**
The main consideration made for the design was safety. At high current my main focus was using strategies to reduce heat in the pcb. I focused on minimizing trace length while maximizing trace width. I did this by carefully planning component placement so that component that would draw the highest current were as close to the battery leads as possible. I also utilized the entire board ensuring large copper pours for high current traces. I also ran into issues when using multiple layers as I found that vias could not support high current paths. my workaround for this issue is simply just using more vias spread throughout traces. 

**Cost**
Another major consideration was cost. Majority of my component decisions would be heavily affected by budget. This project is entirely funded by me and my friends who are working on it. As a result while better solutions to issues exist I would often opt for the cheaper less effective solutions. this is especially evident in the battery management system, which i talk about later. 

**Noise**
The final consideration is the environment that the pcb will be located in. It will be in a underwater uav, cramped with other electronics. The main issue with this is noise reduction. High electromagnetic interference will effect sensors and other electronics on board the sub. To reduce this the main strategy was just using proper grounding techniques. 

## Battery Management System (BMS)
As design for the power systems of the sub have progressed I have spent a lot of time researching both battery management systems (BMS) as well as battery fuel gauges. after my research I came to the conclusion that designing the BMS might not be worth it. For starters the system would need to handle a 3s5p battery array which has a nominal voltage of 11.1V, but more importantly a max discharge current upwards of 40A. When working in this current range mistakes can end up casing fires. On top of this BMSs for 3s systems are often quite inexpensive, even for higher current ratings. Manufacturing it ourselves would include pcb printing charges that already make it more expensive than just buying it prebuilt.  

**As a result of this I have decided to opt for a premade BMS module and incorporating it into the design.** 