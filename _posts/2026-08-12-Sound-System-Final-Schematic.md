---
layout: post
title: "Sound System Final Schematic"
date: 2026-08-12
tags: [Sound System, Project]
---

>**Disclaimer:
>This project involves high-power electronics, including circuits that can carry dangerous voltages and currents. Improper design, construction, or handling may result in electric shock, burns, fire, equipment damage, or personal injury. The information shared here is for educational and documentation purposes and is not a complete guide. Therefore it should not be followed blindly. Stay safe.**

Since the last iteration various changes and additions have been made to the schematic. The main additions being the addition of a custom ESP32-WROOM for bluetooth capabilities and peripheral communications, as well as the TPA3221 class D amplifiers. Original audio circuits remain largely unchanged. These additions are integrated seamlessly into the existing circuits as best as possible. Circuitry for power measurement using a INA226, stereo channel digital to analog conversion using a PCM5102 and audio multiplexing using a TMUX4053 have also been added. 

# Power
The input power comes from a dual rail power supply which provides +-24V DC. This is fed through fuses to prevent shorts and large current spikes that can damage circuitry. From the 24V rails the +24 is brought down to 5V using a Xl1509-5.0. The +-24V is also used to create a dual +-12V supply which is used for powering the operational amplifiers. Since they require so little current two LDOs are used for this, an L7812 for the +12V and an L7912 for the -12V.

![10 ]({{ 'assets/images/2026-08-12 10.png' | relative_url }})

# Microcontroller
As stated above an ESP32-WROOM was added for bluetooth capabilities and peripheral communications. This will allow audio to be streamed through bluetooth while also communicating with the onboard TPA3221, INA226, PCM5102 and TMUX4053. The main reason for choosing an older ESP32-WROOM is that it supports Bluetooth Classic and the A2DP (Advanced Audio Distribution Profile) sink protocol. Newer ESP32s use standard BLE (Bluetooth Low Energy) chips that struggle with high-bandwidth streaming, ESP32-WROOMs handles full audio sink capabilities. 

The specific esp-32 model being used is a ESP32-WROOM-32UE-N16 module. The ESP32-WROOM-32UE-N16 is a compact microcontroller module. it uses a Xtensa LX6 processor operating at up to 240 MHz, 16 MB of external SPI flash memory, and 520 KB of internal SRAM. It operates from a 3.0–3.6V supply, in this implementation 3.3 V, and uses an external antenna connector. 

![3 ]({{ 'assets/images/2026-08-12 3.png' | relative_url }})

### Bootloading
The ESP32-WROOM-32UE-N16 can be bootloaded through its UART interface using the ESP32’s built-in ROM bootloader. To enter bootloader mode, the GPIO0 (BOOT) pin is pulled low while the EN (RESET) pin is toggled high, causing the module to start in serial download mode. Test pads are included to allow manual pulling to gnd or 3.3v as well as for connecting a probe for testing. An external USB-to-UART converter can then be connected to TXD0 (GPIO1) and RXD0 (GPIO3) to communicate with the bootloader and upload firmware to the module’s 16 MB flash memory. During normal operation, GPIO0 is kept high so that the ESP32 starts the previously programmed application instead of entering download mode. 

GPIO2 is pulled LOW to ensure the appropriate boot configuration, while GPIO5 is pulled HIGH to select the required SDIO timing configuration. GPIO12 is pulled LOW to select the appropriate 3.3 V flash voltage configuration for the module, and GPIO15 is pulled HIGH to establish the normal boot and startup configuration. These pull-up and pull-down resistors ensure that the ESP32 consistently starts in the intended operating mode.

## Voltage Converter
To power the microcontroller as well as the connected peripherals a Xl1509-3.3 is used. The Xl1509-3.3 is a fixed output voltage pwm switching buck converter which is used to bring 24V down to 3.3V. This ic has the capability of supplying up to 3A output current which is well above the needs of the connected circuitry.  

## Power Measurement (INA226)
To measure battery capacity a current and voltage sensing ic is used. The INA226 is a high-precision digital current shunt and power monitor that measures bus voltage, current flow, and calculated power. The INA226 takes in supply voltage of 2.7V-5.5V and a bus voltage og 0V-36V, both of which fit the specifications of this board. Data is communicated through I2C, to a programmable address. The ic is connected to a high side 0.001 Ω shunt resistor which was added to the 24V input line. Since the TPA3221 is single supply the +24V line is the only one that is being measured. Below are the schematics.

![4 ]({{ 'assets/images/2026-08-12 4.png' | relative_url }})
![1 ]({{ 'assets/images/2026-08-12 1.png' | relative_url }})

## Digital to Analog Converter (PCM5102 DAC)
ESP32-WROOM models have 8 bit dacs built in which bottleneck the audio quality. To make up for this an external PCM5102 based high-performance, 32-bit, 384kHz stereo digital-to-analog converter (DAC) from Texas Instruments is used. This allows CD audio quality to be output. This chip communicates with the esp32 through I2S (Inter-IC Sound) communication protocol. I2S is a synchronous serial communication protocol used to transfer digital pulse-code modulation (PCM) audio data between integrated circuits, in this case the microcontroller to the DAC. It is noted that the internal hardware of older ESP32-WROOM cannot reliably generate a high-frequency, jitter-free master clock (MCLK) signal while simultaneously running Bluetooth. To work around this the PCM5102 does not require a MCLK signal which makes it well suited for this situation.

![5 ]({{ 'assets/images/2026-08-12 5.png' | relative_url }})

## Multiplexing (TMUX4053)
To toggle between the two audio inputs (headphone jack and bluetooth audio from DAC) a 2:1 multiplexer is used where the select is controlled by the ESP32. This allows the ESP32 to decide which audio channel will be passed through to the rest of the audio chain. A secondary use for the multiplexer is to mute the system. This is done by switching to output the audio coming from the DAC and muting the channel. The selected ic for this task is the TMUX4053. 

The TMUX4053 is a high-performance triple 2-channel analog multiplexer. It allows switching between analog signals while maintaining low on-resistance, low distortion, and excellent channel matching. The TMUX4053 operates from a wide supply voltage range and supporting rail-to-rail signal switching. Its low leakage current and high bandwidth helps preserve signal integrity.

![2 ]({{ 'assets/images/2026-08-12 2.png' | relative_url }})

>**Note: It is important to note that the TMUX4053 is not explicitly intended for audio purposes, design considerations had to be made to make up for this. Since the multiplexer leads into a buffer (high impedance) there should be minimum voltage divider distortion as well as loading effects on the input signals.  **

# Amplifier Circuits (TPA3221)
As stated in a previous post the selected amplifier is a TPA3221 class D amplifier. The reason for selection is also stated, find that information [here](/2026-03-05-Sound System.md#research). The design uses two separate TPA3221s, one for the stereo channels in a BTL configuration and another for the mono channel for the subwoofer in a PBTL configuration. The stereo rails allows for a speaker impedance of 3Ω-8Ω while the mono allows for 2Ω-4Ω. The output power has an absolute cap of 200W from each TPA3221, however they will be operated well below that range. The TPA3221s use the full 24V as a single supply for PVDD and 5V for the AVDD. While the ic contains an internal LDO to bring PVDD down to AVDD an external 5V rail is used to minimize loss from the LDO.

![6 ]({{ 'assets/images/2026-08-12 6.png' | relative_url }})
![9 ]({{ 'assets/images/2026-08-12 9.png' | relative_url }})

![7 ]({{ 'assets/images/2026-08-12 7.png' | relative_url }})
![8 ]({{ 'assets/images/2026-08-12 8.png' | relative_url }})

On the schematic the two OSCM and OSCP of both ics are tied together. When multiple TPA3221 amplifiers are used on the same board, one device acts as the controller (master) and generates the switching clock, while the other acts as a peripheral (slave) and locks to that clock. Tying the OSCP and OSCM pins together between the two amplifiers synchronizes their PWM switching frequencies, preventing beat frequencies and reducing EMI and noise. Furthermore, the GAIN/SLV pin is a multifunction pin that selects both the amplifier gain and whether the device operates as a controller or peripheral. Pulling the pin to a specific voltage using a voltage divider places the device into one of the predefined gain/slave configurations defined in Table 8-1 from the datasheet. The DAC is capable of outputting 2VRMS while phone jacks typically outputting 1VRMS, which corresponds to a gain of 30dB based on the limiting 1VRMS. The configuration used selects a gain of 30dB and selects the stereo ic as the controller and mono as the peripheral. When in peripheral mode the internal oscillator is turned off by pulling FREQ_ADJ high to AVDD.

The input impedance of a TPA3221 changes based on the set gain. Since the gain is set at 30dB the input impedance of the ic is 12kΩ. The coupling capacitor used to block dc voltage at the input creates an RC highpass filter so a high enough capacitance has to be chosen to not cutoff any audible frequency range. A 1uf capacitor makes the cutoff frequency 12hz which is below the audible range.

The TPA3221 RESET pin is an active-low input used to place the amplifier into its reset state. Pulling RESET low disables the amplifier and resets its internal circuitry, while pulling it high enables normal operation. The pin should be held low during power-up and released high only after the power supplies and input signals have stabilized. This pin is controlled by the ESP32 for both TPA3221s and is pulled down to ground by default to ensure it is held low even while the ESP32 boots. 

The TPA3221 CMUTE pin is used to control the amplifier's mute function. When CMUTE is asserted, the amplifier output is muted, preventing the audio signal from being amplified and sent to the speaker. Releasing CMUTE enables normal audio operation. This pin is also controlled by the ESP32 and is useful for preventing unwanted noise or transients during power-up, power-down, or other system events. During the power-down ramp, the CMUTE capacitor is discharged by internal circuitry. With the 33nF CMUTE capacitor, the power-down ramp time is approximately 20 ms.

## Error Handling
The TPA3221 also has pins for outputting information on operation. This data is read and handled by the ESP32. below is Table 8-3. Error Reporting from the datasheet which describes the function of FAULT and OTW_CLIP:

| FAULT | OTW_CLIP | Description |
|--------|----------|-------------|
| 0 | 0 | Overtemperature (OTE), overload (OLP), or undervoltage (UVP). Junction temperature higher than 125°C (overtemperature warning). |
| 0 | 1 | Overload (OLP) or undervoltage (UVP). Junction temperature lower than 125°C. |
| 1 | 0 | Junction temperature higher than 125°C (overtemperature warning). |
| 1 | 1 | Junction temperature lower than 125°C and no OLP or UVP faults (normal operation). |