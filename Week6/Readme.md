# Week 6 – OpenLane Physical Design Flow

This folder contains all tasks, notes, and results completed during **Week 6** of the RISC-V SoC Tapeout Program.  
The focus of this week was on **OpenLane physical design**, covering floorplanning, placement, routing, parasitic extraction, and timing analysis.

---

## Table of Contents

| Day | Subsection | Focus / Description | Details |
|-----|------------|---------------------|---------|
| **Day 1** | Floorplanning | Understanding core/die area, aspect ratio, utilization, pre-placed cells, tap cells, decaps, pin placement, and viewing layout in Magic. | Learned how dimensions influence congestion, power distribution, and routing quality. |
| **Day 2** | Placement | Standard-cell placement, legalization, cell alignment, and power rail alignment. | Studied global vs detailed placement and how OpenLane organizes cells efficiently. |
| **Day 3** | Routing | Global routing (FastRoute), detailed routing (TritonRoute), preferred L-shaped paths, and Lee’s maze routing algorithm. | Completed DRC-clean routing and analyzed wire pitch, spacing, and metal layers. |
| **Day 3** | Parasitic Extraction | SPEF extraction after routing to capture resistance and capacitance. | Generated SPEF for accurate post-route timing and noise analysis. |
| **Day 4** | Timing Optimization | Post-route STA, slack debugging, fixing violations, gate sizing, and buffer insertion. | Replaced slow cells, updated synthesis strategy, and improved slack margins. |
| **Day 5** | Clock Tree Synthesis (CTS) | Building a balanced clock tree to control skew and meet timing. | Performed CTS in OpenROAD and re-ran STA after clock network insertion. |

---

### Notes
- All Week 6 steps were executed in **OpenLane** using the **sky130_fd_sc_hd** PDK.  
- Generated outputs include: floorplan DEF, placement DEF, routed DEF, GDSII, SPEF, and timing reports.


