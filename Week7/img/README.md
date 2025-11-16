# Week 7

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

## Placement of VSDBabySoC:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place

![](img/place1.png)
![](img/place2.png)


### Placement Result Layout:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place


