# 1-Bitsy The SAP CPU

## Files in this Project

- EAGLE Schematic and PCB files
- PDF of the Schematic
- Logisim-Evolution simulation 

## Description 
After reading the book *CPUの創りかた* by Kaoru Watanabe, which describes the creation of a 4 bit CPU called the TD-4. I decided to create a CPU of my own.  I didn't want to spend weeks to months slaving over logism before making a PCB, so I gave myself the goal to make the simplest possible thing that could be called a CPU.  It had to be able to do the following:

- Sequence.  Progress from one instruction to another.
- Selection. Change the course of execution based on some criteria.
- Iteration. Change the course of execution.

There is no ALU at all, no data memory, and no internal registers.  It is just able to give predetermined outputs to some external input.  However the minimum viable system consists of three chips:

- 2 74*161 4-bit counter with clear
- 1 74*02 quad 2 input nor gate

This system allows for addressing 16 bytes of read only 8-bit memory.  Not much (if anything) practical can be done with these constraints, but it does technically meet all the criteria.  As we all know being technically correct is the best kind of correct.

**Personal Note:** I struggled a little to decide the "bitness" of the CPU as that is either based on the ALU width or the data bus width.  The 1Bitsy has no ALU, uses a 8 bit instruction word, 4-bit output, and has only a single bit input.  So it could be seen as a 0, 1, 4, or 8 bit system.  One bit just felt the most intellectually honest


### Expandability 

It would be fairly easy to expand the system to be able to address 256 bytes of 12-bit read only memory by adding addition 74*161s and tweaking the decoding logic.  This is left as an intellectual exercise for the reader

Additionally there is currently an unused bit in the instruction nibble which could be used to add additional instructions.  For example if Bit 5 of the Instruction byte were named `I` and tied to the `ENT` and `ENP` pins of the output register, then every instruction would have a variant that would increment the output register's value:

| `76543210` | `ASM` | Description                               |
| ---------- | ----- | -----------------------------             |     
| `**10VVVV` | `OUT` | Output `VVVV`                             | 
| `**00VVVV` | `OTI` | Increment & Output `VVVV`                 | 
| `0011****` | `NOP` | No Operation                              | 
| `0001****` | `INC` | Increment Output                          | 
| `0101VVVV` | `JMP` | Increment & `PC <- VVVV` If Input Bit Set | 
| `0111VVVV` | `JMP` | `PC <- VVVV` If Input Bit Set             | 
| `1*01VVVV` | `JST` | `PC <- VVVV`                              | 
| `1*11VVVV` | `JST` | Increment &`PC <- VVVV`                   | 


## The System

### I/O

Output consists of 4 output bits (`Out0`...`Out3`), there is a single input bit (`In`).

### Instruction Byte

| Bit(s)   | Name   | Description |
| -------- | -----  | ----------- |
| Bit  7   | `P`    | PC Load     |
| Bit  6   | `C`    | Condition   |
| Bit  5   |        | Unused      |
| Bit  4   | `O`    | Output      |
| Bits 3-0 | `VVVV` | Value       |


### Instructions

| `76543210` | `ASM` | Description                   |
| ---------- | ----- | ----------------------------- |     
| `***0VVVV` | `OUT` | Output `VVVV`                 | 
| `00*1****` | `NOP` | No Operation                  | 
| `01*1VVVV` | `JMP` | `PC <- VVVV` If Input Bit Set | 
| `1**1VVVV` | `JST` | `PC <- VVVV`                  | 

There are two additional "phantom instructions" of dubious utility:

- `01*0VVVV` - `JMO`: Output `VVVV` and Jump to `VVVV`
- `1**0VVVV` - `JSO`: Output `VVVV` and Jump to `VVVV` If Input Bit Set

## Theory of Operation

One of the 74*161s is used as a settable register, and isn't allowed to increment the value. So if the O Bit is not set, the contents of `VVVV` are loaded into the register, and and the `QA`-`QD` pins preserve the output until another `OUT` (or `JMO` or `JSO`) instruction occurs

Instruction Decoding works as follows

- If `O` is 0 then load `VVVV` into the output, regardless of any other bits 
- If `P` is 0 and `C` is 1 then set the `PC` to `VVVV` If In is 1
- If `P` is 1 then set the `PC` to `VVVV`

Or to put it another way:

- `O` is always acted on
- `P` ignores C
- `C` checks if In is set

## License

BSD 2-Clause License

Copyright (c) 2017, Jock Murphy

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

1. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


