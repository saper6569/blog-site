---
layout: post
title: "Sound System Crossover Design"
date: 2026-03-28
tags: [Sound System, Project]
---


# Sound System Project - Crossover Design

>**Disclaimer:
>This project involves high-power electronics, including circuits that can carry dangerous voltages and currents. Improper design, construction, or handling may result in electric shock, burns, fire, equipment damage, or personal injury. The information shared here is for educational and documentation purposes and is not a complete guide. Therefore it should not be followed blindly. Stay safe.**

  

## Background

One of the most important parts of a sound system is the crossover. This is what allows the sound to be split up based on frequency and sent to corresponding drivers. Speakers are designed to play in certain frequency ranges (you can see this by searching speaker drivers in digikeys and looking at the different frequency ranges), so to ensure that audio stays within the range of each speaker the crossover must split the frequencies into the correct bandwidth corresponding to the speakers. If speakers are playing frequencies outside of their recommended bandwidth sound can be distorted muddy or even cause damage to the drivers. The system I am designing is a 2.1 (2 channels setup with a subwoofer) in which audio signals have to be split into the low range for the subwoofer, while everything else is sent to the 2 stereo output channels. It is also important to keep the left and right channels isolated to preserve the stereo image. Since there is only one subwoofer the 2 channels have to be summed to create a mono channel.

  

![schematic]({{ 'assets/images/2026-05-07 schematic.jpg' | relative_url }})

<center>2 way crossover design schematic. As I am new to crossover design I largely used a design I found [here](https://sound-au.com/project09.htm). Feel free to check it out as it also has some pretty good explanations as to how it works.</center>

  

## Design and Consideration

This specific design implements a Linkwitz Riley crossover. It takes advantage of 4th order filters (24 dB/octave) to split audio into each speaker. The cuircuit contains 2 identical sections, one for each input channel (left and right). Like stated before the lows of each input channel are summed at the end using an op amp based summer to create the mono subwoofer output channel.

Common crossover frequencies for 2 way crossovers range from 80Hz to 120Hz. Through research I found that higher frequencies lead to subwoofer sounding localized. This is where the user can audibly "hear" the location of the subwoofer or hearing the bass coming from a specific spot in the room rather than it sounding uniformly distributed. I decided to go with a slightly higher frequency since I plan on using slightly less capable stereo drivers. I don't want them to have a hard time producing lower frequencies, and having a higher crossover frequency allows me to bypass that issue. Feel free to change values for the frequency, I included calculations for both 80Hz and 100Hz below.

### What are Linkwitz-Riley and Butterworth?

A Linkwitz-Riley crossover is constructed by cascading two identical Butterworth filters, one for the low-pass path and one for the high-pass path. 

**Butterworth filters** are commonly used because of their maximally flat passband response. This means they introduce no ripple or peaks in the frequencies they pass, the signal comes through as cleanly as possible before the cutoff begins. 

**Linkwitz-Riley crossovers** take two of these Butterworth filters (one low-pass, one high-pass, matched to the same cutoff frequency) and cascade each one with a copy of itself. This cascading doubles the filter order, which sharpens the cutoff significantly, while also producing outputs that are phase-aligned at the crossover point, allowing the summed response to remain flat. 

The 4th order (24 dB/octave) slope produced by this cascading also means very steep attenuation beyond the cutoff, keeping unwanted frequencies away from each driver and maintaining a controlled overlap region around the crossover frequency.

Each individual filter section in this design is implemented using a **Sallen-Key topology**. Sallen-Key is an active filter configuration built around a single op-amp wired as a unity-gain voltage follower, with a resistor-capacitor network forming filter.

The actual design can be broken into three parts the high pass filters, the lowpass filters and the stereo to mono summer. As stated before the filters use Sallen-Key filters which are cascaded with copies of eachother. Since each Sallen-Key filter is second order (12 dB/octave) cascading them creates a 4th order filter (24 dB/octave) without changing the original cutoff frequency.

### Low-Pass Stage
The low-pass stage places R1 and R2 in the signal path with capacitors sending high frequencies to ground. The op-amp is wired as a unity-gain follower. The RC network increasingly attenuates high frequencies, while the op-amp buffer prevents loading and preserves the intended filter response. Therefore low frequencies pass through to the output.

![schematic]({{ 'assets/images/2026-05-07 high pass filter.png' | relative_url }})

### High-Pass Stage
The high-pass stage places C1 and C2 in the signal path the, at low frequencies the capacitors have high impedance, attenuating low-frequency signals before they reach the op-amp. Again the op-amp acts as a unity-gain follower.  Therefore high frequencies pass through to the output.

![schematic]({{ 'assets/images/2026-05-07 low pass filter.png' | relative_url }})

### Op Amp Summer For Subwoofer

The final stage is a simple op amp based non inverting summer. This takes the 2 outputs from each channel's corresponding low pass filter and sums them into a single mono output for the subwoofer.

![schematic]({{ 'assets/images/2026-05-07 subwoofer summer.png' | relative_url }})

## Calculations

**Cutoff frequency (general):**

Each 2nd-order Sallen-Key stage is configured for a Butterworth response (Q = 0.707). Cascading identical stages does not change the crossover frequency.

$$f_c = \frac{1}{2\pi \sqrt{R_1 R_2 C_1 C_2}}$$

> **Note on component ratios for Q = 1/√2 (Butterworth):**
>
> To achieve Q = 0.707 in a unity-gain Sallen-Key filter, the HPF and LPF stages use *different* strategies:
>
> - **HPF**: equal capacitors (C1 = C2 = C) with **R2 = 2R1**, giving:
> $$f_c = \frac{1}{2\pi C \sqrt{R_1 \cdot 2R_1}} = \frac{1}{2\pi C R_1 \sqrt{2}}$$
>
> - **LPF**: equal resistors (R1 = R2 = R) with **C2 = 2C1**, giving:
> $$f_c = \frac{1}{2\pi R \sqrt{C_1 \cdot 2C_1}} = \frac{1}{2\pi R C_1 \sqrt{2}}$$

---

### Crossover Frequency: ≈ 82.8 Hz

**High-Pass Filter**: C1 = C2 = 68 nF, R1 = 20 kΩ, R2 = 40 kΩ (= 2 × 20 kΩ in series)

$$f_c = \frac{1}{2\pi \times 68 \times 10^{-9} \times \sqrt{20000 \times 40000}}$$

$$f_c \approx 82.8 \text{ Hz}$$

**Low-Pass Filter**: R1 = R2 = 20 kΩ, C1 = 68 nF, C2 = 136 nF (= 2 × 68 nF in parallel)

$$f_c = \frac{1}{2\pi \times 20000 \times \sqrt{68 \times 10^{-9} \times 136 \times 10^{-9}}}$$

$$f_c \approx 82.8 \text{ Hz}$$

---

### Crossover Frequency: ≈ 99.8 Hz

**High-Pass Filter**: C1 = C2 = 47 nF, R1 = 24 kΩ, R2 = 48 kΩ (= 2 × 24 kΩ in series)

$$f_c = \frac{1}{2\pi \times 47 \times 10^{-9} \times \sqrt{24000 \times 48000}}$$

$$f_c \approx 99.8 \text{ Hz}$$

**Low-Pass Filter**: R1 = R2 = 24 kΩ, C1 = 47 nF, C2 = 94 nF (= 2 × 47 nF in parallel)

$$f_c = \frac{1}{2\pi \times 24000 \times \sqrt{47 \times 10^{-9} \times 94 \times 10^{-9}}}$$

$$f_c \approx 99.8 \text{ Hz}$$

---

### Component Summary

| Frequency | Type | R1 | R2 | C1 | C2 | Actual frequency |
|--------|------|----|----|-----|-----|------------|
| ≈ 80 Hz | High-Pass | 20 kΩ | 40 kΩ | 68 nF | 68 nF | 82.8 Hz |
| ≈ 80 Hz | Low-Pass  | 20 kΩ | 20 kΩ | 68 nF | 136 nF | 82.8 Hz |
| ≈ 100 Hz | High-Pass | 24 kΩ | 48 kΩ | 47 nF | 47 nF | 99.8 Hz |
| ≈ 100 Hz | Low-Pass  | 24 kΩ | 24 kΩ | 47 nF | 94 nF | 99.8 Hz |

> R2 in each HPF is two R1 resistors in series; C2 in each LPF is two C1 capacitors in parallel.
> Each row represents one stage, cascade two identical stages per path for the full 4th-order LR response.

## Next Steps

Next I plan on running spice simulations to ensure that all the circuitry is working as intended. I will also complete the amplifiers and hopefully soon start some pcb layouts. I also completed majority of the part selection so I will leave an update on that soon.