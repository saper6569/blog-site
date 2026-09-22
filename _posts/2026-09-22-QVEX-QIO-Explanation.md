---
layout: post
title: "QIO Board Explanation"
date: 2026-09-22
tags: [QVEX-QIO, Design Team, PCB Documentation]
---
>**Note: This document might seem vague/generalized. This is done on purpose to protect sensitive design information.**

# Introduction
What is QIO? QIO is the electrical subteam in charge of the input output for the QVEX robot. Since the robot requires various sensor readings it also requires a mode of communication between those sensors and the brains, as well as a way to provide those sensors with the required power. This has to be taken into consideration in the design of the PCB. QIO also handles a lot of the signal routing and designing for minimized noise/EMI. 

This year QIO is moving away from a single board design to a more modular solution. As part of this change one of the main tasks is separating the QIO section into individual PCBs. The current system works on a main STM32 microcontroller communicating with three separate STM32 boards through SPI. The three separate stm32s are connected to their corresponding three QIO ports (nine total). QIO ports act as an interfaces between sensors and the STM32, and allow for sensor data to be collected through SPI and/or I²C depending on the specific port. 

The main reasons for the three individual STM32s for QIO is for port expansion past that of a single stm32, as well as the ability to run instruction in parallel on the separate microcontroller. By separating the board into section pcb manufacturing costs will be reduced (larger pcbs are more costly to print and import). Another important effect of separating the pcb is that modularity makes it easier to isolate faults in hardware. System isolation makes it easier to pinpoint hardware issues since if it is known that the issue is coming from one board, it is easier to to debug a single smaller board than a larger one. Finally, it also lowers cost in the chance that pcbs have to be reprinted due to issues, as rather than reprinting the large pcb again, the smaller subsystem pcb that the issue is present in can be re-manufactures.

# Microcontroller
The microcontroller implemented on the QIO boards is a STM32. The exact model will not be disclosed here to protect the design. The model used is a 32-bit microcontroller from STMicroelectronics based on the Arm Cortex-M0+ core. It operates at up to 64 MHz and provides a wide range of peripherals, including GPIO, timers, ADCs, SPI, I²C, and UART interfaces. The device offers up to 512 KB of Flash memory and 144 KB of SRAM, making it suitable for moderate processing capability, multiple communication interfaces, and low power consumption, all of which this application requires.

# Power
Since the duaghterboards are isolated from the main board, where the power circuitry is located, external connections need to be used to supply power. The Power bus supplies the needed 3.3V, 5V and GND lines to each individual board from the buck converters on the main board. These connections are isolated from the digital bus that connects to the microcontroller to minimize noise on the digital signals. 

The STM32is powered using 3.3V and has an output and input level of 3.3V, exceeding this voltage can cause damage to the chip. Another important precaution is the power supply has to be designed to tolerate the inrush current of both the stm32 as well as connected sensors. If the power supply is not designed for this the microcontroller can cause brownout which can break expected functionality. 

# QIO Ports
Each port allows connection between sensors and the corresponding stm32. Ports come with three possible modes of communication: SPI, I²C, UART and/or analog. The mode of communication depends on the pin configuration of the stm32. Note that only one communication protocol is used at a time by a single port (software/firmware controlled). Each port also provide a GND, 5V and 3.3V line for powering the connected peripheral. This design allows for the most robust hardware solution as different protocols can be harness by a single port configuration without needing changes to hardware. Ports includes TVS diode protection which are further discussed later. Port 3 should be noted that it does not support spi.

# Communication
understanding the various communication protocols used on the project are essential to understanding the connections between the different systems. The main communication/data transmission methods that are included in the design are I²C, SPI, ADC, serial UART and USB. These are used for various purposes including communication between the odom board and qio boards, qio boards and the connected peripherals as well as for debug and programming the qio boards externally. 

## I²C
I²C (Inter-Integrated Circuit) is a two-wire communication protocol commonly used for connecting microcontroller to sensors, displays, memory, and other peripherals. It uses a SDA data line and SCL clock line, with multiple devices able to share the same bus. Each device is identified by a unique address, allowing the microcontroller to communicate with a specific peripheral.

## SPI
SPI (Serial Peripheral Interface) is a high-speed synchronous communication protocol typically used for peripherals such as displays, ADCs, DACs, and memory. It generally uses four signals: MOSI, MISO, SCLK, and CS. Unlike I²C, SPI does not use device addresses; instead, each peripheral typically has its own chip-select line.

## ADC
An ADC (Analog-to-Digital Converter) converts an analog voltage into a digital value that a microcontroller can process. The ADC samples the input voltage and represents it using a fixed number of bits, with higher resolution providing more possible digital values. ADCs are commonly used to measure sensors, potentiometers, battery voltage, and other analog signals.

## Serial Debug
Serial debugging provides a simple way to send information between a microcontroller and a computer for monitoring and troubleshooting. The implementation used on this design is UART, which uses transmit (TX) and receive (RX) lines to send data serially. It can be used to print variable values, error messages, system status, or other diagnostic information.

## USB
USB (Universal Serial Bus) is a communication interface commonly used to connect a microcontroller to a computer or other USB host. It can provide both data communication and power through the same connector, although in this design it does not provide power. 

# TVS Diodes
A TVS (Transient Voltage Suppression) diode is a device used to protect electronic circuits from short-duration voltage spikes, such as those caused by ESD, inductive switching, or cable transients. Under normal operating voltages, the TVS diode remains essentially inactive. When the voltage exceeds its breakdown or clamping voltage, the diode rapidly conducts and clamps the voltage to a safer level, diverting the transient current away from sensitive components.

For example, a TVS diode can be placed on an SPI or USB signal line near a connector. If an external electrostatic discharge creates a large voltage spike on the connector, the TVS diode conducts and limits the voltage reaching the microcontroller. TVS diodes are therefore commonly used on external interfaces and power inputs where the circuit may be exposed to the outside environment.
