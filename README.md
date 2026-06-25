# RISC-V Single Cycle Processor

A hardware implementation of a **RISC-V Single Cycle Processor** written in Verilog HDL. This project implements the RV32I base integer instruction set, where each instruction completes execution in a single clock cycle.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Supported Instructions](#supported-instructions)
- [Module Structure](#module-structure)
- [Datapath](#datapath)
- [Getting Started](#getting-started)
- [Simulation](#simulation)
- [Waveform](#waveform)
- [Directory Structure](#directory-structure)
- [Tools Used](#tools-used)
- [References](#references)

---

## Overview

The **single-cycle** design executes one instruction per clock cycle. The entire datapath — fetch, decode, execute, memory access, and write-back — completes within a single cycle. While not the most performance-efficient architecture, it provides a clean and easy-to-understand implementation that forms the foundation for pipelined and out-of-order designs.

**Key characteristics:**
- ISA: RISC-V RV32I (32-bit base integer)
- Design style: Single-cycle (non-pipelined)
- HDL: Verilog / SystemVerilog
- Clock: Combinational datapath, clocked memory and register file

---

## Architecture

```
         ┌──────────────────────────────────────────────────────┐
         │                  RISC-V Single Cycle CPU              │
         │                                                        │
  ┌──────┴──────┐   ┌──────────┐   ┌────────┐   ┌────────────┐ │
  │   Instruction│   │  Register│   │  ALU   │   │    Data    │ │
  │    Memory   │   │   File   │   │        │   │   Memory   │ │
  └──────┬──────┘   └────┬─────┘   └───┬────┘   └─────┬──────┘ │
         │               │             │               │         │
         └───────────────┴─────────────┴───────────────┘         │
                         ▲                                        │
                  ┌──────┴──────┐                                │
                  │   Control   │                                │
                  │    Unit     │                                │
                  └─────────────┘                                │
         └──────────────────────────────────────────────────────┘
```

### Key Stages (all in one cycle)

| Stage | Description |
|-------|-------------|
| **IF** – Instruction Fetch | PC → Instruction Memory → Instruction |
| **ID** – Instruction Decode | Decode opcode, read registers, generate control signals |
| **EX** – Execute | ALU performs arithmetic/logic, compute branch target |
| **MEM** – Memory Access | Load/Store operations on Data Memory |
| **WB** – Write Back | Write result back to Register File |

---

## Supported Instructions

### R-Type (Register-Register)

| Instruction | Operation |
|-------------|-----------|
| `ADD` | rd = rs1 + rs2 |
| `SUB` | rd = rs1 - rs2 |
| `AND` | rd = rs1 & rs2 |
| `OR` | rd = rs1 \| rs2 |
| `XOR` | rd = rs1 ^ rs2 |
| `SLL` | rd = rs1 << rs2 |
| `SRL` | rd = rs1 >> rs2 |
| `SRA` | rd = rs1 >>> rs2 |
| `SLT` | rd = (rs1 < rs2) ? 1 : 0 |
| `SLTU` | rd = (rs1 < rs2) unsigned |

### I-Type (Immediate)

| Instruction | Operation |
|-------------|-----------|
| `ADDI` | rd = rs1 + imm |
| `ANDI` | rd = rs1 & imm |
| `ORI` | rd = rs1 \| imm |
| `XORI` | rd = rs1 ^ imm |
| `LW` | rd = Memory[rs1 + imm] |
| `LH`, `LB` | Load halfword / byte |
| `JALR` | Jump and link register |

### S-Type (Store)

| Instruction | Operation |
|-------------|-----------|
| `SW` | Memory[rs1 + imm] = rs2 |
| `SH`, `SB` | Store halfword / byte |

### B-Type (Branch)

| Instruction | Operation |
|-------------|-----------|
| `BEQ` | Branch if rs1 == rs2 |
| `BNE` | Branch if rs1 != rs2 |
| `BLT` | Branch if rs1 < rs2 |
| `BGE` | Branch if rs1 >= rs2 |

### U-Type & J-Type

| Instruction | Operation |
|-------------|-----------|
| `LUI` | rd = imm << 12 |
| `AUIPC` | rd = PC + (imm << 12) |
| `JAL` | rd = PC+4; PC = PC + imm |

---

## Module Structure

```
top.v
├── pc_register.v          # Program Counter
├── instruction_memory.v   # ROM for instructions
├── control_unit.v         # Main control signal generator
├── register_file.v        # 32 x 32-bit register file
├── imm_gen.v              # Immediate value generator / sign-extender
├── alu.v                  # Arithmetic Logic Unit
├── alu_control.v          # ALU operation decoder
├── data_memory.v          # RAM for data (load/store)
└── mux.v                  # Various multiplexers
```

---

## Datapath

The datapath connects all components with control signals from the **Control Unit**:

```
PC ──► Instr Mem ──► [Decode] ──► Reg File ──► ALU ──► Data Mem ──► Reg File
         │                           ▲              ▲
         │                      ImmGen          ALU Control
         │                           │              │
         └──────────── Control Unit ─┴──────────────┘
```

**Control signals generated:**

| Signal | Width | Description |
|--------|-------|-------------|
| `RegWrite` | 1 | Enable write to register file |
| `ALUSrc` | 1 | Select ALU second operand (reg or imm) |
| `MemWrite` | 1 | Enable data memory write |
| `MemRead` | 1 | Enable data memory read |
| `MemToReg` | 2 | Select write-back source |
| `Branch` | 1 | Enable branch condition check |
| `Jump` | 1 | Unconditional jump enable |
| `ALUOp` | 2 | ALU operation type |

---

## Getting Started

### Prerequisites

- [Icarus Verilog](http://iverilog.icarus.com/) or [ModelSim](https://www.intel.com/content/www/us/en/software/programmable/quartus-prime/model-sim.html)
- [GTKWave](http://gtkwave.sourceforge.net/) for waveform viewing (optional)
- Make (optional, for build scripts)

### Clone the Repository

```bash
git clone https://github.com/yourusername/riscv-single-cycle.git
cd riscv-single-cycle
```

---

## Simulation

### Using Icarus Verilog

```bash
# Compile
iverilog -o sim/riscv_sim src/*.v testbench/tb_top.v

# Run simulation
vvp sim/riscv_sim

# View waveform
gtkwave sim/dump.vcd
```

### Using ModelSim

```tcl
vlib work
vlog src/*.v testbench/tb_top.v
vsim -c tb_top -do "run -all; quit"
```

---

## Waveform

After simulation, open `sim/dump.vcd` in GTKWave. Key signals to observe:

- `clk`, `reset`
- `PC` – Program Counter value each cycle
- `instruction` – fetched instruction word
- `alu_result` – ALU output
- `reg_write_data` – value written to register file
- `mem_read_data` – data loaded from memory

---

## Directory Structure

```
riscv-single-cycle/
│
├── src/                     # RTL source files
│   ├── top.v
│   ├── alu.v
│   ├── control_unit.v
│   ├── register_file.v
│   ├── data_memory.v
│   ├── instruction_memory.v
│   ├── imm_gen.v
│   └── ...
│
├── testbench/               # Testbenches
│   └── tb_top.v
│
├── sim/                     # Simulation output (waveforms)
│   └── dump.vcd
│
├── docs/                    # Documentation / diagrams
│   └── datapath.png
│
└── README.md
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Icarus Verilog | HDL simulation |
| GTKWave | Waveform viewer |
| VS Code | Code editor |
| ModelSim (optional) | Functional simulation |
| Vivado (optional) | FPGA synthesis & implementation |

---

## References

- [RISC-V Specification (RV32I)](https://riscv.org/technical/specifications/)
- Patterson & Hennessy — *Computer Organization and Design: RISC-V Edition*
- [RVfpga Course by SysEDA](https://university.imgtec.com/rvfpga-el2-v3-0-english-downloads-page/)
- [Digital Design and Computer Architecture – ETH Zürich](https://safari.ethz.ch/digitaltechnik/spring2021/doku.php)

---

## License

This project is licensed under the [MIT License](LICENSE).

---

> **Note:** If you upload your Verilog files, the README can be updated to reflect your exact module names, supported instructions, and testbench details.
