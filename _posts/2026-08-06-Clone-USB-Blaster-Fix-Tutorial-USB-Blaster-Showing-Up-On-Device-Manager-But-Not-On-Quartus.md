---
layout: post
title: "Clone USB Blaster Fix Tutorial (USB Blaster Showing Up On Device Manager But Not On Quartus"
date: 2026-08-06
tags: [Tutorial, FPGA, Altera, Quartus, USB-Blaster]
---

I recently ran into an issue where I was trying to use the Quartus programmer on an Altera cyclone IV board but it could not detect the USB-blaster for programming, despite it being shown on my device manager. 

![1 ]({{ 'assets/images/2026-08-06 1.png' | relative_url }})
![2 ]({{ 'assets/images/2026-08-06 2.png' | relative_url }})
![3 ]({{ 'assets/images/2026-08-06 3.png' | relative_url }})

I was using a cheap clone from Aliexpress and it is known that often manufacturers replace parts with cheaper counterparts. I found that my USB-Blaster was using a cheap CH55x chip instead of a two-chip design combining an FTDI FT2232D (or FT245-family) USB-to-serial/parallel interface chip with an Altera CPLD (such as the EPM3064) to convert USB protocols into JTAG signals. **The solution to this issue is flashing the CH55x chip with the .bin file provided by dougg3 (original credit to VladimirDuan) on github. Provided below is a step by step guide. **

# Checks
Before beginning the instructions below make sure that you have first checked all of the following:
- check that the needed USB-blaster driver is installed
- check that the blaster is connected to your fpga and that the fpga powered separately 

# Instructions

### 1. Download chip programmer
Download the "WCHISPTool_Setup" .exe file from [https://www.wch-ic.com/downloads/WCHISPTool_Setup_exe.html ](https://www.wch-ic.com/downloads/WCHISPTool_Setup_exe.html) and run the installer. Follow the installation wizard. 
![3 ]({{ 'assets/images/2026-08-06 9.png' | relative_url }})

### 2. Download .bin file
Download the needed .bin file from [https://github.com/dougg3/CH55x-USB-Blaster/releases/tag/v1.1.0](https://github.com/dougg3/CH55x-USB-Blaster/releases/tag/v1.1.0). Navigate to the linked page and click on usb_blaster.bin under the assets dropdown. This should begin the installation.   
![8 ]({{ 'assets/images/2026-08-06 10.png' | relative_url }})

### 3. Set CH55x chip into flash mode using 10k resistor
To set the chip to flash mode open up your USB-blaster and use a 10k resistor to pull up D+ to 3.3V. For this any resistor over 1k will do the job.
![11 ]({{ 'assets/images/2026-08-06 11.jpg' | relative_url }})

### 4. Plug USB-blaster into computer
After the chip programming software is installed and running plug in the USB-blaster with the resistor connected into your computer. The software should automatically detect the device and setup the chip options. 
![12 ]({{ 'assets/images/2026-08-06 12.png' | relative_url }})

### 5. Open .bin file on chip programmer
Click on the three dots beside "Object File1" in the "Download File" section, this should open a file browser window. Navigate to your .bin file in the file browser and click open. Ensure that the check box beside the three dots is checked.
![13 ]({{ 'assets/images/2026-08-06 13.png' | relative_url }})

### 6. Flash the chip
Click "Download" to begin flashing the chip. 
![5 ]({{ 'assets/images/2026-08-06 5.png' | relative_url }})

### 6. Remove the resistor to exit flashing mode
After removing the resistor and plugging the USB-blaster into my fpga Quartus recognized the blaster.
![7 ]({{ 'assets/images/2026-08-06 7.png' | relative_url }})

# Final comments
Feel free to email me at saper6569@gmail.com if you have any questions.