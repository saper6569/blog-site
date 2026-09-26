---
layout: post
author: "Sanija Perera"
github: "https://github.com/saper6569"
title: "Jane-Street Asic Architecture Planning"
date: 2026-09-25
tags: [Asic, Verilog]
---

# Challenge:
The challenge is to design a small, open-source ASIC that acts as a programmable protocol emulator. This is a tiny specialized processor whose job is to precisely control and sample physical I/O pins so that communication protocols can be implemented in software/firmware rather than being hard-coded as separate UART, SPI, I²C, etc. peripherals. The important part is RE-programmability: not want a chip that simply contains a UART block, SPI block, and I²C block; instead, the architecture should provide a small set of primitives-such as reading/writing pins, shifting data, counting cycles, and controlling timing-that can be combined to implement different protocols after the chip has been fabricated. 

The project will be implemented as a Tiny Tapeout ASIC with a max size of 6x4 tiles (24 total), with the protocol-emulation processor fabricated as a small custom chip. An RP2040 development board will be used as the external controller and test platform, providing power, programming, and communication with the ASIC. The RP2040 can load protocol instructions into the ASIC, configure its GPIO pins, and send or receive data while the ASIC handles the timing-critical protocol operations. This setup allows the fabricated chip to be tested with real communication protocols and demonstrates how the custom processor can operate as a standalone hardware protocol emulator.

# Planning
The thought process behind planning the architecture is to minimize a traditional processor to the point where it can minimally perform the required tasks. When thinking of minimizing hardware from the traditional architecture some of the main things that should be addressed are the instructions set, ALU, instruction memory as well as internal register count. By redesigning certain parts with the idea of minimizing hardware a solution that can allow synthesis under the strict size restrictions is possible. 

## Memory
The challenge requires the design to be located entirely on the 24 total tiles. A major implication of this is that instruction memory must be implemented onboard the silicon die. This will likely require a large portion of the die, and also highly depends on the memory type tat is implemented. Currently the main option is SRAM. Instruction memory will also highly depend on the bits used for storing single instruction as well as the amount of instructions that the programmer will be allowed to use.

## Instruction Set
As stated above the instruction set Looking at the task of being able to emulate any communication protocol using instructions, a minimal processor would require the following capabilities: reading a value from an io pin, writing a value to an io pin, polling an io pin, waiting for an amount of time and the ability to loop instructions. This gives 6 mandatory functions that have to be accommodated for. The first optimizations focused on reducing the instruction set further. If the instruction set could be reduced to 4 instructions it could be encoded using 2 bits. 

To reduce the number of instructions first instructions were grouped together to remove any possible redundancy. The main grouping was including the read and write as a single instruction. SHIFT can be used for shifting data from the SHIFT_REG register to an output pin or from an output pin to the SHIFT_REG. To allow both directions of shifting to be handled with the single instruction an internal register REF_BIT can be toggled using TOGGLE_BIT. Since REF_BIT is initialized to a known value the programmer can keep track of it and create replicable function. Now using two instructions both reading and writing are possible. Next is the polling an io pin and waiting for an amount of time. The polling io pins is slightly more complicated to simplify, as it technically encloses waiting for a pin to be either a 1 or a 0. This is 2 instructions. However using the previously mentioned REF_BIT which can be toggled by TOGGLE_BIT, the two instructions can be merged into one WAIT instruction where the REF_BIT decides whether to wait for a 1 or 0. Now there are 3 instructions. The next instruction is a wait for a certain amount of time instruction. This can also be implemented (WAITN), bringing the instruction count to 4. The final functionality that should be implemented is the ability to loop instructions. This is done without the use of an extra instruction. Instead it is done by overflowing the PC down to 0 at the end of the instruction memory. This essentially makes the instructions loop infinitely. This has the added functionality of allowing WAITN 0 to essentially do nothing. By having the instruction memory initialized to WAITN 0 instruction the code will automatically progress toward the PC overflow. However if the programmer decides that they don't want infinite looping they can toggle the BREAK_BIT which will stop the PC from getting incremented and therefore stop te code. This allows all the functionality to be fully implemented using only 4 instructions. 

One of the best optimizations outcomes of the reduced instruction set is the ability to remove the ALU which is prominent in traditional processors.

## GPIO Output Modes: Push-Pull vs Open-Drain
A key requirement for the protocol emulator is supporting different electrical signaling requirements between communication protocols. In particular, I²C requires an open-drain output configuration rather than the standard push-pull configuration used by protocols such as SPI. A push-pull output can actively drive a signal both high and low, while an open-drain output can actively pull the signal low but releases the line when a high value is required. An external pull-up resistor then brings the line high when no device is pulling it low.

To accommodate this without adding a dedicated I²C peripheral, each GPIO will have a configurable output mode controlled by the GPIO_MODE register. In push-pull mode, writing a 1 drives the pin high and writing a 0 drives it low. In open-drain mode, writing a 0 drives the pin low, while writing a 1 places the output driver into a high-impedance state, allowing the external pull-up resistor to bring the line high. The input buffer remains available so that the processor can read the actual state of the bus. This allows the same GPIO hardware and instruction set to implement both conventional push-pull protocols and open-drain protocols such as I²C. The TOGGLE_BIT instruction can modify the GPIO_MODE bits during initialization or operation, eliminating the need for a separate instruction dedicated to changing GPIO direction or output mode.

# Special Registers
- SHIFT_REG - holds the data to be shifted out or in
- REF_BIT - controls direction of data for a SHIFT (in or out), or whether WAIT_EVENT polls for a high or low input (known beginning state)
- BREAK_BIT - controls whether PC will be incremented
- HIGH_BIT - always contains 1, can be shifted in to write to the SHIFT_REG
- LOW_BIT - always contains 0, can be shifted in to write to the SHIFT_REG
- COUNTER - counter for the wait instruction
- PC - program counter for instructions (reset to 0 at the end of the code)
- GPIO_MODE - labels push-pull/open-drain per pin (known beginning state)
- GENERAL_PURPOSE_BIT - allows the programmer to use toggle and shift to output a specific value (known beginning state)

# Custom Instruction set (4 instructions 2 op code bits): 
- (OP) LABLE      Argument : Description
- (00) WAIT       N        : wait N cycles
- (01) TOGGLE_BIT Address  : flip The bit at the given address 
- (10) SHIFT      Address  : shift in or out depending on ref bit 
- (11) WAIT_EVENT Adress   : wait until high or low from input depending on ref bit

WAIT for N=000...., is a do nothing instruction, and simplifies the loop based instruction procedure by allowing instruction storage to be intialized to 00.... where on this instruction will increment PC towards overflow.

## TOGGLE_BIT
TOGGLE_BIT has some special properties that allow it to reduce instructions as well as hardware size. The main property is that it provides a mode of completing a "write" operations without an extra instruction. The best way that a write can be completed is by using the GENERAL_PURPOSE_BIT to toggle to the value that is wanted, and then use a SHIFT call paired with the required REF_BIT value to move the bit to wherever it is needed. Since this bit has a known starting value it can be easily used for replicable results.

Another key usage is for toggling the GPIO_MODE parameter. GPIO_MODE contains a bit for every configurable io port. It is used for selecting between push-pull or open-drain. This means that setup can also be done directly using the instruction set stated above. This functionality is to accommodate for communication protocols such as I²C where they require open drain for both its data line (SDA) and clock line (SCL).

# Custom Hardware Architecture:
- Hard coded uart for programming instructions and shift register
- Loop based processor for allowing repititon (allow overflow of PC)
- Limited onboard instruction memory (memory type not yet decided)
- Custom TOGGLE_BIT instruction allows any bit in the following locations to be manipulated: REF_BIT, BREAK_BIT, GPIO_MODE, GENERAL_PURPOSE_BIT
- Counter for WAIT instruction
- io port hardware for bidirectional communication

# Custom Instruction set (2 instructions 1 op code bits): <- likely will not be used
The instruction set can be reduced to only two instructions by using the principle of state-controlled instruction behavior. Instead of assigning a separate opcode to each operation, a single-bit instruction can perform different operations depending on the values of configurable control bits.

## The instruction set consists of:
- (0) TOGGLE_BIT   Address  : flips the bit at the specified address.
- (1) DO_SOMETHING Value    : performs an operation determined by the current configuration of the processor's control bits.

The TOGGLE_BIT instruction provides a general mechanism for modifying the processor's state. By toggling different control bits, the programmer can configure what the DO_SOMETHING instruction does. This effectively allows multiple operations to be represented using a single instruction opcode.

For example, dedicated control bits could determine whether DO_SOMETHING performs a shift, waits for a specified number of cycles, waits for an input event, or performs another required operation. Additional toggleable bits can be introduced to provide more possible behaviors without increasing the instruction width.

This approach trades additional state bits and decoding logic for a reduction in instruction-memory size. Because the instruction memory is stored on-chip, reducing the instruction width can provide a significant area savings. The tradeoff is that operations may require multiple instructions: a TOGGLE_BIT instruction configures the desired behavior, followed by DO_SOMETHING to execute it.

The architecture can therefore be viewed as a small programmable state machine, where TOGGLE_BIT modifies the state of the processor and DO_SOMETHING acts on that state. This allows the same instruction to implement different communication-protocol operations while maintaining an extremely small instruction encoding.

Due to the tradeoff of requiring more instructions to complete the same action this instruction set is likely not as advantageous as it sounds, and therefore will likely not be implemented. 
