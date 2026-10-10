# Parametric RISC CPU

A parameterized RISC-style CPU implemented in **Verilog HDL**. The
design combines instruction fetch and decode logic with a register file,
an ALU, pipeline registers, and stall-based hazard handling.

> **Project status:** RTL organization and documentation project. Run
> simulation and validate the instruction program before treating the
> design as verified or FPGA-ready.

## Features

-   **Configurable datapath width:** The top-level `cpu` module has a
    parameter `N` (default: 8 bits).
-   **Instruction fetch:** A program counter addresses a 64-word
    instruction memory.
-   **Instruction decoding:** Extracts opcode, destination/source
    register fields, and immediate data from a 16-bit instruction.
-   **Register file:** Eight registers, each `N` bits wide, with two
    combinational read ports and a clocked write port.
-   **ALU operations:** Arithmetic, logical, shifts, comparisons, and
    immediate operations.
-   **Pipeline registers:** Includes an instruction-fetch register and
    an ID/EX register.
-   **Hazard handling:** Detects selected source-register dependencies
    and uses stall control to hold the PC and pipeline register.
-   **Instruction loading:** Instruction memory is initialized from
    `program.mem` using `$readmemb`.

## Architecture

The current RTL is organized around this datapath:

``` text
                  +----------------------+
                  |    Program Counter   |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  |  Instruction Memory  |
                  |   64 x 16-bit words  |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  | Instruction Fetch    |
                  |     IF register      |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  | Instruction Decoder  |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  |      ID/EX Register  |
                  +----------+-----------+
                             |
                             v
                  +----------------------+
                  |    Execution Unit    |
                  | Register File + ALU  |
                  | Hazard / Stall Logic |
                  +----------+-----------+
                             |
                             v
                     Result and Flags
```

**Note:** This diagram reflects the modules and connections in the
provided RTL. Although the design contains pipeline registers and
stall-based hazard handling, the supplied top-level integration does not
implement a complete five-stage pipeline. Branch and jump inputs exist
on the program counter, but the current control unit ties them inactive.

## Module descriptions

  -----------------------------------------------------------------------
  File / module                       Responsibility
  ----------------------------------- -----------------------------------
  `cpu`                               Top-level integration; connects
                                      control, ID/EX register, and
                                      execution unit. `N` defaults to 8.

  `control_unit`                      Connects the program counter,
                                      instruction memory,
                                      instruction-fetch register, and
                                      decoder.

  `program_counter`                   Maintains the instruction address;
                                      supports stall, branch, and jump
                                      inputs.

  `instruction_memory`                Stores 64 16-bit instructions and
                                      loads them from `program.mem`.

  `instruction_fetch`                 Registers the fetched instruction
                                      and supports reset and stall.

  `instruction_decoder`               Extracts opcode, `rd`, `rs1`,
                                      `rs2`, immediate, and
                                      immediate-source control.

  `id_ex`                             Registers decoded instruction
                                      fields between decode and
                                      execution.

  `execution_unit`                    Selects ALU operands, instantiates
                                      the register file and ALU, and
                                      manages stall tracking.

  `register_file`                     Implements eight `N`-bit registers
                                      with two read ports and one write
                                      port.

  `pipeline_alu`                      Implements the registered ALU
                                      stages and four status flags.

  `stall_unit`                        Detects selected register
                                      dependencies using tracked
                                      destination registers and opcodes.
  -----------------------------------------------------------------------

## Supported operations

The ALU opcode cases in the supplied RTL include:

  Operation         Opcode
  ----------------- ---------
  NOP               `00000`
  ADD               `00001`
  SUB               `00010`
  INC               `00011`
  DEC               `00100`
  AND               `00101`
  OR                `00110`
  XOR               `00111`
  NOT               `01000`
  SLL               `01001`
  SRL               `01010`
  SRA               `01011`
  SLT (signed)      `01100`
  SLTU (unsigned)   `01101`
  ADDI              `01110`
  SUBI              `01111`
  ANDI              `10000`
  ORI               `10001`
  XORI              `10010`
  LDI               `10011`

Opcodes are shown as 5-bit binary values. Confirm instruction encodings
and operand conventions against the RTL before writing a program.

## Instruction format

The decoder reads a **16-bit instruction** using these fields:

  Bits        Field
  ----------- ------------------------------------
  `[15:11]`   Opcode (5 bits)
  `[10:8]`    Destination register `rd` (3 bits)
  `[7:5]`     Source register `rs1` (3 bits)
  `[4:2]`     Source register `rs2` (3 bits)
  `[7:0]`     Immediate value (8 bits)

The immediate field overlaps the `rs1` and `rs2` fields. This is how the
current RTL is written; use the instruction format carefully when
preparing `program.mem`.

## Status flags

The `flag` output is four bits wide:

  Bit         Meaning
  ----------- ---------------------
  `flag[3]`   Carry
  `flag[2]`   Zero
  `flag[1]`   Negative (sign bit)
  `flag[0]`   Overflow

## Tools and languages

-   **HDL:** Verilog
-   **Suggested design/simulation environment:** AMD/Xilinx Vivado
-   **Instruction file format:** Binary text loaded with `$readmemb`

The project description mentions Vivado, but no verified simulation
results are included in this repository yet. Add simulator version,
target FPGA board, and test results after validating them.

## Repository contents

``` text
.
├── README.md
├── cpu.txt
├── control_unit.txt
├── program_counter.txt
├── instruction_memory.txt
├── instruction_fetch.txt
├── instruction_decoder.txt
├── id_ex.txt
├── stall_unit.txt
├── execution_unit.txt
├── register_file.txt
├── pipeline_alu.txt
└── program.mem   # Add your actual instruction program
```

The repository currently uses `.txt` extensions for the source files. To
compile the RTL in a Verilog tool, rename the source files to `.v` (for
example, `cpu.txt` to `cpu.v`) and add them to the project. Keep the
module names and file names consistent.

## Simulation instructions

1.  Download or clone the repository.
2.  Rename the source files from `.txt` to `.v`. 
3.  Create `program.mem` containing the binary instruction words for
    your test program. Each line should contain one 16-bit binary word
    compatible with `$readmemb`.
4.  Add all Verilog source files to a Vivado RTL project.
5.  Set `cpu` as the top module and use `N=8` for the default 8-bit
    datapath.
6.  Create a testbench that drives `clk` and `reset`, runs enough clock
    cycles for the program, and checks `Y` and `flag`.
7.  Run behavioral simulation and inspect the waveform. Validate
    dependencies, immediate operations, and reset behavior before
    attempting FPGA implementation.

Example instruction-memory initialization:

``` verilog
initial begin
    $readmemb("program.mem", memory);
end
```

Ensure `program.mem` is available in the simulator's working directory
or configure the appropriate file path in your simulation project.



