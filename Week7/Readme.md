# Week 7 - BabySoC Physical Design & Post-Route

### Preparing the VSDBabySoC Design for OpenROAD

After creating the required directory structure, the next step is to place all design files and supporting data in the correct locations so that OpenROAD can run the full physical design flow.

### 1. Copy Required Design Assets
From your **VSDBabySoC** project folder, copy the following into:

`OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/`

- `gds/`
- `lef/`
- `lib/`
- `include/`
- `vsdbabysoc_synthesis.sdc`
- `macro.cfg`
- `pin_order.cfg`
- Create a new **config.mk** file for flow configuration

These files provide layout data, timing libraries, constraints, and pin information needed during synthesis, floorplan, placement, routing, and signoff.

![](img/cp2.png)
![](img/cp3.png)

- Files in `OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/`:

![](img/he_ls.png)

### 2. Add RTL Source Files


`OpenROAD-flow-scripts/flow/designs/src/vsdbabysoc/`

place **all Verilog RTL files** for the VSDBabySoC design.

This ensures the OpenROAD flow can find your design’s source code during synthesis.

![](img/cp4.png)
![](img/cp5.png)

- Files in `OpenROAD-flow-scripts/flow/designs/src/vsdbabysoc/`:

![](img/src_ls.png)

---
### Synthesis:

- command

    cd OpenROAD-flow-scripts/flow

    #To clean any previously generated files
    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk clean_all

    #Synthesis
    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth
![](img/synth1.png)
![](img/synth2.png)

- it is seen that the synthesis and finished and created some files inside results/sky130hd/vsdbabysoc/base/... 

![](img/ls_after_synth1.png)

- synth_check.txt:

    gvim synth_check.txt

![](img/synth_check.png)

-synth_stat.txt

    gvim synth_check.png

![](img/synth_stat.png)
![](img/synth_stat2.png)

-1_2_yosys.v
    gvim 1_2_yosys.v

![](img/yosys_v.png)

--- 

## FloorPlan:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan

![](img/floorplan1.png)
![](img/floorplan2.png)

- Let us open the floorplann Result Layout :

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan

![](img/floorplan_lay_c.png)
![](im/floorplan_lay.png)

![](img/floorplan_pll.png)
 - the selected bottom section is pll 

![](img/floorplan_dac.png)

- it is dac in vsdbabysoc

---

## Placement of VSDBabySoC:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place

![](img/place1.png)
![](img/place2.png)



### Placement Result Layout:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place

![](img/place_c.png)
![](img/place_lay.png)
![](img/place_lay2.png)
![](img/place_lay.3png)
![](img/place_lay4.png)
![](img/place_lay5.png)

-
![](img/place_vdd.png)
- it is vdd layer 
![](img/place_vdd2.png)

---

## Clock Tree Synthesis (cts):

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts

![](img/cts.png)
![](img/cts2.png)

### Clock Tree Synthesis Result:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts

![](img/cts_lay.png)
![](img/cts_lay1.png)
![](img/cts_lay2.png)

- Clock net :

![](img/cts_clk1.png)
![](img/cts_clk2.png)

### Clock Tree Viewer:

![](img/cls_tree2.png)

- **Top red marker** : The main clock source (clk) where the clock enters the design.
- **Long blue vertical line** : The primary line that carries the clock before it branches.
- **Green horizontal segment** : The first big split where CTS adds buffers to evenly distribute the clock.
- **Multiple blue downward lines** : Branches of the clock tree sending the clock signal to different flop groups; equal heights mean balanced timing.
- **Small red markers at the bottom** : Final clock endpoints showing where the clock is delivered. Here every point are mostly reaches at same time whihc represent less slew .

### Setup Report:

![](img/cts_sreep.png)

- it is clearly seen that required time > arrival time that is the signal arrives before the required time.

- the slack are postive slacks so the cts report for setup is met the timing contrain very well.

![](img/cts_hrep.png)

- here the required time < arrival time and ppsotive slack so, the hold constraints also well met.

### Setup Slack Chart:

![](img/cts_schart.png)

- Most endpoints have setup slack between 6 ns – 9 ns.

- Some endpoints have even more slack around 10–11 ns.

-  no negative slack, meaning no setup violations.

### Hold Slack Chart:

![](img/cts_hchart.png)

- Most endpoints have hold slack between 0.3 ns – 0.6 ns.

- A smaller number stretch between 0.6 ns – 1.5 ns and a few up to ~2.0 ns.

- No negative slack, which means there are no hold violations after CTS

---



