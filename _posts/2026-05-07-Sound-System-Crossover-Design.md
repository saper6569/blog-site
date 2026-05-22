---
layout: post
title: "Sound System Crossover Design"
date: 2026-03-28
tags: [Sound System, Project]
---


>**Disclaimer:
>This project involves high-power electronics, including circuits that can carry dangerous voltages and currents. Improper design, construction, or handling may result in electric shock, burns, fire, equipment damage, or personal injury. The information shared here is for educational and documentation purposes and is not a complete guide. Therefore it should not be followed blindly. Stay safe.**

  

# Background

One of the most important parts of a sound system is the crossover. This is what allows the sound to be split up based on frequency and sent to corresponding drivers. Speakers are designed to play in certain frequency ranges (you can see this by searching speaker drivers in Digikey and looking at the different frequency ranges), so to ensure that audio stays within the range of each speaker the crossover must split the frequencies into the correct bandwidth corresponding to the speakers. If speakers are playing frequencies outside of their recommended bandwidth sound can be distorted muddy or even cause damage to the drivers. The system I am designing is a 2.1 (2 channels setup with a subwoofer) in which audio signals have to be split into the low range for the subwoofer, while everything else is sent to the 2 stereo output channels. It is also important to keep the left and right channels isolated to preserve the stereo image. Since there is only one subwoofer the 2 channels have to be summed to create a mono channel.

  

![schematic]({{ 'assets/images/2026-05-07 schematic.jpg' | relative_url }})

<center>
2 way crossover design schematic. As I am new to crossover design I largely used a design I found 
<a href="https://sound-au.com/project09.htm">here</a>. 
Feel free to check it out as it also has some pretty good explanations as to how it works.
</center>

  

# Design and Consideration

This specific design implements a Linkwitz Riley crossover. It takes advantage of 4th order filters (24 dB/octave) to split audio into each speaker. The circuit contains 2 identical sections, one for each input channel (left and right). Like stated before the lows of each input channel are summed at the end using an op amp based summer to create the mono subwoofer output channel.

Common crossover frequencies for 2 way crossovers range from 80Hz to 120Hz. Through research I found that higher frequencies lead to subwoofer sounding localized. This is where the user can audibly "hear" the location of the subwoofer or hearing the bass coming from a specific spot in the room rather than it sounding uniformly distributed. I decided to go with a slightly higher frequency since I plan on using slightly less capable stereo drivers. I don't want them to have a hard time producing lower frequencies, and having a higher crossover frequency allows me to bypass that issue. **I have my design for 100Hz but feel free to change components to make it whatever you want. I included a tool for finding component values below.**

## What are Linkwitz-Riley and Butterworth?

A 4th-order Linkwitz–Riley crossover is formed by cascading two identical 2nd-order Butterworth filter stages in each signal path (low-pass and high-pass). The resulting overall response has a −6 dB crossover point (both filter responses intersect) and sums to a flat response that is in phase.

**Butterworth filters** are commonly used because of their maximally flat passband response. This means they introduce no ripple or peaks in the frequencies they pass, the signal comes through as cleanly as possible before the cutoff begins. A 2nd-order Butterworth filter section has a Q factor of approximately 0.707, which produces the maximally flat response characteristic.
 
**Linkwitz-Riley crossovers** take two of these Butterworth filters (one low-pass, one high-pass, matched to the same cutoff frequency) and cascade each one with a copy of itself. This cascading doubles the filter order, which sharpens the cutoff significantly, while also producing outputs that are phase-aligned at the crossover point, allowing the summed response to remain flat. 

The 4th order (24 dB/octave) slope produced by this cascading also means very steep attenuation beyond the cutoff, keeping unwanted frequencies away from each driver and maintaining a controlled overlap region around the crossover frequency.

Each individual filter section in this design is implemented using a **Sallen-Key topology**. Sallen-Key is an active filter configuration built around a single op-amp wired as a unity-gain voltage follower, with a resistor-capacitor network forming the filter.

The actual design can be broken into three parts the high pass filters, the lowpass filters and the stereo to mono summer. As stated before the filters use Sallen-Key filters which are cascaded with copies of each other. Since each Sallen-Key filter is second order (12 dB/octave) cascading them creates a 4th order filter (24 dB/octave) without changing the original cutoff frequency.

## Low-Pass Stage
The low-pass stage places R1 and R2 in the signal path with capacitors sending high frequencies to ground. The op-amp is wired as a unity-gain follower. The RC network increasingly attenuates high frequencies, while the op-amp buffer prevents loading and preserves the intended filter response. Therefore low frequencies pass through to the output.

![schematic]({{ 'assets/images/2026-05-07 high pass filter.png' | relative_url }})

## High-Pass Stage
The high-pass stage places C1 and C2 in the signal path the, at low frequencies the capacitors have high impedance, attenuating low-frequency signals before they reach the op-amp. Again the op-amp acts as a unity-gain follower.  Therefore high frequencies pass through to the output.

![schematic]({{ 'assets/images/2026-05-07 low pass filter.png' | relative_url }})

## Op Amp Summer For Subwoofer

The final stage is a simple op amp based non inverting summer. This takes the 2 outputs from each channel's corresponding low pass filter and sums them into a single mono output for the subwoofer.

![schematic]({{ 'assets/images/2026-05-07 subwoofer summer.png' | relative_url }})

# Online Crossover Calculator  

> **Component namings are consistent with the images in sections above [low pass](#low-pass-stage) and [high pass](#high-pass-stage)**

[This](http://sim.okawa-denshi.jp/en) is the resource I used for component calculations. I used their [Sallen-Key lowpass filter](http://sim.okawa-denshi.jp/en/OPseikiLowkeisan.htm) and [Sallen-Key highpass filter](http://sim.okawa-denshi.jp/en/OPseikiHikeisan.htm) calculators. To use them click the hyper links and navigate down to "Calculate the R and C values for the Sallen-Key filter at a given frequency and Q factor". Here you are able to enter a frequency and Quality factor Q on the right. ensure that the quality factor radio button is selected and press calculate. This will give you the values for the components.
> **Note: The calculator find values using standard E12/E24 component values. You might notice that the frequency or Q factor are slightly off from what you entered. Try to keep them as close as possible by changing around the frequency.** 

# Crossover Filter Calculations

>**Note: Values used here are calculated using an online calculator, this is for confirmation.**
 
**Cutoff frequency (general):**

Each 2nd-order Sallen-Key stage is configured for a Butterworth response (Q = 0.707). Cascading two identical stages gives the full 4th-order Linkwitz–Riley response.
 
$$f_c = \frac{1}{2\pi \sqrt{R_1 R_2 C_1 C_2}}$$
 
 
---
 
## Crossover Frequency: ≈ 100 Hz
 
**High-Pass Filter:**
 
C1 = C2 = 100 nF, R1 = 22 kΩ, R2 = 11 kΩ 
$$f_c = \frac{1}{2\pi \times 100 \times 10^{-9} \times \sqrt{11000 \times 22000}}$$
 
$$f_c = \frac{1}{2\pi \times 100 \times 10^{-9} \times 15{,}556}$$
 
$$f_c \approx 102.3 \text{ Hz}$$
 
$$Q = \frac{1}{2}\sqrt{\frac{R_2}{R_1}} = \frac{1}{2}\sqrt{2} \approx 0.707 \checkmark$$
 
---
 
**Low-Pass Filter:**
 
R1 = 30 kΩ, R2 = 18 kΩ, C1 = 47 nF, C2 = 100 nF 
 
$$f_c = \frac{1}{2\pi \sqrt{18000 \times 30000 \times 100 \times 10^{-9} \times 47 \times 10^{-9}}}$$
 
$$f_c = \frac{1}{2\pi \times 1.593 \times 10^{-3}}$$
 
$$f_c \approx 99.9 \text{ Hz}$$
 
$$Q = \frac{\sqrt{R_1 R_2 \cdot C_1/C_2}}{R_1 + R_2} = \frac{\sqrt{18000 \times 30000 \times (100/47)}}{48000} = \frac{33{,}897}{48{,}000} \approx 0.706 \checkmark$$
 
---
 
## Component Summary
 
| Target | Type | R1 | R2 | C1 | C2 | Actual fc |
|--------|------|----|----|-----|-----|-----------|
| ≈ 100 Hz | High-Pass | 22 kΩ | 11 kΩ | 100 nF | 100 nF | 102.3 Hz |
| ≈ 100 Hz | Low-Pass  | 30 kΩ | 18 kΩ | 47 nF | 100 nF  | 99.9 Hz  |
 
> **The small difference between HPF (102.3 Hz) and LPF (99.9 Hz) crossover frequencies is a consequence of using standard E12/E24 component values. Both paths are within 2.5% of 100 Hz.**

# Limitations

## Crossover Frequency Mismatch

A Linkwitz-Riley crossover performs best when the low-pass and high-pass sections share the same cutoff frequency. When both sections are closely matched, their phase and amplitude responses combine to produce a nearly flat summed output through the crossover region.

In this design, the target crossover frequency was chosen within the 80–120 Hz range. The resulting cutoff frequencies are approximately **99.9 Hz** for the low-pass section and **102.3 Hz** for the high-pass section. This small mismatch is a result of using standard **E12 and E24 series** resistor and capacitor values. These are common and widely available component series, but they only provide discrete values, making it difficult to achieve an exact target frequency without combining components or using higher-precision parts.

The values selected are the closest practical matches provided by the online calculator referenced earlier, resulting in a difference of approximately 2.4 Hz between the two filter sections.

### Effect on Linkwitz-Riley Behaviour

Because the crossover frequencies are not perfectly identical, the crossover will not behave as a mathematically ideal Linkwitz-Riley design. However, the mismatch is relatively small, so the practical impact is expected to be minimal.

Some possible effects include:

- **Slight deviation from perfectly flat summation**: Around the crossover region, the low-pass and high-pass outputs may not combine into a perfectly flat response. Any resulting dip or variation is expected to be small.
- **Minor phase offset near crossover**: Since the two sections do not reach their −6 dB points at exactly the same frequency, there may be a small phase difference between the outputs near the crossover region.
- **Butterworth characteristics remain intact within each stage**: Each individual filter stage still maintains the intended Butterworth response because the internal component ratios preserve the target quality factor of approximately 0.707.

## Practical Impact

In practice, a 2.4 Hz difference is relatively minor, corresponding to roughly 2.4% of the target crossover frequency. Real-world speaker systems already contain additional sources of variation such as driver tolerances, acoustic placement, enclosure effects, and room acoustics, so this small electrical mismatch is unlikely to produce a noticeable audible issue in most applications. The magnitude of these effects will become more evident through SPICE simulation. 

If greater precision is required, the cutoff frequencies could be matched more closely by using E96 series resistors, combining components in series or parallel, or incorporating adjustable trimmer components.