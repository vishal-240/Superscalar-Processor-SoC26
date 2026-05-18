<div align="center">

# PRAVAH
### A 2-Wide Out-of-Order Superscalar RISC-V Processor

<!--*Seasons of Code 2026 · IIT Bombay*-->

[![Verilog](https://img.shields.io/badge/HDL-Verilog-blue.svg)]()
[![ISA](https://img.shields.io/badge/ISA-RISC--V%20(RV32I)-orange.svg)]()
[![Tools](https://img.shields.io/badge/Tools-Quartus%20%2B%20ModelSim-green.svg)]()
[![Status](https://img.shields.io/badge/Status-In%20Progress-yellow.svg)]()

</div>

---

## About

**PRAVAH** (प्रवाह — "flow") is a 2-wide out-of-order superscalar processor built from scratch in Verilog as part of **Seasons of Code 2026** at IIT Bombay. The project takes 2nd-year undergraduates from a textbook understanding of 5-stage pipelined MIPS to a working out-of-order RISC-V core, implementing the same architectural principles that power modern CPUs like Apple's M-series and AMD's Zen cores.

Mentees fetch two instructions per cycle, rename registers to eliminate false dependencies, dispatch them out of order through reservation stations, execute on multiple functional units, and commit them in order via a reorder buffer — Tomasulo's algorithm, brought to life in synthesizable RTL.

> **Scope:** This project is **simulation-based**. The full processor is developed, verified, and benchmarked on ModelSim. The RTL is written to be synthesizable, so the design can be extended to run on any Intel Cyclone FPGA (Cyclone V, DE10-Lite, DE10-Standard, etc.) in the future — but FPGA bring-up is **not** a requirement of the program. Simulation correctness and a clean Quartus synthesis report are the deliverables.

## What we're building

A synthesizable 2-wide superscalar processor with:

- **2-wide fetch** with aligned instruction memory
- **2-wide decoder** producing internal micro-ops
- **Register renaming** with a physical register file (48–64 physical registers)
- **Reservation stations** (4 ALU + 2 MUL/MEM) with CDB snooping
- **Reorder buffer** (8–16 entries) for precise in-order commit
- **Functional units:** 2 ALUs, 1 multi-cycle multiplier, 1 load/store unit
- **Common data bus** with arbitration
- **2-bit saturating branch predictor**

**Target ISA:** RISC-V RV32I subset (~12–15 instructions: ADD, SUB, AND, OR, XOR, SLL, SRL, SLT, ADDI, LW, SW, BEQ, BNE, JAL)

**Target FPGA (for synthesis sign-off):** Intel Cyclone V / DE10-Lite — design is verified to synthesize cleanly but is not deployed on hardware in this program.

## Tech stack

| Layer           | Tool                          |
|-----------------|-------------------------------|
| HDL             | Verilog                       |
| Simulation      | ModelSim-Intel Starter Edition|
| Synthesis       | Quartus Prime Lite            |
| ISA             | RISC-V (RV32I subset)         |
| Version control | Git + GitHub                  |

## Repository structure (tentative)

> **Note to mentees:** Please mirror this exact directory structure in your own fork. It is not arbitrary — it matches how the mentor will review your code, where the milestone scripts expect files to live, and how the debugging checklist references modules. Following this structure will save significant time when something breaks at Milestone 3 and we need to bisect the bug together. A clean, predictable repo also looks great on a resume.

```
pravah/
├── README.md
├── docs/                        # Design documents, block diagrams, reports
│   ├── design_decisions.md
│   ├── block_diagram.png
│   └── final_report.pdf
├── rtl/                         # Synthesizable Verilog modules
│   ├── fetch.v
│   ├── decode.v
│   ├── rename_unit.v
│   ├── register_file.v
│   ├── reservation_station.v
│   ├── rob.v
│   ├── alu.v
│   ├── mul.v
│   ├── lsu.v
│   ├── cdb.v
│   └── top.v
├── tb/                          # Testbenches
│   ├── tb_register_file.v
│   ├── tb_reservation_station.v
│   ├── tb_rob.v
│   └── tb_top.v
├── programs/                    # Benchmark micro-programs (hex/asm)
│   ├── dot_product.hex
│   ├── bubble_sort.hex
│   └── branch_heavy.hex
├── quartus/                     # Quartus project files
├── sim/                         # ModelSim scripts and waveforms
├── learning_log.md              # Weekly learning notes
└── LICENSE
```

## How it works (high level)

```
        ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
   ────►│  FETCH   ├──►│  DECODE  ├──►│  RENAME  ├──►│ DISPATCH │
        │ (2-wide) │   │ (2-wide) │   │  + ROB   │   │          │
        └──────────┘   └──────────┘   └──────────┘   └────┬─────┘
                                                          │
                ┌─────────────────────────────────────────┘
                │
                ▼
        ┌───────────────┐   wakeup    ┌──────────────┐
        │ RESERVATION   ├────────────►│ FUNC. UNITS  │
        │   STATIONS    │             │ ALU/ALU/MUL  │
        │               │◄────────────┤   /LSU       │
        └───────┬───────┘   CDB       └──────┬───────┘
                │  broadcast                 │
                │                            │
                └────────────►  CDB  ◄───────┘
                                │
                                ▼
                          ┌──────────┐
                          │   ROB    │  in-order commit → arch RF
                          │ (commit) │
                          └──────────┘
```

A real block diagram will replace this ASCII sketch from Week 5 onward (see `docs/block_diagram.png`).

## Progress

This repository is updated week-by-week.

### Week 0 — Setup & Prerequisites

### Week 1 — Pipeline Hazards (Theory)

### Week 2 — Tomasulo & Scoreboarding — Milestone 1

### Week 3 — Register Renaming & ROB; Verilog Kickoff

### Week 4 — Branch Prediction & Reservation Stations — Milestone 2

### Week 5 — Superscalar Issue Logic; Integration Begins

### Week 6 — ROB, CDB, Wakeup/Select — Milestone 3

### Week 7 — Debugging, IPC, Synthesis

### Week 8 — Polish, Documentation, Final Review

## Build & simulate

> Detailed build instructions will be added after Week 5 when the top-level integration begins.

**Prerequisites:**

- Quartus Prime Lite (free) — Intel FPGA toolchain
- ModelSim-Intel Starter Edition (bundled with Quartus)

Synthesis is run to confirm the RTL is synthesizable and to extract Fmax, logic-element usage, and the timing summary. The design is not deployed on a physical FPGA in this program — that is left as a post-program extension for any mentee who wants to take it further on a Cyclone V, DE10-Lite, or similar Intel board.

## Resources we're using

**Textbooks**

- Hennessy & Patterson — *Computer Architecture: A Quantitative Approach* (6th ed., Chapter 3)
- Patterson & Hennessy — *Computer Organization and Design* (5th ed., Chapter 4)
- Shen & Lipasti — *Modern Processor Design: Fundamentals of Superscalar Processors*

**Lectures**

- Onur Mutlu — Computer Architecture, ETH Zürich (YouTube)
- Smruti Sarangi — Advanced Computer Architecture, NPTEL (IIT Delhi)
- MIT OCW 6.823 — Computer System Architecture

**Specifications**

- RISC-V Unprivileged ISA Specification

## Reference implementations (for study, not copying)

- [BOOM](https://github.com/riscv-boom/riscv-boom) — Berkeley Out-of-Order Machine
- [RISC-V Sodor](https://github.com/ucb-bar/riscv-sodor) — Simple educational RISC-V cores
- [lowRISC ibex](https://github.com/lowRISC/ibex) — Clean in-order RISC-V core

## Mentors

**Krishna Kukreja and Naman Nayak**

---

<div align="center">

*"From IPC = 1 to out-of-order in 8 weeks."*

</div>
