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

![](img/cts_tree2.png)

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

### Routing:

- routing fails with congestion error , while inspecting the error through the drc viewer it is clear that the error is from pins of dac 

![](img/drc1.png)
![](img/drc2.png)

Routing Debug Attempts Before Identifying the MET4 OBS Issue

Before discovering that the large MET4 OBS blockage in the LEF was the root cause of routing congestion, multiple routing-related settings were tried to reduce congestion and improve routability. Below are the methods attempted along with their expected effects and syntax.

| Control Area            | Variable Name             | Example Line in `config.mk`                      | Explanation (why it is done)                                               |
|-------------------------|----------------------------|--------------------------------------------------|-------------------------------------------------------------------------------|
| Core Sizing             | CORE_UTILIZATION          | export CORE_UTILIZATION = 0.50                   | Defines how much of the core can be filled with standard cells. A lower value leaves more free space for routing. |
| Placement Density       | PLACE_DENSITY             | export PLACE_DENSITY = 0.40                      | Controls how tightly cells are packed. Reducing it increases whitespace for routing and reduces congestion. |
| Die/Core Area Size      | DIE_AREA                  | export DIE_AREA = "0 0 3000 3000"                | Expands the chip area. A larger die gives more room for placement and routing, reducing congestion and allowing cleaner routing paths. |
| Macro Halo (Base)       | MACRO_PLACE_HALO          | export MACRO_PLACE_HALO = 50                     | Adds a keep-out margin around macros to prevent cells from crowding macro pins, improving pin escape routing. |
| Macro Halo (Parser Fix) | MACRO_PLACE_HALO_X        | export MACRO_PLACE_HALO_X = 50                   | Explicitly sets horizontal halo spacing to avoid internal Tcl parsing issues and maintain consistent spacing. |
| Macro Channel Spacing   | FP_MACRO_CHANNEL_X        | export FP_MACRO_CHANNEL_X = 100                  | Ensures a minimum routing gap between macros, creating clean routing channels for signals to pass through. |
| Router Effort           | GRT_CONGESTION_ITERATIONS | export GRT_CONGESTION_ITERATIONS = 120           | Increases the number of congestion-repair attempts during global routing, helping the tool find better routing solutions. |




### Correction in avsddac.lef file (OBS - Obstruction Section Reduction) :

- avsddac.lef

![](img/dac_lef_b.png)

The big MET4 blockage inside the OBS section was covering most of the macro area, so the router had almost no space left to route signals on MET4. Because of this, the tool ran into heavy congestion and couldn’t complete routing. After removing that large blockage, MET4 became available again, giving the router enough room to finish routing without errors.

###  correction is made in avsddac.lef:
- in metal layer 4 under the obs section we removed

![](img/dac_lef_b.png)


- also the  PLL position may be adjusted  because placing it below the DAC can lead to some issues. It may cause wires to overlap, make routing crowded, or create unnecessary routing complications. Keeping some spacing helps the tools place and route the design more cleanly.So creating a macro.tcl file for relocate the pll 

- macro.tcl

        place_macro -macro_name dac -location {100.0 100.0}
        place_macro -macro_name pll -location {1500.0 100.0}

- config.mk

        export DESIGN_NICKNAME = vsdbabysoc
        export DESIGN_NAME = vsdbabysoc
        export PLATFORM    = sky130hd
        export DESIGN_HOME = /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs
    
    
        # export VERILOG_FILES_BLACKBOX = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/IPs/*.v
        # export VERILOG_FILES = $(sort $(wildcard $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/*.v))
        # Explicitly list the Verilog files for synthesis
        export VERILOG_FILES = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/vsdbabysoc.v \
                            $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/rvmyth.v \
                            $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/clk_gate.v
    
        export SDC_FILE      = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/vsdbabysoc_synthesis.sdc
    
        export MACRO_PLACEMENT_TCL = $(DESIGN_HOME)/sky130hd/vsdbabysoc/macro.tcl
    
        export vsdbabysoc_DIR = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)
    
        export VERILOG_INCLUDE_DIRS = $(wildcard $(vsdbabysoc_DIR)/include/)
    
        export ADDITIONAL_GDS = $(wildcard $(vsdbabysoc_DIR)/gds/*.gds)
        export ADDITIONAL_LEFS = $(wildcard $(vsdbabysoc_DIR)/lef/*.lef)
        export ADDITIONAL_LIBS = $(wildcard $(vsdbabysoc_DIR)/lib/*.lib)
        #export PDN_TCL = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/pdn.tcl
    
    
        # Clock Configuration
        #export CLOCK_PERIOD = 11.00
        export CLOCK_PORT = CLK
        export CLOCK_NET  = $(CLOCK_PORT)
    
    
        # Pin Order and Macro Placement Configurations
        export FP_PIN_ORDER_CFG = $(vsdbabysoc_DIR)/pin_order.cfg
        export MACRO_PLACEMENT_CFG = $(vsdbabysoc_DIR)/macro.cfg
    
        # Floorplanning Configuration
        export DIE_AREA   = 0 0  2000 2000
        export CORE_AREA  = 20 20 1990 1990
    
    
        # Placement Configuration
        export PLACE_PINS_ARGS = -exclude left:0-600 -exclude left:1000-1600 -exclude right:* -exclude top:* -exclude bottom:*
    
        # Tuning for Timing and Buffers
        export TNS_END_PERCENT     = 100
        export REMOVE_ABC_BUFFERS  = 1
    
        # CTS tuning
        export CTS_BUF_DISTANCE = 600
        export SKIP_GATE_CLONING = 1
    
        # Magic Tool Configuration
        export MAGIC_ZEROIZE_ORIGIN = 0
        export MAGIC_EXT_USE_GDS    = 1
    
        # export CORE_UTILIZATION=0.1
        # # Set utilization to 10% for core sizing
        #export CORE_UTILIZATION = 0.1
    
        # Set place density to match, controlling local packing
        #export PLACE_DENSITY = 0.3
        # # Reduce this value to allow more whitespace for routing.# Reduce this value to allow more whitespace for routing.


## FloorPlan:

### FloorPlan Result:
    
    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan

![](img/floorplan2_1.png)
![](img/floorplan2_2.png)

- in this floorplan the pll is place right side of the dac 

![](img/floorplan2_3.png)


---

## Placement of VSDBabySoC:


### Placement Result Layout:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place

![](img/place2_1.png)
![](img/place2_2.png)



---

## Clock Tree Synthesis (cts):

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts


### Clock Tree Synthesis Result:

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts

![](img/cts2_1.png)
![](img/cts2_2.png)
![](img/cts2_3.png)


### Clock Tree Viewer:

![](img/cts2_8.png)

- **Top red marker** : The main clock source (clk) where the clock enters the design.
- **Long blue vertical line** : The primary line that carries the clock before it branches.
- **Green horizontal segment** : The first big split where CTS adds buffers to evenly distribute the clock.
- **Multiple blue downward lines** : Branches of the clock tree sending the clock signal to different flop groups; equal heights mean balanced timing.
- **Small red markers at the bottom** : Final clock endpoints showing where the clock is delivered. Here every point are mostly reaches at same time whihc represent less slew .

### Setup Report:

![](img/cts2_6.png)

- it is clearly seen that required time > arrival time that is the signal arrives before the required time.

- the slack are postive slacks so the cts report for setup is met the timing contrain very well.

### Hold Report:

![](img/cts2_7.png)

- here the required time < arrival time and ppsotive slack so, the hold constraints also well met.

### Setup Slack Chart:

![](img/cts2_4.png)

- Most endpoints have setup slack between 6 ns – 9 ns.

- Some endpoints have even more slack around 10–11 ns.

-  no negative slack, meaning no setup violations.

### Hold Slack Chart:

![](img/cts2_5.png)

- Most endpoints have hold slack between 0.4 ns – 0.6 ns.

- A smaller number stretch between 0.6 ns – 1.5 ns and a few up to ~1.6 ns.

- No negative slack, which means there are no hold violations after CTS

- *cts final report:*

            ==========================================================================
        cts final report_tns
        --------------------------------------------------------------------------
        tns max 0.00
        
        ==========================================================================
        cts final report_wns
        --------------------------------------------------------------------------
        wns max 0.00
        
        ==========================================================================
        cts final report_worst_slack
        --------------------------------------------------------------------------
        worst slack max 6.37
        
        ==========================================================================
        cts final report_clock_min_period
        --------------------------------------------------------------------------
        clk period_min = 4.63 fmax = 216.00
        
        ==========================================================================
        cts final report_clock_skew
        --------------------------------------------------------------------------
        Clock clk
           0.83 source latency core.CPU_Xreg_value_a5[17][7]$DFF_P/CLK ^
          -0.99 target latency core.OUT[7]$DFF_P/CLK ^
           0.00 CRPR
        --------------
          -0.17 setup skew
        
        
        ==========================================================================
        cts final report_checks -path_delay min
        --------------------------------------------------------------------------
        Startpoint: core.CPU_reset_a2$DFF_P
                    (rising edge-triggered flip-flop clocked by clk)
        Endpoint: core.CPU_reset_a3$DFF_P
                  (rising edge-triggered flip-flop clocked by clk)
        Path Group: clk
        Path Type: min
        
        Fanout     Cap    Slew   Delay    Time   Description
        -----------------------------------------------------------------------------
                                  0.00    0.00   clock clk (rise edge)
                                  0.00    0.00   clock source latency
             1    0.34    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                                 CLK (net)
                          0.06    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             8    0.26    0.27    0.30    0.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_0_CLK (net)
                          0.27    0.01    0.34 ^ clkbuf_3_0_f_CLK/A (sky130_fd_sc_hd_clkbuf_16)
             9    0.15    0.17    0.30    0.64 ^ clkbuf_3_0_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
                                                 clknet_3_0__leaf_CLK (net)
                          0.17    0.00    0.64 ^ clkbuf_leaf_2_CLK/A (sky130_fd_sc_hd__clkbuf_16)
            12    0.04    0.06    0.19    0.83 ^ clkbuf_leaf_2_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_leaf_2_CLK (net)
                          0.06    0.00    0.83 ^ core.CPU_reset_a2$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_1)
             1    0.00    0.02    0.29    1.12 v core.CPU_reset_a2$DFF_P/Q (sky130_fd_sc_hd__dfxtp_1)
                                                 core.CPU_reset_a2 (net)
                          0.02    0.00    1.12 v core.CPU_reset_a3$DFF_P/D (sky130_fd_sc_hd__dfxtp_4)
                                          1.12   data arrival time
        
                                  0.00    0.00   clock clk (rise edge)
                                  0.00    0.00   clock source latency
             1    0.34    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                                 CLK (net)
                          0.06    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             8    0.26    0.27    0.30    0.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_0_CLK (net)
                          0.27    0.01    0.34 ^ clkbuf_3_0_f_CLK/A (sky130_fd_sc_hd_clkbuf_16)
             9    0.15    0.17    0.30    0.64 ^ clkbuf_3_0_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
                                                 clknet_3_0__leaf_CLK (net)
                          0.17    0.00    0.64 ^ clkbuf_leaf_2_CLK/A (sky130_fd_sc_hd__clkbuf_16)
            12    0.04    0.06    0.19    0.83 ^ clkbuf_leaf_2_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_leaf_2_CLK (net)
                          0.06    0.00    0.83 ^ core.CPU_reset_a3$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_4)
                                  0.00    0.83   clock reconvergence pessimism
                                 -0.03    0.79   library hold time
                                          0.79   data required time
        -----------------------------------------------------------------------------
                                          0.79   data required time
                                         -1.12   data arrival time
        -----------------------------------------------------------------------------
                                          0.32   slack (MET)
        
        
        
        ==========================================================================
        cts final report_checks -path_delay max
        --------------------------------------------------------------------------
        Startpoint: core.CPU_src1_value_a3[12]$DFF_P
                    (rising edge-triggered flip-flop clocked by clk)
        Endpoint: core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P
                  (rising edge-triggered flip-flop clocked by clk)
        Path Group: clk
        Path Type: max
        
        Fanout     Cap    Slew   Delay    Time   Description
        -----------------------------------------------------------------------------
                                  0.00    0.00   clock clk (rise edge)
                                  0.00    0.00   clock source latency
             1    0.34    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                                 CLK (net)
                          0.06    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             8    0.26    0.27    0.30    0.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_0_CLK (net)
                          0.27    0.01    0.34 ^ clkbuf_3_3_f_CLK/A (sky130_fd_sc_hd_clkbuf_16)
            14    0.24    0.24    0.35    0.69 ^ clkbuf_3_3_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
                                                 clknet_3_3__leaf_CLK (net)
                          0.25    0.00    0.70 ^ clkbuf_leaf_53_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             6    0.05    0.07    0.21    0.91 ^ clkbuf_leaf_53_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_leaf_53_CLK (net)
                          0.07    0.00    0.91 ^ core.CPU_src1_value_a3[12]$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_1)
             7    0.04    0.38    0.55    1.46 ^ core.CPU_src1_value_a3[12]$DFF_P/Q (sky130_fd_sc_hd__dfxtp_1)
                                                 core.CPU_src1_value_a3[12] (net)
                          0.38    0.00    1.47 ^ 10830/A (sky130_fd_sc_hd__ha_1)
            15    0.05    0.26    0.48    1.94 v 10830/SUM (sky130_fd_sc_hd__ha_1)
                                                 00098 (net)
                          0.26    0.00    1.94 v 07812/B (sky130_fd_sc_hd__nor4_1)
             3    0.01    0.50    0.53    2.47 ^ 07812/Y (sky130_fd_sc_hd__nor4_1)
                                                 02899 (net)
                          0.50    0.00    2.47 ^ 07813/B (sky130_fd_sc_hd__nand2_1)
             1    0.00    0.10    0.13    2.60 v 07813/Y (sky130_fd_sc_hd__nand2_1)
                                                 02900 (net)
                          0.10    0.00    2.60 v 07814/C1 (sky130_fd_sc_hd__a211oi_1)
             2    0.02    0.57    0.48    3.08 ^ 07814/Y (sky130_fd_sc_hd__a211oi_1)
                                                 02901 (net)
                          0.57    0.00    3.08 ^ 08170/A (sky130_fd_sc_hd__nor2_1)
             1    0.00    0.10    0.10    3.18 v 08170/Y (sky130_fd_sc_hd__nor2_1)
                                                 03248 (net)
                          0.10    0.00    3.18 v 08174/A2 (sky130_fd_sc_hd__a21oi_1)
             3    0.03    0.60    0.52    3.70 ^ 08174/Y (sky130_fd_sc_hd__a21oi_1)
                                                 03252 (net)
                          0.60    0.00    3.70 ^ 08325/A (sky130_fd_sc_hd__nand2_1)
             2    0.01    0.14    0.14    3.85 v 08325/Y (sky130_fd_sc_hd__nand2_1)
                                                 03399 (net)
                          0.14    0.00    3.85 v 08329/B (sky130_fd_sc_hd__and3_1)
             1    0.00    0.04    0.21    4.05 v 08329/X (sky130_fd_sc_hd__and3_1)
                                                 03403 (net)
                          0.04    0.00    4.05 v 08333/A3 (sky130_fd_sc_hd__o32ai_1)
             3    0.02    0.66    0.56    4.61 ^ 08333/Y (sky130_fd_sc_hd__o32ai_1)
                                                 03407 (net)
                          0.66    0.00    4.61 ^ 08357/A (sky130_fd_sc_hd__nand2_1)
            12    0.05    0.39    0.47    5.08 v 08357/Y (sky130_fd_sc_hd__nand2_1)
                                                 03431 (net)
                          0.39    0.00    5.08 v 08955/A2 (sky130_fd_sc_hd__a311oi_1)
             1    0.00    0.22    0.37    5.45 ^ 08955/Y (sky130_fd_sc_hd__a311oi_1)
                                                 00867 (net)
                          0.22    0.00    5.45 ^ core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P/D (sky130_fd_sc_hd__dfxtp_1)
                                          5.45   data arrival time
        
                                 11.00   11.00   clock clk (rise edge)
                                  0.00   11.00   clock source latency
             1    0.34    0.00    0.00   11.00 ^ pll/CLK (avsdpll)
                                                 CLK (net)
                          0.06    0.03   11.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             8    0.26    0.27    0.30   11.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_0_CLK (net)
                          0.27    0.01   11.34 ^ clkbuf_3_3_f_CLK/A (sky130_fd_sc_hd_clkbuf_16)
            14    0.24    0.24    0.35   11.69 ^ clkbuf_3_3_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
                                                 clknet_3_3__leaf_CLK (net)
                          0.25    0.00   11.70 ^ clkbuf_leaf_38_CLK/A (sky130_fd_sc_hd__clkbuf_16)
            14    0.04    0.07    0.21   11.91 ^ clkbuf_leaf_38_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_leaf_38_CLK (net)
                          0.07    0.00   11.91 ^ core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P/CLK (sky130_fd_sc_hd__dfxtp_1)
                                  0.00   11.91   clock reconvergence pessimism
                                 -0.09   11.82   library setup time
                                         11.82   data required time
        -----------------------------------------------------------------------------
                                         11.82   data required time
                                         -5.45   data arrival time
        -----------------------------------------------------------------------------
                                          6.37   slack (MET)
        
        
        
        ==========================================================================
        cts final report_checks -unconstrained
        --------------------------------------------------------------------------
        Startpoint: core.CPU_src1_value_a3[12]$DFF_P
                    (rising edge-triggered flip-flop clocked by clk)
        Endpoint: core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P
                  (rising edge-triggered flip-flop clocked by clk)
        Path Group: clk
        Path Type: max
        
        Fanout     Cap    Slew   Delay    Time   Description
        -----------------------------------------------------------------------------
                                  0.00    0.00   clock clk (rise edge)
                                  0.00    0.00   clock source latency
             1    0.34    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                                 CLK (net)
                          0.06    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             8    0.26    0.27    0.30    0.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_0_CLK (net)
                          0.27    0.01    0.34 ^ clkbuf_3_3_f_CLK/A (sky130_fd_sc_hd_clkbuf_16)
            14    0.24    0.24    0.35    0.69 ^ clkbuf_3_3_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
                                                 clknet_3_3__leaf_CLK (net)
                          0.25    0.00    0.70 ^ clkbuf_leaf_53_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             6    0.05    0.07    0.21    0.91 ^ clkbuf_leaf_53_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_leaf_53_CLK (net)
                          0.07    0.00    0.91 ^ core.CPU_src1_value_a3[12]$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_1)
             7    0.04    0.38    0.55    1.46 ^ core.CPU_src1_value_a3[12]$DFF_P/Q (sky130_fd_sc_hd__dfxtp_1)
                                                 core.CPU_src1_value_a3[12] (net)
                          0.38    0.00    1.47 ^ 10830/A (sky130_fd_sc_hd__ha_1)
            15    0.05    0.26    0.48    1.94 v 10830/SUM (sky130_fd_sc_hd__ha_1)
                                                 00098 (net)
                          0.26    0.00    1.94 v 07812/B (sky130_fd_sc_hd__nor4_1)
             3    0.01    0.50    0.53    2.47 ^ 07812/Y (sky130_fd_sc_hd__nor4_1)
                                                 02899 (net)
                          0.50    0.00    2.47 ^ 07813/B (sky130_fd_sc_hd__nand2_1)
             1    0.00    0.10    0.13    2.60 v 07813/Y (sky130_fd_sc_hd__nand2_1)
                                                 02900 (net)
                          0.10    0.00    2.60 v 07814/C1 (sky130_fd_sc_hd__a211oi_1)
             2    0.02    0.57    0.48    3.08 ^ 07814/Y (sky130_fd_sc_hd__a211oi_1)
                                                 02901 (net)
                          0.57    0.00    3.08 ^ 08170/A (sky130_fd_sc_hd__nor2_1)
             1    0.00    0.10    0.10    3.18 v 08170/Y (sky130_fd_sc_hd__nor2_1)
                                                 03248 (net)
                          0.10    0.00    3.18 v 08174/A2 (sky130_fd_sc_hd__a21oi_1)
             3    0.03    0.60    0.52    3.70 ^ 08174/Y (sky130_fd_sc_hd__a21oi_1)
                                                 03252 (net)
                          0.60    0.00    3.70 ^ 08325/A (sky130_fd_sc_hd__nand2_1)
             2    0.01    0.14    0.14    3.85 v 08325/Y (sky130_fd_sc_hd__nand2_1)
                                                 03399 (net)
                          0.14    0.00    3.85 v 08329/B (sky130_fd_sc_hd__and3_1)
             1    0.00    0.04    0.21    4.05 v 08329/X (sky130_fd_sc_hd__and3_1)
                                                 03403 (net)
                          0.04    0.00    4.05 v 08333/A3 (sky130_fd_sc_hd__o32ai_1)
             3    0.02    0.66    0.56    4.61 ^ 08333/Y (sky130_fd_sc_hd__o32ai_1)
                                                 03407 (net)
                          0.66    0.00    4.61 ^ 08357/A (sky130_fd_sc_hd__nand2_1)
            12    0.05    0.39    0.47    5.08 v 08357/Y (sky130_fd_sc_hd__nand2_1)
                                                 03431 (net)
                          0.39    0.00    5.08 v 08955/A2 (sky130_fd_sc_hd__a311oi_1)
             1    0.00    0.22    0.37    5.45 ^ 08955/Y (sky130_fd_sc_hd__a311oi_1)
                                                 00867 (net)
                          0.22    0.00    5.45 ^ core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P/D (sky130_fd_sc_hd__dfxtp_1)
                                          5.45   data arrival time
        
                                 11.00   11.00   clock clk (rise edge)
                                  0.00   11.00   clock source latency
             1    0.34    0.00    0.00   11.00 ^ pll/CLK (avsdpll)
                                                 CLK (net)
                          0.06    0.03   11.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
             8    0.26    0.27    0.30   11.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_0_CLK (net)
                          0.27    0.01   11.34 ^ clkbuf_3_3_f_CLK/A (sky130_fd_sc_hd_clkbuf_16)
            14    0.24    0.24    0.35   11.69 ^ clkbuf_3_3_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
                                                 clknet_3_3__leaf_CLK (net)
                          0.25    0.00   11.70 ^ clkbuf_leaf_38_CLK/A (sky130_fd_sc_hd__clkbuf_16)
            14    0.04    0.07    0.21   11.91 ^ clkbuf_leaf_38_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                                 clknet_leaf_38_CLK (net)
                          0.07    0.00   11.91 ^ core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P/CLK (sky130_fd_sc_hd__dfxtp_1)
                                  0.00   11.91   clock reconvergence pessimism
                                 -0.09   11.82   library setup time
                                         11.82   data required time
        -----------------------------------------------------------------------------
                                         11.82   data required time
                                         -5.45   data arrival time
        -----------------------------------------------------------------------------
                                          6.37   slack (MET)
        
        
        
        ==========================================================================
        cts final report_check_types -max_slew -max_cap -max_fanout -violators
        --------------------------------------------------------------------------
        
        ==========================================================================
        cts final max_slew_check_slack
        --------------------------------------------------------------------------
        0.19928325712680817
        
        ==========================================================================
        cts final max_slew_check_limit
        --------------------------------------------------------------------------
        1.4979510307312012
        
        ==========================================================================
        cts final max_slew_check_slack_limit
        --------------------------------------------------------------------------
        0.1330
        
        ==========================================================================
        cts final max_fanout_check_slack
        --------------------------------------------------------------------------
        1.0000000150474662e+30
        
        ==========================================================================
        cts final max_fanout_check_limit
        --------------------------------------------------------------------------
        1.0000000150474662e+30
        
        ==========================================================================
        cts final max_capacitance_check_slack
        --------------------------------------------------------------------------
        0.009770261123776436
        
        ==========================================================================
        cts final max_capacitance_check_limit
        --------------------------------------------------------------------------
        0.021067000925540924
        
        ==========================================================================
        cts final max_capacitance_check_slack_limit
        --------------------------------------------------------------------------
        0.4638
        
        ==========================================================================
        cts final max_slew_violation_count
        --------------------------------------------------------------------------
        max slew violation count 0
        
        ==========================================================================
        cts final max_fanout_violation_count
        --------------------------------------------------------------------------
        max fanout violation count 0
        
        ==========================================================================
        cts final max_cap_violation_count
        --------------------------------------------------------------------------
        max cap violation count 0
        
        ==========================================================================
        cts final setup_violation_count
        --------------------------------------------------------------------------
        setup violation count 0
        
        ==========================================================================
        cts final hold_violation_count
        --------------------------------------------------------------------------
        hold violation count 0
        
        ==========================================================================
        cts final report_checks -path_delay max reg to reg
        --------------------------------------------------------------------------
        Startpoint: core.CPU_src1_value_a3[12]$DFF_P
                    (rising edge-triggered flip-flop clocked by clk)
        Endpoint: core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P
                  (rising edge-triggered flip-flop clocked by clk)
        Path Group: clk
        Path Type: max
        
          Delay    Time   Description
        ---------------------------------------------------------
           0.00    0.00   clock clk (rise edge)
           0.00    0.00   clock source latency
           0.00    0.00 ^ pll/CLK (avsdpll)
           0.33    0.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.36    0.69 ^ clkbuf_3_3_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
           0.22    0.91 ^ clkbuf_leaf_53_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.00    0.91 ^ core.CPU_src1_value_a3[12]$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_1)
           0.55    1.46 ^ core.CPU_src1_value_a3[12]$DFF_P/Q (sky130_fd_sc_hd__dfxtp_1)
           0.48    1.94 v 10830/SUM (sky130_fd_sc_hd__ha_1)
           0.53    2.47 ^ 07812/Y (sky130_fd_sc_hd__nor4_1)
           0.13    2.60 v 07813/Y (sky130_fd_sc_hd__nand2_1)
           0.48    3.08 ^ 07814/Y (sky130_fd_sc_hd__a211oi_1)
           0.10    3.18 v 08170/Y (sky130_fd_sc_hd__nor2_1)
           0.52    3.70 ^ 08174/Y (sky130_fd_sc_hd__a21oi_1)
           0.14    3.85 v 08325/Y (sky130_fd_sc_hd__nand2_1)
           0.21    4.05 v 08329/X (sky130_fd_sc_hd__and3_1)
           0.56    4.61 ^ 08333/Y (sky130_fd_sc_hd__o32ai_1)
           0.47    5.08 v 08357/Y (sky130_fd_sc_hd__nand2_1)
           0.37    5.45 ^ 08955/Y (sky130_fd_sc_hd__a311oi_1)
           0.00    5.45 ^ core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P/D (sky130_fd_sc_hd__dfxtp_1)
                   5.45   data arrival time
        
          11.00   11.00   clock clk (rise edge)
           0.00   11.00   clock source latency
           0.00   11.00 ^ pll/CLK (avsdpll)
           0.33   11.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.36   11.69 ^ clkbuf_3_3_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
           0.21   11.91 ^ clkbuf_leaf_38_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.00   11.91 ^ core.CPU_Xreg_value_a4[17][27]$SDFFE_PP0P/CLK (sky130_fd_sc_hd__dfxtp_1)
           0.00   11.91   clock reconvergence pessimism
          -0.09   11.82   library setup time
                  11.82   data required time
        ---------------------------------------------------------
                  11.82   data required time
                  -5.45   data arrival time
        ---------------------------------------------------------
                   6.37   slack (MET)
        
        
        
        ==========================================================================
        cts final report_checks -path_delay min reg to reg
        --------------------------------------------------------------------------
        Startpoint: core.CPU_reset_a2$DFF_P
                    (rising edge-triggered flip-flop clocked by clk)
        Endpoint: core.CPU_reset_a3$DFF_P
                  (rising edge-triggered flip-flop clocked by clk)
        Path Group: clk
        Path Type: min
        
          Delay    Time   Description
        ---------------------------------------------------------
           0.00    0.00   clock clk (rise edge)
           0.00    0.00   clock source latency
           0.00    0.00 ^ pll/CLK (avsdpll)
           0.33    0.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.31    0.64 ^ clkbuf_3_0_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
           0.19    0.83 ^ clkbuf_leaf_2_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.00    0.83 ^ core.CPU_reset_a2$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_1)
           0.29    1.12 v core.CPU_reset_a2$DFF_P/Q (sky130_fd_sc_hd__dfxtp_1)
           0.00    1.12 v core.CPU_reset_a3$DFF_P/D (sky130_fd_sc_hd__dfxtp_4)
                   1.12   data arrival time
        
           0.00    0.00   clock clk (rise edge)
           0.00    0.00   clock source latency
           0.00    0.00 ^ pll/CLK (avsdpll)
           0.33    0.33 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.31    0.64 ^ clkbuf_3_0_f_CLK/X (sky130_fd_sc_hd_clkbuf_16)
           0.19    0.83 ^ clkbuf_leaf_2_CLK/X (sky130_fd_sc_hd__clkbuf_16)
           0.00    0.83 ^ core.CPU_reset_a3$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_4)
           0.00    0.83   clock reconvergence pessimism
          -0.03    0.79   library hold time
                   0.79   data required time
        ---------------------------------------------------------
                   0.79   data required time
                  -1.12   data arrival time
        ---------------------------------------------------------
                   0.32   slack (MET)
        
        
        
        ==========================================================================
        cts final critical path target clock latency max path
        --------------------------------------------------------------------------
        0
        
        ==========================================================================
        cts final critical path target clock latency min path
        --------------------------------------------------------------------------
        0
        
        ==========================================================================
        cts final critical path source clock latency min path
        --------------------------------------------------------------------------
        0
        
        ==========================================================================
        cts final critical path delay
        --------------------------------------------------------------------------
        5.4461
        
        ==========================================================================
        cts final critical path slack
        --------------------------------------------------------------------------
        6.3704
        
        ==========================================================================
        cts final slack div critical path delay
        --------------------------------------------------------------------------
        116.971778
        
        ==========================================================================
        cts final report_power
        --------------------------------------------------------------------------
        Group                  Internal  Switching    Leakage      Total
                                  Power      Power      Power      Power (Watts)
        ----------------------------------------------------------------
        Sequential             4.37e-03   3.45e-04   9.26e-09   4.71e-03  40.4%
        Combinational          8.35e-04   1.90e-03   9.62e-09   2.74e-03  23.5%
        Clock                  2.26e-03   1.95e-03   1.88e-09   4.22e-03  36.1%
        Macro                  0.00e+00   0.00e+00   0.00e+00   0.00e+00   0.0%
        Pad                    0.00e+00   0.00e+00   0.00e+00   0.00e+00   0.0%
        ----------------------------------------------------------------
        Total                  7.46e-03   4.20e-03   2.08e-08   1.17e-02 100.0%
                                  64.0%      36.0%       0.0%

---

## Routing

    make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route

![](img/route_c1.png)

### Routing Gui:

![](img/route1.png)
![](img/route2.png)
![](img/route3.png)
![](img/route4.png)


### Final Report Check:

    report_checks

![](img/total_script.png)

    OpenROAD v2.0-26087-g3bcda7705d 
    Features included (+) or not (-): +GPU +GUI +Python
    This program is licensed under the BSD-3 license. See the LICENSE file for details.
    Components of this program may be licensed under more restrictive licenses which must be honored.
    read_liberty /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/platforms/sky130hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_liberty /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/avsddac.lib
    read_liberty /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/avsdpll.lib
    read_liberty /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    [WARNING STA-1140] /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/sky130_fd_sc_hd_tt_025C_1v80.lib line 1, library sky130_fd_sc_hd_tt_025C_1v80 already exists.
    [WARNING STA-1173] /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/sky130_fd_sc_hd__tt_025C_1v80.lib line 23, default_fanout_load is 0.0.
    read_db ./results/sky130hd/vsdbabysoc/base/5_route.odb
    GUI_TIMING=1 reading timing, takes a little while for large designs...
    read_sdc /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.sdc
    grt::have_routes
    estimate_parasitics -global_routing
    sta::find_timing
    sta::find_requireds
    gui::select_chart "Endpoint Slack"
    gui::update_timing_report
    >>> report_checks
    Startpoint: core.CPU_src1_value_a3[16]$DFF_P
                (rising edge-triggered flip-flop clocked by clk)
    Endpoint: core.CPU_Xreg_value_a4[17][28]$SDFFE_PP0P
              (rising edge-triggered flip-flop clocked by clk)
    Path Group: clk
    Path Type: max
    
      Delay    Time   Description
    ---------------------------------------------------------
       0.00    0.00   clock clk (rise edge)
       0.88    0.88   clock network delay (propagated)
       0.00    0.88 ^ core.CPU_src1_value_a3[16]$DFF_P/CLK (sky130_fd_sc_hd__dfxtp_1)
       0.44    1.32 v core.CPU_src1_value_a3[16]$DFF_P/Q (sky130_fd_sc_hd__dfxtp_1)
       0.73    2.04 ^ 10843/SUM (sky130_fd_sc_hd__ha_1)
       0.38    2.43 v 05444/Y (sky130_fd_sc_hd__nand4_1)
       0.43    2.86 ^ 08027/Y (sky130_fd_sc_hd__nor4_1)
       0.33    3.18 v 08028/Y (sky130_fd_sc_hd__o311ai_0)
       0.72    3.90 ^ 08206/Y (sky130_fd_sc_hd__a311oi_1)
       0.20    4.10 v 08381/Y (sky130_fd_sc_hd__a31oi_1)
       0.21    4.31 ^ 08382/Y (sky130_fd_sc_hd__mux2i_1)
       0.22    4.53 v 08384/Y (sky130_fd_sc_hd__nand3b_1)
       0.83    5.36 ^ 08401/Y (sky130_fd_sc_hd__o21ai_1)
       0.19    5.55 v 08956/Y (sky130_fd_sc_hd__nand3_1)
       0.20    5.75 ^ 08958/Y (sky130_fd_sc_hd__a21oi_1)
       0.00    5.75 ^ core.CPU_Xreg_value_a4[17][28]$SDFFE_PP0P/D (sky130_fd_sc_hd__dfxtp_1)
               5.75   data arrival time
    
      11.00   11.00   clock clk (rise edge)
       0.87   11.87   clock network delay (propagated)
       0.00   11.87   clock reconvergence pessimism
              11.87 ^ core.CPU_Xreg_value_a4[17][28]$SDFFE_PP0P/CLK (sky130_fd_sc_hd__dfxtp_1)
      -0.07   11.80   library setup time
              11.80   data required time
    ---------------------------------------------------------
              11.80   data required time
              -5.75   data arrival time
    ---------------------------------------------------------
               6.05   slack (MET)


### Reports After Synthesis to Routing:

![](img/reports_fin.png)

### Logs After Synthesis to Routing:

![](img/logs_fin.png)

###  Convert .odb file to .def file

    cd flow
    openroad

![](img/results_b.png)

- in results we have route.odb file which is converted to .def file

        # Loads the routed design database into OpenROAD
        read_db ~/Desktop/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_2_route.odb
    
        # Exports the loaded design as a DEF file
        write_def ~/Desktop/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_2_route.def

![](img/results_a.png)

- opening .def file

![](img/file_o.png)

### Generating Post-Route SPEF and Final Netlist for VSDBabySoC:

- This section shows the steps to generate the post-route SPEF and the final Verilog netlist for the VSDBabySoC design using OpenROAD. These files are extracted after routing and used in later analysis stages.

        #Cd to flow and run openroad
    
        cd flow/
        openroad
    
        #Loading Technology and Macro LEF Files
    
        read_lef ~/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/sky130hd.lef
    
        read_lef ~/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsdpll.lef
    
        read_lef ~/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsddac.lef
    
        # reading lib files which contains timing and power information about standard cells
    
        read_liberty /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/platforms/sky130hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    
        # loading def file 
    
        read_def /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_2_route.def

![](img/def1.png)



## RC Extraction & Output Generation :

### Cloning the Open_PDKs Repository:

    git clone https://github.com/RTimothyEdwards/open_pdks.git


- This command downloads the open_pdks repository, which provides the process design kits (PDKs) required for Sky130. It is included here because OpenROAD and OpenLane use these PDK files for tasks such as parasitic extraction, device modeling, and layout-related calculations.


### Command:

    # Sets the process corner using the specified RC extraction model
    define_process_corner -ext_model_index 0 /home/ritesh/Desktop/OpenROAD-flow-scripts/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre

    # Runs parasitic extraction using the given RC model file
    extract_parasitics -ext_model_file /home/ritesh/Desktop/OpenROAD-flow-scripts/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre

    # Saves the extracted parasitics into a SPEF file
    write_spef /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc.spef

    # Exports the final post-placement Verilog netlist
    write_verilog /home/ritesh/Desktop/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc_post_place.v


![](img/spef1.png)

- SPEF File:
The SPEF contains the extracted parasitic resistance and capacitance values from the routed layout. These parasitics affect signal delays, so the SPEF is used during timing analysis (STA) to check whether the chip still meets timing after routing.

- Post-Route Verilog (.v):
The post-route Verilog netlist reflects the final, actual connectivity after placement and routing. It is used for post-layout simulation, timing verification, and signoff checks to ensure the design functions correctly with real routing delays.

### Opening .v and .spef file:

![](img/posr_place.v.png)
![](img/spef2.png)




---

















