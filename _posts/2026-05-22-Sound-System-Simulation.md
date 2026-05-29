---
layout: post
title: "Sound System SPICE Simulations"
date: 2026-05-22
tags: [Sound System, Project]
---

>**Disclaimer:
>This project involves high-power electronics, including circuits that can carry dangerous voltages and currents. Improper design, construction, or handling may result in electric shock, burns, fire, equipment damage, or personal injury. The information shared here is for educational and documentation purposes and is not a complete guide. Therefore it should not be followed blindly. Stay safe.**

>**Note: LTspice will be used for simulations however any spice software can be used to gain the same results. All models and simulations are included for anyone who would like to explore, modify, or run the simulations themselves.**

# Background
Simulation Program with Integrated Circuit Emphasis (SPICE) is a tool used for predicting the behaviour of a design. It allows verification and analysis of circuits much more efficiently than manual calculations. SPICE relies on mathematical models in analysing circuits. It can be used for DC, AC, and transient analysis. Usage of SPICE can allow the designer to pinpoint potential issues before they are actually implemented.

It is important to note that results from spice simulation will often differ from the real-world implementations, which is discussed later.

## Simulation Objectives
As stated above, spice simulation allows relatively simple design validation. This is to ensure that the system works as intended and to identify any issues. For preliminary analysis, simulations are run for both AC analysis and transient analysis. AC analysis is used to analyze the frequency response of the crossover. This is to validate filter behaviour, such as crossover frequency and filter slopes. Overall, the overarching purpose of running this analysis is to confirm the behaviour of the Linkwitz-Riley crossover.

The transient analysis is to evaluate the full system behaviour in the time domain using real audio input. In practical terms, it provides an indication of whether the signal processing behaves as expected when playing music. Plotting waveforms at various locations in the system allows the effects of each stage to be analyzed. Apart from qualitatively analysing output through listening, waveforms can be analysed to show distortion, clipping, phase misalignment, and saturation effects.

# Circuit Implementation And Assumptions
The system being modeled consists of 4 parts: the input stage, the volume control, the equalizer stage, and the crossover stage. The 4 stages are cascaded while also keeping left and right channels isolated. A schematic of the system is included below for reference.

![schematic]({{ 'assets/images/2026-05-15 system.png' | relative_url }})

All operational amplifiers are modeled using LTspice's built-in model “opamp”. This model uses an open loop gain (Aol) of 100k, an infinite input impedance, and no supply rail voltage limits. The built-in LTspice op-amp model was used as an approximation. This model assumes infinite input impedance, fixed open-loop gain, and does not model output swing limits, slew rate, or rail clipping. Since expected signal amplitudes remain well below the intended hardware supply rails, this approximation is considered sufficient for preliminary frequency and transient analysis. Using this built-in op amp model greatly simplifies analysis as well as setup. 

## Brief System Description
- **Input Stage:** This is where input audio is fed in. This stage consists of ac coupling capacitors to remove unwanted DC offsets, current limiting resistors, a high pass filter to remove unwanted low frequency noise below the audible range, as well as a unity buffer to reduce loading effects.
- **Volume Control:** The volume stage consists of a simple potentiometer based voltage divider for passive volume control.
- **Equalizer:** This design implements a 3-band active equalizer using a Baxandall topology, allowing control over highs, mids, and lows.
- **Crossover:** The 2-way crossover allows audio to be split into the stereo and subwoofer channels based on frequency. This design implements a Linkwitz-Riley crossover using Sallen-Key topology for the filters.

## Schematics
Since both AC and Transient analysis are being performed 2 schematic layouts are prepared as shown below. SPICE directives and setup needed to run these simulations will be discussed later.

### Crossover Schematic for AC Analysis
For assessing the crossover functionality, a single channel can be analyzed, since both channels are identical. The voltage source is set to an AC signal with an amplitude of 1V.

![schematic]({{ 'assets/images/2026-05-15 crossover.jpg' | relative_url }})

### Full System Schematic for Transient Analysis
For assessing the system as a whole, all stages are cascaded as they would in the physical implementation. The input waveform comes directly from a 2-channel wav file, which LTspice normalizes to 1V amplitude centred at 0.

>**Note: LTspice does not have native potentiometer support; implementation of potentiometers is discussed later on.**

![schematic]({{ 'assets/images/2026-05-15 system schematic.png' | relative_url }})

# Frequency Response (AC Analysis)
The frequency response of the crossover is used to validate proper 4th-order Linkwitz-Riley behaviour. This is achieved by cascading two Butterworth filters of the same cutoff frequency, resulting in outputs that are each attenuated by −6 dB at the crossover frequency and remain in phase when summed. Another important confirmation is that the crossover has a roll-off rate of 24 dB per octave, meaning frequencies beyond the crossover point are attenuated very rapidly. The steep slope helps reduce overlap between drivers, improving frequency separation and reducing unwanted distortion or interference outside each speaker's intended operating range.

## Simulation Setup and SPICE Directives
To evaluate the frequency response of the crossover network, an AC sweep analysis was performed in LTspice. The simulation was configured to analyze how the filter stages attenuate and pass signals across the audible frequency spectrum. An AC sweep analysis tests the circuit at various frequencies to show behaviour at different frequencies.

The following SPICE directives were used:
 ```
 .ac dec 20 20 20k
 .lib opamp.sub
 ```

These directives configure an AC small-signal analysis with the following parameters:
- dec specifies a logarithmic sweep by decade.
- 20 indicates that 20 simulation points are calculated per decade.
- The sweep begins at 20 Hz, corresponding to the lower limit of typical human hearing.
- The sweep ends at 20 kHz, covering the upper range of the audible spectrum.
- .lib opamp.sub loads external operational amplifier models required by the circuit.

This setup allows the crossover's gain and attenuation characteristics to be observed across the full audio band.

## Results
Below is a frequency response plot of the crossover circuit, as well as a close up of the crossover frequency. From these plots, the crossover frequency is found to be 101Hz at an attenuation of -6.26dB. Furthermore, these plots display that the crossover has a steep roll-off rate of 24 dB per octave as intended.

![schematic]({{ 'assets/images/2026-05-17 crossover filters.png' | relative_url }})

![schematic]({{ 'assets/images/2026-05-17 crossover frequency.png' | relative_url }})

The other key characteristics of the crossover that are analyzed are ensuring a flat summed amplitude response across the passband. This can be done in LTspice by showing the summed waveforms of highs-out and lows-out. As seen in the 2 graphs below, there is a relatively flat response. There is a -226.12mdB attenuation at the crossover frequency, which is likely due to the differing cut-off frequencies between the high-pass and low-pass filters (read more on this from the [previous post](/2026-05-07-Sound-System-Crossover-Design.md#crossover-frequency-mismatch)). It is very unlikely that this difference is perceptible to the human ear under normal listening conditions.

![schematic]({{ 'assets/images/2026-05-17 sum.png' | relative_url }})

![schematic]({{ 'assets/images/2026-05-17 sum minimum.png' | relative_url }})

### Conclusions
The frequency response of the crossover circuit confirms that it behaves as intended. From the plots, the crossover frequency is identified at approximately 101 Hz, where both the high-pass and low-pass sections exhibit –6 dB attenuation. The response also demonstrates a steep 24 dB/octave roll-off, confirming that the circuit achieves the desired fourth-order filtering characteristics.

Further analysis of the summed output shows that the overall amplitude response remains effectively flat across the passband, indicating good phase and magnitude matching between the two filter sections. At the crossover frequency, a small deviation of approximately –226 mdB (–0.226 dB) is observed. This is extremely small and effectively negligible in practical audio terms.

Overall, the results confirm that the crossover is performing as a Linkwitz–Riley design should, with accurate frequency splitting and an essentially flat combined response across the operating band.

# Transient Audio Simulation
The transient simulation is used for assessing the entire system using “real” audio. This presents both “listenable result” as well as allowing waveforms at various nodes to be analyzed. This will also allow the effects of each stage to be closely analyzed. The main information gathered from this simulation is confirmation of equalizer functionality, overall distortion, and clipping through comparisons against input waveforms.

## Potentiometer Setup
Due to potentiometers not being natively supported by LTspice, they are implemented using simple voltage dividers. A real potentiometer is just a single resistive strip with a movable wiper that splits the total resistance into two sections, which exactly matches a voltage divider. Below is the volume potentiometer with the changeable parameter v:

![schematic]({{ 'assets/images/2026-05-19 pot.png' | relative_url }})

where v represents the wiper position and ranges from 0 to 1.

As v changes, resistance shifts between the upper and lower resistors while maintaining a constant total resistance of 10 kΩ. For example, when v = 0.5, both resistors become 5 kΩ, and the wiper is positioned at the midpoint. When v = 0.99, nearly all of the resistance is below the wiper, placing the output close to the upper terminal voltage.

This method provides a simple way to simulate adjustable controls in LTspice without requiring a dedicated potentiometer component.

## Simulation Setup and SPICE Directives
To simulate the circuit under a “real” input, LTspice's built-in wav file support is used. LTspice has the ability to import WAV files as waveforms, as well as export WAV files from waveforms.

To set up the input waveforms from a wav file, voltage sources V1 and V2 are set as follows:
```
V1 (Left channel) => wavefile="Daft-Punk-One-More-Time.wav" chan=1 
V2 (Right channel) => wavefile="Daft-Punk-One-More-Time.wav" chan=0
```

This allows a stereo WAV file to be split into left and right channels and be fed into the corresponding channels in the circuit as a voltage waveform.

For changing the input audio:
1. Place a WAV file in the same directory as the project.
2. Change the directives under V1 and V2 to the wav file name, but keep chan=1 and chan=0
3. Set the .tran to be longer than the length of the wav file.

```
.param b=0.5
.param m=0.5
.param t=0.5
 
.param v 0.99
 
.wave "l+r.wav" 16 44.1k V(Left-Out) V(Right-Out)
.wave "bass.wav" 16 44.1k V(Sub-Out) 
BLEFT  Lmix 0 V=V(Left-Out)+V(Sub-Out)
BRIGHT Rmix 0 V=V(Right-Out)+V(Sub-Out)

.wave "together.wav" 16 44.1k V(Lmix) V(Rmix)
 
.lib opamp.sub
.tran 10
```

These directives configure the simulation as follows:

- .param b, m, t defines adjustable parameters (between 0 and 1 exclusive) that can be referenced in the circuit to control the bass, mid, and treble, respectively, of the 3-band equalizer.
- .param v defines an adjustable parameter (between 0 and 1 exclusive) to control the master volume.
- .wave exports simulation node voltages directly into .wav audio files.
16 specifies 16-bit audio depth for standard PCM audio quality.
44.1k sets the sample rate to 44.1 kHz, matching CD-quality audio (lossless).
- BLEFT and BRIGHT are used to combine the left/right outputs with the subwoofer output into mixed audio channels, Lmix and Rmix. These are then combined as channels into “together.wav”.
- .lib opamp.sub loads external operational amplifier models required by the circuit.
- .tran 10 performs a transient simulation over 10 seconds to capture the circuit's time-domain response.

This setup allows both electrical verification of the complete audio system and subjective evaluation by generating playable audio files from the simulated outputs.

## Results
This section presents the audio outputs generated from the simulation, including the original input signal, the separated frequency bands (crossover), and the final reconstructed stereo mix. The files included are the original audio reference, the isolated subwoofer output, the processed left and right stereo channels, and the combined output file where all processed signals are summed together.

<div style="text-align: center;">

  <figure>
    <audio controls>
      <source src="assets/files/2026-05-19 Daft-Punk-One-More-Time.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Original Audio File</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="assets/files/2026-05-19 bass.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Subwoofer (Bass) Output</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="assets/files/2026-05-19 l+r.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Left + Right Stereo Channels</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="assets/files/2026-05-19 together.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Final Combined Output</figcaption>
  </figure>

</div>

Listening to these files demonstrates how the crossover network is functioning in practice. The original audio serves as a baseline for comparison, while the subwoofer file highlights how low-frequency content has been successfully extracted and isolated from the full signal. The stereo left and right outputs show how higher-frequency information is distributed across the two channels, maintaining spatial clarity and channel separation.

When the signals are recombined, the final output demonstrates that the system preserves the overall tonal balance of the original audio while applying frequency-based filtering.

The performance of the 3-band equalizer is evaluated by systematically varying the control parameters b (bass), m (mid), and t (treble) within the simulation. Each parameter is individually tested at two extreme values, 0.1 and 0.9, while all other parameters are held constant at 0.5. This approach isolates the effect of each frequency band, allowing the contribution of bass, midrange, and treble shaping to be clearly observed.

By setting a single parameter to 0.1, the corresponding frequency band is significantly attenuated, making it possible to observe how the circuit behaves under reduced gain conditions. Conversely, setting the parameter toward 0.9 increases the relative emphasis of that frequency band, showing how strongly the filter allows or boosts that portion of the spectrum. This contrast provides a clear visual and audible indication of each control's influence on the overall signal.

Repeating this process for all three parameters ensures that the response of each band can be independently verified. The resulting outputs are then compared to confirm that the equalizer behaves predictably.

Below are the audio samples demonstrating this behaviour:


<div style="text-align: center;">

  <figure>
    <audio controls>
      <source src="/assets/files/2026-05-19 b_0p1.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Bass (b = 0.1): reduced low-frequency output</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="/assets/files/2026-05-19 b_0p9.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Bass (b = 0.9): boosted low-frequency output</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="/assets/files/2026-05-19 m_0p1.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Mid (m = 0.1): reduced midrange output</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="/assets/files/2026-05-19 m_0p9.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Mid (m = 0.9): enhanced midrange output</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="/assets/files/2026-05-19 t_0p1.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Treble (t = 0.1): reduced high-frequency output</figcaption>
  </figure>

  <figure>
    <audio controls>
      <source src="/assets/files/2026-05-19 t_0p9.wav" type="audio/wav">
      Your browser does not support the audio element.
    </audio>
    <figcaption>Treble (t = 0.9): enhanced high-frequency output</figcaption>
  </figure>

</div>

Apart from sonic analysis, waveforms can also be interpreted to display system behaviour. below a section from the plot showing the left input against the left output summed with the subwoofer channel. This displays the lack of distortion as the waveform remains very similar to the original input, apart from the small phase shift. The phase shift does not affect the audio as the same shift is applied to all the output channels.

![schematic]({{ 'assets/images/2026-05-19 phase.png' | relative_url }})

The functionality of the volume potentiometer can also be verified in a similar manner. Below, the output waveform is plotted, where the green line represents parameter v set to 0.1 and the blue where v is set to 0.9. The change in amplitude, as well as the lack of distortion between the signals, displays proper functionality.

![schematic]({{ 'assets/images/2026-05-19 volume.png' | relative_url }})

### Conclusions
By running the transient analysis, the audio outputs generated from the simulations demonstrate that the system is functioning as intended. The crossover functionality is confirmed by the separation of frequency content into the subwoofer and stereo channels. The isolated bass track confirms effective extraction of low-frequency audio, and the left/right outputs preserve the higher-frequency stereo information.

When the output audio channels are recombined, the final output closely matches the original audio. This indicates that the system maintains overall tonal balance while applying frequency-based filtering and therefore does not affect perceived audio quality.

Overall, the simulation results provide strong evidence that the system performs as designed, both in measurable response and in audible output.

# Limitations of Simulation
While the LTspice simulation provides a useful validation of the design, several limitations should be acknowledged.

Firstly, the op-amp used in the model is an idealized LTspice model. Although it demonstrates key behaviours such as gain and frequency response of audio op-amps, it does not fully represent physical behaviour like input currents, slew-rate induced distortion, thermal drift, output current limits, or supply rail imperfections. In practice, these factors can introduce additional distortion or slight changes in the frequency response that are not visible in simulation.

Secondly, the simulation does not include PCB parasitics or wiring effects. Real circuits exhibit capacitance, trace inductance, and grounding impedance, all of which can slightly shift cutoff frequencies or alter filter Q factors, especially at higher frequencies.

# Summary 
Through the use of LTspice, the behaviour of the complete multi-stage audio system is validated before physical implementation. Two forms of analysis were performed: AC analysis to verify crossover performance and transient analysis to evaluate full-system audio behaviour using real music input.

The AC simulations confirms the intended performance of the 4th-order Linkwitz–Riley crossover, achieving a crossover frequency of approximately 101 Hz, –6 dB attenuation at crossover, and a 24 dB/octave roll-off and also maintaining an essentially flat summed response.

The transient simulation demonstrated correct operation of the complete signal path, including volume control, equalization, stereo routing, and subwoofer routing. Furthermore, WAV files and waveform comparisons showed that the processed output closely matches the original input while preserving tonal balance. No obvious clipping or severe waveform deformation was observed under the simulated conditions.

Although the simulations rely on idealized component models and do not capture all real-world effects, they provide strong confidence that the design will behave as expected when implemented in hardware.