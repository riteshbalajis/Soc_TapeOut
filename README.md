# RISC-V SoC Tapeout Program – Learning Journey

This repository contains the **weekly submissions** for the RISC-V SoC tapeout program.  
It summarizes the content covered and tasks completed each week.

---

## Table of Contents

| Week | Subsection | Focus / Description | Details |
|------|-------------|---------------------|----------|
| [**Week 0**](/Week0) | – | **Introduction to SoC Tapeout Flow** | Understanding the application specification, chip modeling, and installing open-source tools like **Yosys**, **Icarus Verilog**, and **GTKWave**. |
| [**Week 1**](/Week1) | – | **RTL Design and Synthesis** | Hands-on Verilog coding, simulation, synthesis, optimization, and understanding library usage, hierarchical vs flat synthesis, flip-flop usage, and hardware replication. |
| [**Week 1**](/Week1/Day1) | **Day 1** | RTL Design and Simulation Workshop | Basics of RTL design, simulation setup, testbench creation, and waveform verification. |
| [**Week 1**](/Week1/Day2) | **Day 2** | Timing Libraries & Efficient Use of Flip-Flops | Understanding timing libraries, hierarchical vs flat synthesis, flip-flop coding styles, and multiplication optimizations. |
| [**Week 1**](/Week1/Day3) | **Day 3** | Gate-Level Modeling and Simulation | Introduction to gate-level models, netlist generation, timing-aware vs functional models, and GLS verification. |
| [**Week 1**](/Week1/Day4) | **Day 4** | Gate-Level Simulation (GLS) | Detailed GLS process, blocking vs non-blocking statements, sensitivity list issues, and waveform analysis for verification. |
| [**Week 1**](/Week1/Day5) | **Day 5** | Optimization in Synthesis | IF/case statement pitfalls, inferred latches, partial/overlapping assignments, loops, for-generate, and ripple carry adder example. |
| [**Week 2**](/Week2) | – | **BabySoC Fundamentals & Functional Modelling** | Explored the **VSD BabySoC** modules including **RISC-V core (rvmyth)**, **PLL**, and **DAC**. Simulated these modules and analyzed waveforms in **GTKWave** to study digital-analog interfacing and clock behavior. |
| [**Week 3**](/Week3) | – | **Post-Synthesis GLS & Static Timing Analysis** | Performed **Gate-Level Simulation** of BabySoC using **Yosys**, explored **Static Timing Analysis (STA)** fundamentals, and analyzed timing reports using **OpenSTA**. |
| [**Week 3**](/Week3/Task1) | **Task 1** | Post-Synthesis Gate-Level Simulation (GLS) | Synthesized VSD BabySoC with **Yosys**, simulated with **Icarus Verilog**, and verified waveforms using **GTKWave**. Pre- and post-synthesis waveforms matched, confirming correct synthesis. |
| [**Week 3**](/Week3/Task2) | **Task 2** | Fundamentals of Static Timing Analysis (STA) | Studied timing parameters — **arrival time**, **required time**, **slack**, **setup/hold**, **slew**, **load**, **OCV**, and **fanout**. Analyzed timing paths using DAG models and computed setup/hold slacks. |
| [**Week 3**](/Week3/Task3) | **Task 3** | STA of VSDBabySoC using OpenSTA | Installed and used **OpenSTA** with Liberty, Netlist, and SDC files. Generated setup/hold timing reports. Verified that both setup (slack = +2.26 ns) and hold (slack = +0.09 ns) met requirements. |
| [**Week 4**](/Week4) | – | **Device-Level and CMOS Inverter Analysis using NGSpice** | Implemented NMOS ID-VDS/ID-VGS characteristics, studied **velocity saturation**, analyzed **CMOS inverter** VTC, switching threshold, rise/fall delays, noise margins, and power-supply variations using **NGSpice** simulations. |

---

### Notes
- All simulations and analyses were performed using open-source tools: **Yosys**, **Icarus Verilog**, **GTKWave**, **OpenSTA**, and **NGSpice**.  
- Reports, netlists, and waveform outputs are available in their respective weekly folders.

---
