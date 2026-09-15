---
layout: post
title: "Verilog Based I2S dac"
date: 2026-08-12
tags: [Project, Code Explanation]
---

# Background 
I²S (Inter-IC Sound) is a digital communication protocol commonly used to transfer PCM (Pulse-Code Modulation) audio data between devices such as microcontrollers, DSPs, and audio DACs. It typically uses three signals: Bit Clock (BCK), Word Select (WS/LRCLK), and Serial Data (SD). BCK provides the timing for each individual data bit, while WS indicates whether the transmitted sample belongs to the left or right audio channel. The audio samples are transmitted serially over the SD line, with each sample represented by a fixed number of bits, such as 16, 24, or 32 bits. standard I²S supports a maximum word length or slot width of 32 bits per channel, smaller sample sizes are padded with zeroes. For example, with 16-bit stereo audio, 16 bits of data followed by 16 null bits are transmitted for the left channel followed by 16 bits of data followed by 16 null bits for the right channel. Totaling 32 bits per channel. The I²S transmitter shifts each sample out one bit at a time, synchronized to BCK, while the receiving device uses BCK and WS to reconstruct the individual left and right audio samples.

Below is a timing diagram from Philips Semiconductors I²S specifications sheet. Note that BCK is referenced as SCK:

![1 ]({{ 'assets/images/2026-09-13 1.png' | relative_url }})

PCM is a way of converting an analog audio signal, into digital data. The process begins by preparing discrete data from a sampled analog waveform at regular intervals, with the sampling rate determining how many measurements are taken per second (for example, 44.1 kHz means 44,100 samples per second). The number of bits used for each sample determines the resolution of the audio. For example, 16-bit PCM provides 2^16=65,536 possible amplitude levels. Finally, these values are represented as binary numbers and transmitted as digital audio. During playback, a DAC converts the PCM samples back into an analog voltage waveform. 

# Code
This verilog module is a customizable I²S transmitter that converts left/right audio samples into a serial I²S data stream. It generates the required BCK and LRCK signals from any given system clock using a fractional phase-accumulator divider, while providing ready pulses to flag the loading of new audio samples.

Find the github repo [here](https://github.com/saper6569/Verilog-I2S-Transmitter).

```verilog
module I2S_Transmitter
#(	parameter WORD_SIZE = 16,
	parameter SAMPLE_RATE = 44_100,
	parameter CLOCK_FREQ = 50_000_000,
	parameter ACC_WIDTH = 32
)
(
	input wire clk,
	input wire reset_n,
	input wire [WORD_SIZE-1:0] input_word_left,
	input wire [WORD_SIZE-1:0] input_word_right,
	output reg BCK = 0,
	output reg DOUT = 0,
	output reg left_ready = 0, 
	output reg right_ready = 0 
);
	localparam BITS_PER_CHANNEL = 32;
	localparam integer BCK_TOGGLE_FREQ = 4 * BITS_PER_CHANNEL * SAMPLE_RATE;
	localparam [63:0] PHASE_STEP_NUM = (64'd1 * BCK_TOGGLE_FREQ) << ACC_WIDTH;
	localparam [ACC_WIDTH-1:0] PHASE_STEP = PHASE_STEP_NUM / CLOCK_FREQ;

	initial begin
		if (BCK_TOGGLE_FREQ >= CLOCK_FREQ) begin
			$error("CLOCK_FREQ (%0d) must be greater than 4*BITS_PER_CHANNEL*SAMPLE_RATE (%0d)", CLOCK_FREQ, BCK_TOGGLE_FREQ);
		end
		if (WORD_SIZE > BITS_PER_CHANNEL) begin
			$error("WORD_SIZE (%0d) must not exceed BITS_PER_CHANNEL (%0d)", WORD_SIZE, BITS_PER_CHANNEL);
		end
		if (PHASE_STEP == 0) begin
			$error("PHASE_STEP resolved to 0 -- BCK will never toggle. Check CLOCK_FREQ/SAMPLE_RATE/ACC_WIDTH.");
		end
	end

	reg [WORD_SIZE-1:0] LEFT_WORD = 0;
	reg [WORD_SIZE-1:0] RIGHT_WORD = 0;
	reg [ACC_WIDTH-1:0] phase_acc = 0;
	reg [$clog2(BITS_PER_CHANNEL)-1:0] bit_counter = 0; 
	wire [ACC_WIDTH:0] phase_acc_next = phase_acc + PHASE_STEP;
	wire BCK_en = phase_acc_next[ACC_WIDTH];

	always @(posedge clk) begin
		if (!reset_n) begin
			phase_acc <= 0;
		end else begin
			phase_acc <= phase_acc_next[ACC_WIDTH-1:0];
		end
	end

	always @(posedge clk) begin
		if (!reset_n) begin
			BCK <= 0;
		end else if (BCK_en) begin
			BCK <= ~BCK;
		end
	end

	wire bit_tick = BCK_en && BCK;

	always @(posedge clk ) begin
		left_ready <= 1'b0;
		right_ready <= 1'b0;

		if (!reset_n) begin
			LRCK <= 0;
			DOUT <= 0;
			left_ready  <= 0;
			right_ready <= 0;
			bit_counter <= 0;

		end else if (bit_tick) begin
			if (LRCK) begin
				DOUT <= RIGHT_WORD[WORD_SIZE-1];
				RIGHT_WORD <= RIGHT_WORD << 1;
			end else begin
				DOUT <= LEFT_WORD[WORD_SIZE-1];
				LEFT_WORD <= LEFT_WORD << 1;
			end
			if (bit_counter >= (BITS_PER_CHANNEL - 1)) begin
				bit_counter <= 0;
				LRCK <= ~LRCK;
				if (LRCK) begin
					LEFT_WORD <= input_word_left;
					left_ready <= 1'b1;
				end else begin
					RIGHT_WORD <= input_word_right;
					right_ready <= 1'b1;
				end
			end else begin
				bit_counter <= bit_counter + 1;
			end
		end
	end
endmodule
```

## Customizable Module Parameters:
These values can be overridden on instantiation of the module to fit the needs of the use case. 

- WORD_SIZE
  - Controls the number of bits in each input audio sample.
  - Default: 16 bits.
  - Example: Setting WORD_SIZE = 24 allows 24-bit audio samples.

- SAMPLE_RATE
  - Controls the audio sample rate in samples per second.
  - Default: 44,100 Hz.
  - Example: SAMPLE_RATE = 48_000 produces a 48 kHz audio stream.

- CLOCK_FREQ
  - Specifies the frequency of the FPGA/system clock driving the module.
  - Used by the fractional clock divider to generate the required BCK frequency.
  - Default: 50 MHz.

- ACC_WIDTH
  - Controls the width of the phase accumulator used by the fractional clock divider.
  - A larger value provides finer frequency resolution and allows the desired 
    BCK frequency to be represented more accurately.
  - Default: 32 bits.

## Module Input/Output

These variables are exposed to the top level to allow them to be accessed and/or modified from outside the module.

- clk: input
  - Input wire connecting the top-level clock (FPGA clock) to the internal clock controlling the I²S module.
  - The default FPGA clock is 50 MHz.
- reset_n: input
  - Active-low reset signal that allows the module to be fully reset by the top level.
  - When reset_n is low, the internal clock-generation and data-transmission logic is returned to its initial state.
- input_word_left: input
  - Parallel audio data input for the left channel.
  - The width of this signal is determined by the WORD_SIZE parameter, which is 16 bits by default.
  - The sample is loaded into an internal shift register before being transmitted serially through DOUT.
- input_word_right: input
  - Parallel audio data input for the right channel.
  - Like input_word_left, its width is determined by WORD_SIZE.
  - The sample is loaded into an internal shift register and transmitted one bit at a time when the right channel is selected by LRCK.
- BCK: output
  - Bit clock generated by the I²S transmitter.
  - Controls the timing of individual bits transmitted through DOUT.
  - For the default configuration, the BCK frequency is approximately 2.8224 MHz, corresponding to a 44.1 kHz sample rate and 32-bit slots for each of the two channels.
- LRCK: output
  - Left/right clock used to indicate which audio channel is currently being transmitted.
  - A low LRCK state corresponds to the left channel, while a high state corresponds to the right channel.
  - The signal changes state after every 32 BCK cycles, resulting in one complete stereo frame every 64 BCK cycles.
- DOUT: output
  - Serial audio data output containing the individual bits of the left and right audio samples.
  - Audio samples are transmitted most-significant bit (MSB) first.
  -  The output is synchronized to the I²S bit-clock timing.
- left_ready: output
  - One-clock-cycle status pulse indicating that a new left-channel sample has been loaded into the internal left-channel shift register.
  - Can be used by the top-level logic to determine when the transmitter is ready for a new left-channel sample.
- right_ready: output
  - One-clock-cycle status pulse indicating that a new right-channel sample has been loaded into the internal right-channel shift register.
  - Provides the top-level logic with an indication that the right-channel is ready for a new sample.

## Clock Generation and Phase Accumulator

The FPGA operates using a system clock (default 50 MHz), while the I²S interface requires a much lower bit clock (BCK). Since the required I²S clock frequency is not always an integer division of the FPGA clock, a phase accumulator is used instead of a conventional integer clock divider. This allows the transmitter to generate a more accurate average clock frequency.

### Required I²S Clock Frequency

The transmitter uses 32 bits for each channel, giving a total of 64 bits per stereo audio frame. With a sample rate of 44.1 kHz, the required BCK frequency is:

$$
f_{BCK} = 2 \times 32 \times 44,100
$$

$$
f_{BCK} = 2.8224\text{ MHz}
$$

The code defines the frequency at which BCK must toggle as:

```verilog 
localparam integer BCK_TOGGLE_FREQ = 4 * BITS_PER_CHANNEL * SAMPLE_RATE;
```

The factor of four is used because BCK is toggled on each overflow of the phase accumulator. Therefore, the calculated toggle frequency is:

$$
f_{toggle}=4\times32\times44,100=5.6448\text{ MHz}
$$

Since two toggles are required to produce one complete BCK period, this corresponds to the required 2.8224 MHz BCK frequency.

### Phase Accumulator

Much of the phase accumulator was generated using AI as this is not something I have much knowledge on. However I will try my best to provide an explanation on how it works, but [here](https://www.digikey.ca/en/articles/the-basics-of-direct-digital-synthesizers-ddss) is a good resource I found when trying to figure out how they work. 

The phase accumulator is implemented using the following registers and signals:

```verilog
reg [ACC_WIDTH-1:0] phase_acc = 0;

wire [ACC_WIDTH:0] phase_acc_next = phase_acc + PHASE_STEP;

wire BCK_en = phase_acc_next[ACC_WIDTH];
```
phase_acc stores the current phase of the generated clock. On every rising edge of the system clock, PHASE_STEP is added to the accumulator:
```verilog
phase_acc <= phase_acc_next[ACC_WIDTH-1:0];
```
The accumulator has an additional bit in phase_acc_next. When the addition exceeds the maximum value that can be represented by the ACC_WIDTH-bit accumulator, this extra bit becomes 1. This overflow is detected by BCK_en.

The phase step is calculated using:
```verilog
localparam [63:0] PHASE_STEP_NUM = (64'd1 * BCK_TOGGLE_FREQ) << ACC_WIDTH;

localparam [ACC_WIDTH-1:0] PHASE_STEP = PHASE_STEP_NUM / CLOCK_FREQ;
```

This effectively implements the relationship:

$$
PHASE STEP =
\frac{f_{toggle}}{f_{clk}}2^{ACCWIDTH}
$$

For the default parameters:

$$
PHASESTEP =
\frac{5.6448\text{ MHz}}{50\text{ MHz}}
\times2^{32}
$$

The resulting phase step determines how quickly the accumulator progresses toward overflow.

### Generating BCK

Once the phase accumulator overflows, BCK_en becomes active and the BCK signal is toggled:

```verilog
else if (BCK_en) begin
    BCK <= ~BCK;
end  
```

The phase accumulator therefore acts as a frequency divider with fractional resolution. Unlike an integer divider, it does not require the 50 MHz clock to be an exact multiple of the desired BCK frequency. Instead, small variations in the number of FPGA clock cycles between BCK transitions are distributed over time, producing the desired average frequency.

For the default configuration, the generated BCK is approximately 2.8224 MHz, which is appropriate for 44.1 kHz audio with 32-bit slots per channel.

### Why a Phase Accumulator Is Used

A conventional clock divider would require an integer value such as:

$$
N=\frac{50\text{ MHz}}{2.8224\text{ MHz}}
$$

However, this result is not an integer. Rounding it to an integer would introduce a significant frequency error. The phase accumulator avoids this limitation by representing the fractional portion of the required division using the additional resolution provided by ACC_WIDTH.

Increasing ACC_WIDTH increases the frequency resolution of the phase accumulator. A larger accumulator allows PHASE_STEP to represent smaller fractional frequency changes, resulting in a more precise average output frequency.

## How to Use the Module
- 	This I²S module is used by connecting the system clock and left/right 
audio sample inputs to the corresponding input ports and connecting BCK, 
LRCK, and DOUT to the I²S-compatible audio device. 
- 	The module should be instantiated with parameters matching the desired 
audio format and the FPGA system clock frequency.
- 	input_word_left and input_word_right inputs should contain the next left 
and right audio samples, respectively. **Initialize as 0 in top level to 
prevent artifacts on startup.
- 	left_ready and right_ready signals indicate when the module is ready to 
load a new sample, top level logic should update the input when the 
corresponding ready signal is flagged. 
- 	The module automatically serializes the samples MSB-first through DOUT, 
generates the required BCK, and alternates LRCK between the left and 
right channels. 
- 	The generated BCK, LRCK, and DOUT signals can then be connected directly 
to the corresponding I²S inputs of an audio DAC, codec or amplifier.

# Example Top Level
The example code below instantiates the transmitter with the default 16-bit, 44.1 kHz format and generates a 440 Hz square wave on both channels. It updates the sample generator when left_ready pulses and exposes BCK, LRCK, and DOUT at the top level for hardware connection.

The example is tested on a PCM5102 and can be used as a starting point for an FPGA project. Add both Verilog source files to the project, connect clk and reset_n, and assign the three I2S outputs to the FPGA pins connected to the audio device.

```verilog
module TopLevel_SquareTest #(
    parameter WORD_SIZE = 16,
    parameter SAMPLE_RATE = 44_100,
    parameter CLOCK_FREQ = 50_000_000,
    parameter ACC_WIDTH = 32,

    parameter FREQUENCY = 440
)(
    input wire clk,
    input wire reset_n,

    output wire BCK,
    output wire LRCK,
    output wire DOUT
);

    reg [WORD_SIZE-1:0] input_word_left;
    reg [WORD_SIZE-1:0] input_word_right;
    wire left_ready;
    wire right_ready;

    reg [ACC_WIDTH-1:0] phase_accumulator;
    localparam [ACC_WIDTH-1:0] PHASE_INCREMENT = (FREQUENCY * (64'd1 << ACC_WIDTH)) / SAMPLE_RATE;

    I2S_Transmitter #(
        .WORD_SIZE(WORD_SIZE),
        .SAMPLE_RATE(SAMPLE_RATE),
        .CLOCK_FREQ(CLOCK_FREQ),
        .ACC_WIDTH(ACC_WIDTH)
    ) i2s_tx (
        .clk(clk),
        .reset_n(reset_n),
        .input_word_left(input_word_left),
        .input_word_right(input_word_right),

        .BCK(BCK),
        .LRCK(LRCK),
        .DOUT(DOUT),

        .left_ready(left_ready),
        .right_ready(right_ready)
    );

    always @(posedge clk) begin
        if (!reset_n) begin
            phase_accumulator <= {ACC_WIDTH{1'b0}};
            input_word_left <= 16'h7FFF;
            input_word_right <= 16'h7FFF;
        end
        else begin
            if (left_ready) begin
                phase_accumulator <= phase_accumulator + PHASE_INCREMENT;
                if (phase_accumulator[ACC_WIDTH-1] == 1'b0) begin
                    input_word_left <= 16'h7FFF;
                    input_word_right <= 16'h7FFF;
                end
                else begin
                    input_word_left <= 16'h8000;
                    input_word_right <= 16'h8000;
                end
            end
        end
    end
endmodule
```