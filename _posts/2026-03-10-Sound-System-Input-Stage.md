---
layout: post
title: "Sound System Research"
date: 2026-03-10
tags: [Sound System, Project]
---


# Sound System Project

**Disclaimer:
This project involves high-power electronics, including circuits that can carry dangerous voltages and currents. Improper design, construction, or handling may result in electric shock, burns, fire, equipment damage, or personal injury. The information shared here is for educational and documentation purposes and is not a complete guide. Therefore it should not be followed blindly. Stay safe.**

In the past couple of days during my spare time I have been working on the input stages of the sound system. this is everything leading to the amplifier, more specifically: headphone jack input, volume control, eq and buffering.
  
## Changes From Original Plan
I ended up changing the process order, as research led to me finding that it is more common to buffer the input, following this with volume control and finally the eq stage. The buffer earl on would prevent loading effects on the input device by providing high impedance.

Another major difference I made was switching to a dual power supply. My original plan was to use dc coupling to push signals to entirely positive voltage range and operate rail to rail op amps with single supply. However I noticed that dual power supplies cost relatively the same as single supply and therefore I though it would save the hassle of having to add dc offsets and decided to just go with the dual power supply.
  
# Input Stage

![Input Schematic image]({{ 'assets/images/2026-03-10 input.png' | relative_url }})

<center>Input schematic</center>

The first step in the audio processing is the headphone jack. this acts as the entrance point for audio signals into the system. I found that modern devices use a standard called CTIA for TRRS jacks. TRRS are the headphone jacks that include ground, 2 stereo lines and a microphone line. They are compatible with TRS as well (no microphone line).

  

  

After the signal enters the headphone jack I use a high pass filter to remove any unwanted noise. I made sure that the cut-off frequency of the filter wouldn't remove any frequencies from 20hz-20khz to ensure no audible cut-off. I found that isn't fully necessary but I did it just in case.

  

  

After being filtered a bit I went to a op amp based buffer. Buffers have a gain of 1. This means that the signal coming in will be the same as the signal coming out. It is wired by connecting the output back to the inverting input. It has extremely high input impedance and very low output impedance, isolating stages to prevent signal degradation caused by loading.

  

  

# Volume Stage

![Volume Schematic image]({{ 'assets/images/2026-03-10 volume.png' | relative_url }})

<center>Volume stage schematic</center>

  

The next part of the circuit is the volume control. This uses a simple logarithmic potentiometer to divert a varying amount of the signal to ground. This decreases or increases the amplitude of the audio signal without adding distortion or clipping. The logarithmic potentiometer should allow the volume to feel more "linear", since humans hear on a logarithmic scale.

  
  
  

# Equalizer

![Equalizer Left Schematic image]({{ 'assets/images/2026-03-10 left eq.png' | relative_url }})

![Equalizer Right Schematic image]({{ 'assets/images/2026-03-10 right eq.png' | relative_url }})

<center>Baxandall EQ schematic</center>

  

For the equalizer I went with a pretty widely used 3 band active Baxandall equalizer. It has tone control for highs mids and lows. And allows boosting thanks to the op amp. Without it you would only be able to reduce frequencies making it passive filter.

  

  

The first part of the eq uses some pretty simple RC circuits to first split the signal into frequency ranges (high, mid, low). And then from there the potentiometers decide how much of each frequency range is allowed through the feedback network.

  

  

As you might have noticed there are variations in each frequency section. The high pass filter uses a small capacitor which only allows high frequencies to pass through. The low pass filter uses a large capacitor which only lets lower frequencies through. While the mid control is made by using a band-pass filter which cuts of low and high frequencies only letting the middle frequencies through.

  

  

Next is the op amp stage. The op amp is setup as a inverting summing amplifier. It is used to bring back all the 3 separate signals into a single signal again, using weighted addition. the amplifier gain changes with Frequency, which allows different frequency ranges to be boost or cut. I think my explanation of this is not the best so feel free to check out [this](https://sound-au.com/articles/eq.htm) to get a better understanding of equalisers. There is some really good information on Baxandall eqs in it too.

**All op amps in schematics are OPA1656ID. They very low distortion and very low noise and are relatively cheap.**
  
  

# Next Steps

Now that I have completed a lot of the preliminary systems, my plan is to make a pcb layout and print the pcb using my schools free pcb printer. I want to go to the lab and test everything out on an oscilloscope before I finalize any designs. I will also continue to design the rest of the schematics, namely the active crossover, and the class d amplifier. The class d amplifier will likely be entirely from the datasheets so I think most of the hard design parts of this project are completed.

  

Stay tuned for updates.