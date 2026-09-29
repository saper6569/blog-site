---
layout: post
author: "Sanija Perera"
github: "https://github.com/saper6569"
title: "Jane-Street Asic Memory and Programmer" 
date: 2026-09-28
tags: [Asic, Verilog]
---

# Challenge:
The challenge is to design a small, open-source ASIC that acts as a programmable protocol emulator. This is a tiny specialized processor whose job is to precisely control and sample physical I/O pins so that communication protocols can be implemented in software/firmware rather than being hard-coded as separate UART, SPI, I²C, etc. peripherals. The important part is RE-programmability: not want a chip that simply contains a UART block, SPI block, and I²C block; instead, the architecture should provide a small set of primitives-such as reading/writing pins, shifting data, counting cycles, and controlling timing-that can be combined to implement different protocols after the chip has been fabricated. 

The project will be implemented as a Tiny Tapeout ASIC with a max size of 6x4 tiles (24 total), with the protocol-emulation processor fabricated as a small custom chip. An RP2040 development board will be used as the external controller and test platform, providing power, programming, and communication with the ASIC. The RP2040 can load protocol instructions into the ASIC, configure its GPIO pins, and send or receive data while the ASIC handles the timing-critical protocol operations. This setup allows the fabricated chip to be tested with real communication protocols and demonstrates how the custom processor can operate as a standalone hardware protocol emulator.

# Introduction
From the previous post on this project, one of the main recognized challenges of the design is implementing instruction memory onboard the silicon die. This plays a particularly large issue as memory can be area hungry in a design that is heavily area constrained. Another topic that will be discussed here is implementing instruction memory programming capabilities. The custom hardware has to be designed to allow the RP2040 to write instructions to the instruction memory while also being as simplified as possible to abide to area restrictions.  

# Memory
As discussed before the main option for memory is SRAM. SRAM stands for static random access memory. SRAM is a fast volatile memory. The volatility of SRAM allows data to be written, erased and changed, which is a requirement for the instruction memory. Memory also has to be word addressable to allow individual instructions to be accessed and or written to. Since each instruction has been decided to be stored as 1 byte (8 bits), the memory structure has to be byte addressable. 

## implementation
Below is a verilog based module for instruction memory. It contains 2^8 (256 total) possible addresses, holding 8 bits each, by default. This implementation has a synchronous read/write where w_en can be asserted to enable writing and disable reading. The memory includes a parallel input and output port for reading and writing respectively, each with enough parallel bits to hold an entire instruction. 

```verilog
module instruction_memory #(
    parameter ADDR_WIDTH = 8, // 2^8 = 256 instructions
    parameter DATA_WIDTH = 8  // 2-bit opcode + 6-bit data
)(
    input wire clk,
    input wire w_en,

    input wire [DATA_WIDTH-1:0] data_in,
    input wire [ADDR_WIDTH-1:0] addr,

    output reg [DATA_WIDTH-1:0] data
);
    reg [DATA_WIDTH-1:0] memory [0:(1 << ADDR_WIDTH)-1];
    // Synchronous read/write memory
    always @(posedge clk) begin
        if (w_en) begin
            memory[addr] <= data_in;
        end
        else begin
            data <= memory[addr];
        end
    end
endmodule
```

## Macro Based implementation
While the above implementation provides the necessary functionality required by the instruction memory, macros can be used to create a more compact design. An ASIC macro is a pre-designed and pre-optimized hardware block that can be used a chip design. Instead of synthesizing every component from basic logic gates, a macro provides a ready-made implementation of a specific function, for this case SRAM. 

The final design is likely going the macro based route. Tiny Tapeout has resources on implementing memory [here](https://tinytapeout.com/specs/memory/). Tiny Tapeout has data on SRAM macros for IHP implementations, however it lacks in depth assistance on designing for SkyWater's 130nm open-source process design kit (PDK) which is what this project opted to use as it has widely more documentation. 

Due to a lack of a premade SkyWater SRAM configuration of 256x8 in its standard factory IP library (which defaults to sizes like 8×1024, 32×256, and 32×512), this project will utilize [OpenRAM](https://openram.org/), an open-source memory compiler that fully supports the SkyWater 130nm process. This software will allow generation of a custom 256x8 block using the compiler.

# Instruction Memory Programming
With the instruction memory completed, a module is required to write instructions to the memory from the external RP2040 controller. To minimize the number of connections required between the RP2040 and the ASIC, a simple 2-wire programming interface is used. One wire carries the programming data, while the second wire acts as a program enable signal. The RP2040 sends the address and instruction serially over the program data line while program_en is asserted. This is possible since the clock is also provided from the RP2040, meaning that both the asic and the RP2040 are clock synchronized. 

## implementation
The programmer receives a complete address and instruction as a serial stream of bits. These bits are shifted into a shift register while program_en is high. A counter tracks the number of received bits to ensure that a complete address and instruction have been transmitted. Once program_en is driven low, the programmer checks that the expected number of bits was received. If the transfer is complete, the received address and instruction are stored in a data register and write_en is asserted for one clock cycle. This write enable signal is connected directly to the write enable input of the instruction memory.

The address and instruction are transmitted as a single word, with the address occupying the most significant bits and the instruction occupying the least significant bits. With the default parameters, both the address and instruction are 8 bits, resulting in a 16-bit word being transmitted for each programming operation.

Below is the Verilog implementation of the programming interface.

```verilog
module programmer #(
    parameter INSTRUCTION_WIDTH = 8,
    parameter ADDRESS_WIDTH = 8
)(
    input wire clk,
    input wire rst_n,

    // 2-wire programming interface
    input wire program_data,
    input wire program_en,

    // Interface to instruction memory
    output wire [ADDRESS_WIDTH-1:0] address_out,
    output wire [INSTRUCTION_WIDTH-1:0] instruction_out,
    output reg write_en
);

    localparam TOTAL_WIDTH = ADDRESS_WIDTH + INSTRUCTION_WIDTH;
    localparam COUNT_WIDTH = $clog2(TOTAL_WIDTH + 1);

    // Shift register
    reg [TOTAL_WIDTH-1:0] shift_reg;
    // Holds the completed address + instruction
    reg [TOTAL_WIDTH-1:0] data_reg;
    // Counts the number of received bits
    reg [COUNT_WIDTH-1:0] bit_count;
    // Previous cycle's program enable
    reg prev_program_en;

    // Address and instruction outputs
    assign address_out = data_reg[TOTAL_WIDTH-1:INSTRUCTION_WIDTH];
    assign instruction_out = data_reg[INSTRUCTION_WIDTH-1:0];

    always @(posedge clk) begin
        if (!rst_n) begin
            shift_reg <= {TOTAL_WIDTH{1'b0}};
            data_reg <= {TOTAL_WIDTH{1'b0}};
            bit_count <= {COUNT_WIDTH{1'b0}};
            prev_program_en <= 1'b0;
            write_en <= 1'b0;
        end
        else begin
            // write enable is only active for one cycle
            write_en <= 1'b0;
            // Remember previous program_en state
            prev_program_en <= program_en;
            // Receive programming data
            if (program_en) begin
                // Shift in one bit
                if (bit_count < TOTAL_WIDTH) begin
                    shift_reg <= {
                        shift_reg[TOTAL_WIDTH-2:0],
                        program_data
                    };
                    bit_count <= bit_count + 1'b1;
                end
            end

            // program_en has just gone low
            if (prev_program_en && !program_en) begin
                // Check that exactly one complete word was received
                if (bit_count == TOTAL_WIDTH) begin
                    data_reg <= shift_reg;
                    write_en <= 1'b1;
                end
                // Reset counter for next word to be programmed
                bit_count <= {COUNT_WIDTH{1'b0}};
            end
        end
    end
endmodule
```

