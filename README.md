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
| [**Week 5**](/Week5) | – | **GCD Design Floorplanning & Placement using OpenROAD** | Executed the OpenROAD flow for **GCD** using Nangate45. Covered installation, floorplanning, PDN generation, global and detailed placement. Verified outputs with DEF and Verilog files. |
| [**Week 6**](/Week6) | – | **Routing, DRC, Parasitics & Layout Verification** | Covered global/detailed routing, DRC rules, parasitic extraction (SPEF), and post-route timing analysis. |
| [**Week 6**](/Week6/Day1) | **Day 1** | Introduction to Routing | Learned preferred L-shaped routing, maze routing (Lee’s algorithm), and grid-based shortest-path finding. |
| [**Week 6**](/Week6/Day2) | **Day 2** | Global & Detailed Routing | Explored FastRoute for global routing, route guide creation, and TritonRoute for detailed routing. |
| [**Week 6**](/Week6/Day3) | **Day 3** | DRC Rules & Layout Validation | Studied width, spacing, pitch, short checks, and layer orientation rules for clean layout routing. |
| [**Week 6**](/Week6/Day4) | **Day 4** | Parasitic Extraction (SPEF) | Generated SPEF after routing and learned how RC parasitics improve post-route timing accuracy. |
| [**Week 6**](/Week6/Day5) | **Day 5** | Post-Route Timing & Verification | Performed post-route STA with parasitics and verified DRC cleanliness and timing closure. |
| [**Week 7**](/Week7) | – | **BabySoC Physical Design & Post-Route Debug** | Completed synthesis, floorplanning, placement, clock tree synthesis, and initial routing of BabySoC. Debugged routing congestion and MET4 OBS violations. Updated LEF/macros and resolved DRC and timing issues. |
| [**Week 8**](/Week8) | – | **Post-Layout STA & Timing Analysis** | Performed post-route static timing analysis across multiple PVT corners using routed SPEF, Liberty, netlist, and SDC files. Compared WNS, TNS, and worst slack reports; analyzed parasitic and CTS effects on setup/hold timing. |

---



