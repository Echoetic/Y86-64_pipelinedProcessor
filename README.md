# Y86-64 Pipelined Processor in Verilog

![Verilog](https://img.shields.io/badge/Language-Verilog-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

## Overview

This project presents a comprehensive implementation of a 5-stage pipelined processor for the Y86-64 instruction set architecture (ISA), developed entirely in Verilog. The processor design includes critical features such as data hazard management through forwarding, control hazard mitigation using a Return Address Stack (RAS), and a direct-mapped cache system for memory operations. It is designed to correctly execute the full range of Y86-64 instructions, demonstrating key concepts in modern computer architecture.

## Key Features

- **5-Stage Pipeline:** Implements Fetch (F), Decode (D), Execute (E), Memory (M), and Write-back (W) stages with corresponding pipeline registers.
- **Full Y86-64 ISA Support:** Compatible with all standard Y86-64 instructions, including conditional moves and jumps.
- **Data Hazard Handling:**
  - **Full Forwarding Logic:** Implements forwarding paths from Execute, Memory, and Write-back stages to the Decode stage to minimize stalls.
  - **Load-Use Hazard Detection:** Stalls the pipeline for one cycle when a load-use (Read-After-Write) hazard is detected.
- **Control Hazard Handling:**
  - **Return Address Stack (RAS):** A 16-entry stack for predicting return addresses for `call` and `ret` instructions.
  - **Misprediction Recovery:** Flushes the pipeline and refetches the correct instruction upon a control hazard misprediction.
- **Cache System:**
  - **Direct-Mapped Cache:** A simple and efficient cache for the data memory (`DMEM`).
  - **16-Byte Cache Lines:** Organizes memory into 16-byte blocks to leverage spatial locality.
  - **Cache Coherency:** Ensures consistency between the cache and main memory.
- **System Reset:** A global reset mechanism to initialize all registers, memories, and processor state to a known starting point.

## Architecture Details

### 1. Core Modules

- **`IMEM` (Instruction Memory):**
  - 256-byte storage capacity.
  - Pre-loaded with an example program for summing array elements.
  - Little-endian data organization.

- **`DMEM` (Data Memory with Cache):**
  - Implements a direct-mapped cache with 16-byte cache lines.
  - Pre-initialized with an array: `[0x0d...d, 0x0c...c, 0x0b...b, 0x0a...a]`.

- **`Y86` (Processor Core):**
  - Contains the 5-stage pipeline logic.
  - Manages data forwarding, hazard detection, and pipeline stalls.
  - Implements the Return Address Stack (RAS) for control prediction.
  - Updates condition codes (CC) and status flags (`Stat`).

### 2. Pipeline Stages

- **Fetch (F):** Predicts the next Program Counter (PC) and fetches the instruction from `IMEM`.
- **Decode (D):** Decodes the instruction, reads operands from the register file, and prepares values for the next stage.
- **Execute (E):** Performs ALU operations and evaluates conditional jump conditions.
- **Memory (M):** Accesses the data memory (`DMEM`) for `mrmovq` and `rmmovq` instructions.
- **Write-back (W):** Writes results back to the register file.

## Example Program: Array Sum

The processor is pre-loaded with a YAS assembly program that calculates the sum of a 4-element array.

```yas
# Initialization
irmovq stack, %rsp  # Initialize stack pointer
call main           # Call the main function
halt                # Halt the processor

main:
    irmovq array, %rdi  # Load array base address into %rdi
    irmovq $4, %rsi     # Load array length (4) into %rsi
    call sum            # Call the sum function
    ret                 # Return from main

sum:
    irmovq $8, %r8      # Constant 8 (for address increment)
    irmovq $1, %r9      # Constant 1 (for loop decrement)
    xorq %rax, %rax     # Initialize sum (%rax) to 0
    andq %rsi, %rsi     # Check if loop counter is zero
    jmp test            # Jump to loop condition test

loop:
    mrmovq (%rdi), %r10 # Read element from memory
    addq %r10, %rax     # Add element to sum
    addq %r8, %rdi      # Move to next element
    subq %r9, %rsi      # Decrement loop counter

test:
    jne loop            # If counter is not zero, continue loop
    ret                 # Return from sum
```

**Functionality:** This program computes the sum of the array `[0x0d...d, 0x0c...c, 0x0b...b, 0x0a...a]` and stores the final result in the `%rax` register.

## Usage Instructions

### Input Signals

| Signal  | Description                 |
|---------|-----------------------------|
| `CLK`   | System clock signal.        |
| `reset` | Reset signal (active high). |

### Output Signals

The processor's internal state can be monitored through the following output signals from each pipeline stage:

| Signal       | Description                               |
|--------------|-------------------------------------------|
| `F_predPC`   | Predicted PC in the Fetch stage.          |
| `D_icode`    | Instruction code in the Decode stage.     |
| `D_rA`, `D_rB` | Register operands from the Decode stage.  |
| `E_icode`    | Instruction code in the Execute stage.    |
| `E_valA`, `E_valB` | Operands for the ALU in Execute.      |
| `M_icode`    | Instruction code in the Memory stage.     |
| `M_valE`     | ALU result from the Memory stage.         |
| `W_icode`    | Instruction code in the Write-back stage. |
| `W_valE`, `W_valM` | Values to be written back.            |
| `Stat`       | Processor status (`AOK`, `HLT`, `ADR`).   |

### Operating Procedure

1.  **Reset the System:**
    - Assert the `reset` signal to initialize all modules, clear registers, and load initial memory values.
2.  **Start Execution:**
    - Provide a `CLK` signal to begin program execution automatically.
3.  **Monitor Execution:**
    - Observe the output signals to trace the state of each pipeline stage.
    - The program completes when the `Stat` output becomes `HLT`.

## Design Highlights

1.  **Advanced RAS Implementation:**
    - Handles up to 16 nested `call` instructions.
    - Recovers efficiently from mispredictions to maintain state consistency.
2.  **Efficient Cache System:**
    - Optimized for fast access with 16-byte cache lines.
    - Includes handling for cache misses and a write-back policy.
3.  **Precise Hazard Handling:**
    - Full data forwarding minimizes pipeline stalls.
    - Accurate pipeline control during hazard detection and recovery.
4.  **Robust Verification:**
    - Includes mechanisms for detecting invalid memory addresses and illegal instructions.
    - Provides status monitoring for debugging and verification.
