# 1-Bitsy The SAP CPU

## Description 
The goal was to make the simplest possible thing that could be called a CPU.  It had to be able to do the following:

- Sequence.  Progress from one instruction to another.
- Selection. Change the course of execution based on some criteria.
- Iteration. Change the course of execution.

There is no ALU at all, no data memory, and no internal registers.  It is just able to give predetermined outputs to some external input.  However the minimum viable system consists of three chips:

- 2 74*161 4-bit counter with clear
- 1 74*02 quad 2 input nor gate

This system allows for addressing 16 bytes of read only 8-bit memory.  Not much (if anything) practical can be done with these constraints, but it does technically meet all the criteria.  As we all know being technically correct is the best kind of correct.


### Expandability 

It would be fairly easy to expand the system to be able to address 256 bytes of 12-bit read only memory by adding addition 74*161s and tweaking the decoding logic.  This is left as an intellectual exercise for the reader

Additionally there is currently an unused bit in the instruction nibble

## The System

### I/O

Output consists of 4 output bits, there is a single input bit.

### Instruction Byte

| Bit(s)   | Name | Description |
| -------- | ---- | ----------- |
| Bit  7   | P    | PC Load     |
| Bit  6   | C    | Condition   |
| Bit  5   |      | Unused      |
| Bit  4   | O    | Output      |
| Bits 3-0 | VVVV | Value       |


### Instructions

| 76543210 | ASM | Description                |
| -------- | --- | -------------------------- |     
| ***0VVVV | OUT | Output VVVV                | 
| 00*1**** | NOP | No Operation               | 
| 01*1VVVV | JMP | PC = VVVV If Input Bit Set | 
| 1**1VVVV | JST | PC = VVVV                  | 

There are two additional "phantom instructions" of dubious utility:

- 01*0VVVV - JMO: Output VVVV and Jump to VVVV
- 1**0VVVV - JSO: Output VVVV and Jump to VVVV If Input Bit Set

## Theory of Operation

One of the 74*161s is used as a settable register, and isn't allowed to increment the value. So if the O Bit is not set, the contents of VVVV are loaded into the register, and and the QA-QD pins preserve the output until another OUT (or JMO or JSO) instruction occurs

Instruction Decoding works as follows

- If O is 0 then load VVVV into the output 
- If P is 0 and C is 1 then set the PC to VVVV If In is 1
- If P is 1 then set the PC to VVVV

Or to put it another way:

- O is always acted on
- P ignores C
- C checks if In is set
