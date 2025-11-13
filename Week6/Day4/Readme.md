# Day 4 - Pre-layout timing analysis and Importance of good clock tree


### Inverter check for good Layout:

- **Conditions for a Good Layout**

For a layout to be well-structured and compatible with automated tools, certain conditions must be met.

- The input and output ports of a standard cell should align with the defined vertical and horizontal grid lines.

- The cell width must be an odd multiple of the horizontal grid spacing
- while the cell height should be exactly three times the vertical grid value to maintain uniform placement and routing alignment.

### Opening Layout:

    cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign
    magic -T ./lib/sky130A.tech sky130_inv.mag &

![](img/layout_cons.png)
![](img/inv_sch.png)

- tracks.info file can be used to geth the vertical and horizontal grid value
![](img/box_size.png)

    Horizontal grid length = 0.46 um

    Vertical grid length = 0.34 um

- adding grid to invertes layout to check whether layout satisfies the rule

        grid 0.46um 0.34um 0.23um 0.17um

![](img/grid_set.png)

- **Rule 1** -(i/p and o/p must contain vertical and horizontal line):

![](img/rule1.png)

- the i/p and o/p contains vertical and horizontal grid line so satisfies rule 1

**Rule 2 :**
![](img/rule2.png)

- width = 1.38 um = 0.46 um(horizontal grid spacing)*3 (odd) == (Hence Satisfies Rule 2)

**Rule 3 :**
![](img/rule3.png)
- height = 2.72 um = 0.34 um(veritical grid spacing) * 8(even) == (Hence Satisfies Rule 3)

### save the layout and exit

    save sky130_vsdinv.mag
    exit

### Open the Saved Layout and write lef file:

    lef write

![](img/lef_write.png)

- lef file is created inside vsdstdcelldesign directory

![](img/lef_file_ls.png)

## Implementing a Custom Inverter within the PicoRV32a Design Flow

- Copy the .lef files to the picorv32a/src/

        cp sky130_vsdinv.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
    
        cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

- modify the confif.tcl file 

    set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
    set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
    set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
    set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
    set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]

![](img/config_tcl1.png)

### Openlane:

    cd ~/Desktop/work/tools/openlane_working_dir/openlane

    alias docker='docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) efabless/openlane:v0.21'

    docker

    ./flow.tcl -interactive

![](img/openlane.png)

### Command to Synthesis:

    package require openlane 0.9

    prep -design picorv32a -tag 30-10_11-30 -overwrite

    set lefs [glob $::env(DESIGN_DIR)/src/*.lef]

    add_lefs -src $lefs

![](img/inv_openlane.png)

- synthesis
  
        run_synthesis
![](img/inv_synthesis.png)
![](img/inv_synth2.png)
![](img/inv_synth3.png)

- slack value is  Negative slack time occurs when a signal path fails to meet its required timing limit. It shows that the circuit is too slow in that path, potentially leading to setup time violations and unreliable operation.

- improve timing by incresing area which can be done by changing few parameter, the details of parameter can be taken from README.md from ~/Desktop/work/tools/openlane_working_dir/openlane/configuration/

![](img/synth_readme.png)

### Synthesis by changing parameter:

    prep -design picorv32a -tag new -overwrite

    set lefs [glob $::env(DESIGN_DIR)/src/*.lef]

    add_lefs -src $lefs

    echo $::env(SYNTH_STRATEGY)

    set ::env(SYNTH_STRATEGY) "DELAY 3"

    echo $::env(SYNTH_BUFFERING)

    echo $::env(SYNTH_SIZING)

    set ::env(SYNTH_SIZING) 1

    echo $::env(SYNTH_DRIVING_CELL)

![](img/changepara_invsynth.png)
![](img/synth2_inv.png)

- it is cleary seen that are of the chip increased from 147712.918 to 181730.544

181730.544 - 147712.918 = 34,017.626 units(increased)

![](img/synth2_inv_twns.png)

- but tns and wns are changed to 0

    tns=0
    wns=0
## Floor Plan:

- while running floor plan there will be error arrising so we can use the command from readme file from openlane OpenLane_commands.md

![](img/openlane_commands.png)

- Command:

        init_floorplan
        place_io
        tap_decap_or

![](img/floorplan_inv.png)
![](img/floorplan_inv2.png)

## Placement :

    run_placement

![](img/placement_inv.png)

- the result is stored in the picorv32a file in directory /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/new/results/placement/

        cd /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/new/results/placement/
        magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
        lef read ../../tmp/merged.lef def read picorv32a.placement.def &

![](img/inv_place1.png)

![](img/inv_place2.png)
 
- the inverter will be present inside this layout 

![](img/inv_what.png)

- to view the inverter coplete layout we can use **expand** command
![](img/inv_expand.png)

## Performing Timing Checks with OpenSTA

- Static Timing Analysis (STA) is a method used to evaluate the timing performance of a circuit without running simulations. It checks all possible paths to ensure signals meet setup and hold time requirements for reliable operation.

- intialize docker and run openlane
  
        cd ~/Desktop/work/tools/openlane_working_dir/openlane
    
        alias docker='docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT \
        -u $(id -u $USER):$(id -g $USER) efabless/openlane:v0.21'
    
        docker

- synthesis 

        ./flow.tcl -interactive
    
        package require openlane 0.9
    
        prep -design picorv32a
    
        set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
        add_lefs -src $lefs
    
        set ::env(SYNTH_SIZING) 1
    
        run_synthesis

- load capacitance, fanout, and other parameters are determined from the sky130_fd_sc_hd__typical.lib located under openlane/vsdstdcelldesign/libs/ directory.

![](img/sky_typical.png)


![](img/ta1.png)

- creation of pre_sta.conf file for run sta :

![](img/presta.png)

- create my_base.sdc under  directory openlane/designs/picorv32a/src 

![](img/my_base.png)

### Running Static Timing Analysis (STA) through OpenSTA:

    cd ~/Desktop/work/tools/openlane_working_dir/openlane
    sta pre_sta.conf

![](img/slack1.png)
![](img/slack1r1.png)
![](img/slack1r2.png)
![](img/slack1r3.png)

TNS (Total Negative Slack) and WNS (Worst Negative Slack) are negative whihc indicates the timing constraint doesnt meets

- Re Synthesis with adding some parameter

        prep -design picorv32a -tag 11-11_16-13 -overwrite
    
        set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
        add_lefs -src $lefs
    
        # value for SYNTH_SIZING
        set ::env(SYNTH_SIZING) 1
    
        # value for SYNTH_MAX_FANOUT
        set ::env(SYNTH_MAX_FANOUT) 4
    
        echo $::env(SYNTH_DRIVING_CELL)
    
        run_synthesis

![](img/pic_synth_success.png)

    cd ~/Desktop/work/tools/openlane_working_dir/openlane/
    sta pre_sta.conf

![](img/slacktill.png)

- even modifying some parameters like fanout the slack time is negative so we will start to replace some cells whihc is under performing with low speed models.

- slack -23.90
![](img/replace1_1.png)


- The cell sky130_fd_sc_hd__or3_2 represents a 3-input OR gate with a moderate drive strength of 2.
Replacing it with sky130_fd_sc_hd__or3_4, a 4-input OR gate with higher drive strength, allows it to drive larger loads more efficiently.
Higher drive strength means the gate can charge and discharge output capacitances faster.
This reduces the signal delay, improving path timing and thereby decreasing negative slack.

- command 

        report_net -connections _11672_
        replace_cell _14510_ sky130_fd_sc_hd__or3_4

- to get timing details:

        report_checks -fields {net cap slew input_pins fanout} -digits 4

![](img/slack2.png)

- slack time reduce to -23.5092

- let reduce some more to Analysis the impact of it

![](img/replace2.png)
- it can be replaced with faster circuit 

![](img/repalace2_c.png)

### Timing details

![](img/slack3.png)

- slack time = -23.5049

![](img/replace3.png)
- it can be replaced with faster circuit 

![](img/repalace3_c.png)

### Timing details

![](img/slack4.png)

- slack time = -22.9860

![](img/replace5.png)
- it can be replaced with faster circuit 

![](img/repalace5_c.png)

### Timing details

![](img/slack5.png)

- slack time = -22.7765

- fot this 4 reduction of slack time we reduced 

-22.7765 - (-23.9) =  1.1235 (reduced slack time)

- This optimisation is  ECO (Engineering Change Order) changes.


- First, create a backup of the original synthesis netlist file.
Then, save and overwrite it with the updated Verilog netlist containing the modified cell changes. 

        cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-10_11-30/results/synthesis/
    
        cp picorv32a.synthesis.v picorv32a.synthesis_old.v

![](img/synth_copy.png)

- write the modfied file :

        write_verilog ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/11-11_16-13/results/synthesis/picorv32a.synthesis.v

![](img/write_verilog.png)

- open verilog file :

![](img/synth_vchanged.png)

- we can see that the changes made are updated in this file

## Executing Clock Tree Synthesis
    - Clock Tree Synthesis (CTS) creates a balanced path for the clock signal so it reaches all parts of the circuit at nearly the same time. This helps keep the circuit in sync and avoids timing issues.

    - to perform Clock Tree Synthesis,add the following lines to the config.tcl file present in directory  ~/Desktop/work/tools/openlane_working_diropenlane/designs/picorv32a/ 

![](img/config_tcl2.png)

    ### Re-running Synthesis with Updated Configuration

    prep -design picorv32a -tag 30-10_11-30 -overwrite

    set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
    add_lefs -src $lefs

    set ::env(SYNTH_STRATEGY) "DELAY 3"
    set ::env(SYNTH_SIZING) 1

![](img/cts1.png)

- Running Synthesis:

        run_synthesis
![](img/cts2.png)

- tns and wns is 0 

### Floorplan & Placement :

    init_floorplan
    place_io
    tap_decap_or
![](img/cts3.png)

    run_placement
![](img/cts4.png)

### Clock Tree Synthesis

    run_cts

![](img/cts5.png)

![](img/cts6.png)

- after this, .cts file is stored in the results/synthesis/
![](img/cts_file_ls.png)
![](img/cts_v.png)


### Executing Post-CTS Timing Analysis:

    % openroad
    OpenROAD 0.9.0 1415572a73
    This program is licensed under the BSD-3 license. See the LICENSE file for details.
    Components of this program may be licensed under more restrictive licenses which must be honored.
    % read_lef /openLANE_flow/designs/picorv32a/runs/30-10_11-30/tmp/merged.lef
    Notice 0: Reading LEF file:  /openLANE_flow/designs/picorv32a/runs/30-10_11-30/tmp/merged.lef
    Notice 0:     Created 13 technology layers
    Notice 0:     Created 25 technology vias
    Notice 0:     Created 442 library cells
    Notice 0: Finished LEF file:  /openLANE_flow/designs/picorv32a/runs/30-10_11-30/tmp/merged.lef
    % read_def /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/cts/picorv32a.cts.def
    Notice 0: 
    Reading DEF file: /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/cts/picorv32a.cts.def
    Notice 0: Design: picorv32a
    Notice 0:     Created 409 pins.
    Notice 0:     Created 25737 components and 145892 component-terminals.
    Notice 0:     Created 18131 nets and 60130 connections.
    Notice 0: Finished DEF file: /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/cts/picorv32a.cts.def
    % write_db pico_cts.db
    % read_db pico_cts.db
    % read_verilog /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/synthesis/picorv32a.synthesis_cts.v
    % read_liberty $::env(LIB_SYNTH_COMPLETE)
    1
    % link_design picorv32a
    [WARNING ORD-1000] LEF master sky130_fd_sc_hd__tapvpwrvgnd_1 has no liberty cell.
    % read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
    [INFO]: Setting output delay to: 4.946000000000001
    [INFO]: Setting input delay to: 4.946000000000001
    [INFO]: Setting load to: 0.017653
    % set_propagated_clock [all_clocks]
    % help report_checks
    report_checks
    report_checks   [-from from_list|-rise_from from_list|-fall_from from_list] [-through through_list|-rise_through through_list|-fall_through through_list] [-to to_list|-rise_to to_list|-fall_to to_list] [-unconstrained] [-path_delay min|min_rise|min_fall|max|max_rise|max_fall|min_max] [-corner corner_name] [-group_count path_count]  [-endpoint_count path_count] [-unique_paths_to_endpoint] [-slack_max slack_max] [-slack_min slack_min] [-sort_by_slack] [-path_group group_name] [-format full|full_clock|full_clock_expanded|short|end|summary] [-fields [capacitance|slew|input_pin|net]] [-digits digits] [-no_line_splits] [> filename] [>> filename]
    % report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
    Startpoint: 31002 (rising edge-triggered flip-flop clocked by clk)
    Endpoint: 30967 (rising edge-triggered flip-flop clocked by clk)
    Path Group: clk
    Path Type: min

Fanout       Cap      Slew     Delay      Time   Description
-------------------------------------------------------------------------------------
                              0.0000    0.0000   clock clk (rise edge)
                              0.0000    0.0000   clock source latency
                    0.0225    0.0100    0.0100 ^ clk (in)
     1    0.0079                                 clk (net)
                    0.0225    0.0000    0.0100 ^ clkbuf_0_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0283    0.1016    0.1116 ^ clkbuf_0_clk/X (sky130_fd_sc_hd__clkbuf_16)
     2    0.0044                                 clknet_0_clk (net)
                    0.0283    0.0000    0.1116 ^ clkbuf_1_1_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0388    0.0697    0.1814 ^ clkbuf_1_1_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_1_1_0_clk (net)
                    0.0388    0.0000    0.1814 ^ clkbuf_1_1_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0388    0.0734    0.2547 ^ clkbuf_1_1_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_1_1_1_clk (net)
                    0.0388    0.0000    0.2547 ^ clkbuf_1_1_2_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0910    0.3458 ^ clkbuf_1_1_2_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_1_1_2_clk (net)
                    0.0635    0.0000    0.3458 ^ clkbuf_2_3_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0390    0.0811    0.4268 ^ clkbuf_2_3_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_2_3_0_clk (net)
                    0.0390    0.0000    0.4268 ^ clkbuf_2_3_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0388    0.0734    0.5003 ^ clkbuf_2_3_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_2_3_1_clk (net)
                    0.0388    0.0000    0.5003 ^ clkbuf_2_3_2_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0910    0.5913 ^ clkbuf_2_3_2_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_2_3_2_clk (net)
                    0.0635    0.0000    0.5913 ^ clkbuf_3_7_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0390    0.0811    0.6724 ^ clkbuf_3_7_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_3_7_0_clk (net)
                    0.0390    0.0000    0.6724 ^ clkbuf_3_7_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0911    0.7635 ^ clkbuf_3_7_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_3_7_1_clk (net)
                    0.0635    0.0000    0.7635 ^ clkbuf_4_14_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0987    0.8622 ^ clkbuf_4_14_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_4_14_0_clk (net)
                    0.0635    0.0000    0.8622 ^ clkbuf_5_28_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0390    0.0811    0.9433 ^ clkbuf_5_28_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_5_28_0_clk (net)
                    0.0390    0.0000    0.9433 ^ clkbuf_5_28_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.5776    0.4454    1.3886 ^ clkbuf_5_28_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
     7    0.0492                                 clknet_5_28_1_clk (net)
                    0.5776    0.0000    1.3886 ^ clkbuf_leaf_104_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0416    0.2342    1.6229 ^ clkbuf_leaf_104_clk/X (sky130_fd_sc_hd__clkbuf_16)
     2    0.0038                                 clknet_leaf_104_clk (net)
                    0.0416    0.0000    1.6229 ^ 31002/CLK (sky130_fd_sc_hd__dfxtp_1)
                    0.0339    0.2909    1.9138 ^ 31002/Q (sky130_fd_sc_hd__dfxtp_1)
     1    0.0018                                 alu_add_sub[26] (net)
                    0.0339    0.0000    1.9138 ^ 29511/A1 (sky130_fd_sc_hd__mux2_2)
                    0.0322    0.1227    2.0364 ^ 29511/X (sky130_fd_sc_hd__mux2_2)
     1    0.0017                                 alu_out[26] (net)
                    0.0322    0.0000    2.0364 ^ 30967/D (sky130_fd_sc_hd__dfxtp_2)
                                        2.0364   data arrival time

                              0.0000    0.0000   clock clk (rise edge)
                              0.0000    0.0000   clock source latency
                    0.0225    0.0100    0.0100 ^ clk (in)
     1    0.0079                                 clk (net)
                    0.0225    0.0000    0.0100 ^ clkbuf_0_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0283    0.1016    0.1116 ^ clkbuf_0_clk/X (sky130_fd_sc_hd__clkbuf_16)
     2    0.0044                                 clknet_0_clk (net)
                    0.0283    0.0000    0.1116 ^ clkbuf_1_1_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0388    0.0697    0.1814 ^ clkbuf_1_1_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_1_1_0_clk (net)
                    0.0388    0.0000    0.1814 ^ clkbuf_1_1_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0388    0.0734    0.2547 ^ clkbuf_1_1_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_1_1_1_clk (net)
                    0.0388    0.0000    0.2547 ^ clkbuf_1_1_2_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0910    0.3458 ^ clkbuf_1_1_2_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_1_1_2_clk (net)
                    0.0635    0.0000    0.3458 ^ clkbuf_2_2_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0390    0.0811    0.4268 ^ clkbuf_2_2_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_2_2_0_clk (net)
                    0.0390    0.0000    0.4268 ^ clkbuf_2_2_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0388    0.0734    0.5003 ^ clkbuf_2_2_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_2_2_1_clk (net)
                    0.0388    0.0000    0.5003 ^ clkbuf_2_2_2_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0910    0.5913 ^ clkbuf_2_2_2_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_2_2_2_clk (net)
                    0.0635    0.0000    0.5913 ^ clkbuf_3_4_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0390    0.0811    0.6724 ^ clkbuf_3_4_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_3_4_0_clk (net)
                    0.0390    0.0000    0.6724 ^ clkbuf_3_4_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0911    0.7635 ^ clkbuf_3_4_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_3_4_1_clk (net)
                    0.0635    0.0000    0.7635 ^ clkbuf_4_8_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0635    0.0987    0.8622 ^ clkbuf_4_8_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     2    0.0044                                 clknet_4_8_0_clk (net)
                    0.0635    0.0000    0.8622 ^ clkbuf_5_17_0_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    0.0390    0.0811    0.9433 ^ clkbuf_5_17_0_clk/X (sky130_fd_sc_hd__clkbuf_1)
     1    0.0022                                 clknet_5_17_0_clk (net)
                    0.0390    0.0000    0.9433 ^ clkbuf_5_17_1_clk/A (sky130_fd_sc_hd__clkbuf_1)
                    1.0082    0.7429    1.6862 ^ clkbuf_5_17_1_clk/X (sky130_fd_sc_hd__clkbuf_1)
    11    0.0868                                 clknet_5_17_1_clk (net)
                    1.0082    0.0000    1.6862 ^ clkbuf_leaf_170_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0688    0.2991    1.9853 ^ clkbuf_leaf_170_clk/X (sky130_fd_sc_hd__clkbuf_16)
    12    0.0225                                 clknet_leaf_170_clk (net)
                    0.0688    0.0000    1.9853 ^ 30967/CLK (sky130_fd_sc_hd__dfxtp_2)
                              0.0000    1.9853   clock reconvergence pessimism
                             -0.0254    1.9598   library hold time
                                        1.9598   data required time
-------------------------------------------------------------------------------------
                                        1.9598   data required time
                                       -2.0364   data arrival time
-------------------------------------------------------------------------------------
                                        0.0766   slack (MET)


Startpoint: resetn (input port clocked by clk)
Endpoint: mem_la_read (output port clocked by clk)
Path Group: clk
Path Type: max

Fanout       Cap      Slew     Delay      Time   Description
-------------------------------------------------------------------------------------
                              0.0000    0.0000   clock clk (rise edge)
                              0.0000    0.0000   clock network delay (propagated)
                              4.9460    4.9460 ^ input external delay
                    0.0182    0.0064    4.9524 ^ resetn (in)
     1    0.0049                                 resetn (net)
                    0.0182    0.0000    4.9524 ^ input101/A (sky130_fd_sc_hd__buf_6)
                    0.0587    0.1008    5.0532 ^ input101/X (sky130_fd_sc_hd__buf_6)
     7    0.0234                                 net101 (net)
                    0.0587    0.0000    5.0532 ^ 15304/A (sky130_fd_sc_hd__clkbuf_4)
                    0.0934    0.1738    5.2269 ^ 15304/X (sky130_fd_sc_hd__clkbuf_4)
     6    0.0265                                 12638 (net)
                    0.0934    0.0000    5.2269 ^ 17093/C (sky130_fd_sc_hd__nand3_4)
                    0.1284    0.1274    5.3543 v 17093/Y (sky130_fd_sc_hd__nand3_4)
     4    0.0282                                 13857 (net)
                    0.1284    0.0000    5.3543 v 18867/B1 (sky130_fd_sc_hd__a21oi_4)
                    0.0799    0.1239    5.4782 ^ 18867/Y (sky130_fd_sc_hd__a21oi_4)
     1    0.0023                                 net199 (net)
                    0.0799    0.0000    5.4782 ^ output199/A (sky130_fd_sc_hd__clkbuf_2)
                    0.1052    0.1596    5.6378 ^ output199/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0177                                 mem_la_read (net)
                    0.1052    0.0000    5.6378 ^ mem_la_read (out)
                                        5.6378   data arrival time

                             24.7300   24.7300   clock clk (rise edge)
                              0.0000   24.7300   clock network delay (propagated)
                              0.0000   24.7300   clock reconvergence pessimism
                             -4.9460   19.7840   output external delay
                                       19.7840   data required time
-------------------------------------------------------------------------------------
                                       19.7840   data required time
                                       -5.6378   data arrival time
-------------------------------------------------------------------------------------
                                       14.1462   slack (MET)


% Ritesh Balaji S

![](img/cts7.png)
![](img/cts8.png)

### Evaluating Post-CTS Timing by Removing a Clock Buffer in OpenROAD :

![](img/pcts1.png)

    % openroad
    OpenROAD 0.9.0 1415572a73
    This program is licensed under the BSD-3 license. See the LICENSE file for details.
    Components of this program may be licensed under more restrictive licenses which must be honored.
    % read_lef /openLANE_flow/designs/picorv32a/runs30-10_11-30/tmp/merged.lef
    [ERROR ORD-0001] /openLANE_flow/designs/picorv32a/runs30-10_11-30/tmp/merged.lef does not exist.
    ORD-0001
    % read_lef /openLANE_flow/designs/picorv32a/runs/30-10_11-30/tmp/merged.lef
    Notice 0: Reading LEF file:  /openLANE_flow/designs/picorv32a/runs/30-10_11-30/tmp/merged.lef
    Notice 0:     Created 13 technology layers
    Notice 0:     Created 25 technology vias
    Notice 0:     Created 442 library cells
    Notice 0: Finished LEF file:  /openLANE_flow/designs/picorv32a/runs/30-10_11-30/tmp/merged.lef
    % read_def /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/cts/picorv32a.cts.def
    Notice 0: 
    Reading DEF file: /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/cts/picorv32a.cts.def
    Notice 0: Design: picorv32a
    Notice 0:     Created 409 pins.
    Notice 0:     Created 25737 components and 145892 component-terminals.
    Notice 0:     Created 18131 nets and 60130 connections.
    Notice 0: Finished DEF file: /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/cts/picorv32a.cts.def
    % write_db pico_cts1.db
    % read_db pico_cts.db
    % read_verilog /openLANE_flow/designs/picorv32a/runs/30-10_11-30/results/synthesis/picorv32a.synthesis_cts.v
    % read_liberty $::env(LIB_SYNTH_COMPLETE)
    1
    % link_design picorv32a
    [WARNING ORD-1000] LEF master sky130_fd_sc_hd__tapvpwrvgnd_1 has no liberty cell.
    % read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
    [INFO]: Setting output delay to: 4.946000000000001
    [INFO]: Setting input delay to: 4.946000000000001
    [INFO]: Setting load to: 0.017653
    % set_propagated_clock [all_clocks]
    % report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
    
    Startpoint: 31002 (rising edge-triggered flip-flop clocked by clk)
    Endpoint: 30967 (rising edge-triggered flip-flop clocked by clk)
    Path Group: clk
    Path Type: min

Fanout       Cap      Slew     Delay      Time   Description
-------------------------------------------------------------------------------------
                              0.0000    0.0000   clock clk (rise edge)
                              0.0000    0.0000   clock source latency
                    0.0225    0.0100    0.0100 ^ clk (in)
     1    0.0079                                 clk (net)
                    0.0225    0.0000    0.0100 ^ clkbuf_0_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0285    0.1019    0.1119 ^ clkbuf_0_clk/X (sky130_fd_sc_hd__clkbuf_16)
     2    0.0046                                 clknet_0_clk (net)
                    0.0285    0.0000    0.1119 ^ clkbuf_1_1_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0817    0.1936 ^ clkbuf_1_1_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_1_1_0_clk (net)
                    0.0264    0.0000    0.1936 ^ clkbuf_1_1_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0810    0.2746 ^ clkbuf_1_1_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_1_1_1_clk (net)
                    0.0264    0.0000    0.2746 ^ clkbuf_1_1_2_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0913    0.3659 ^ clkbuf_1_1_2_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_1_1_2_clk (net)
                    0.0379    0.0000    0.3659 ^ clkbuf_2_3_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0851    0.4510 ^ clkbuf_2_3_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_2_3_0_clk (net)
                    0.0264    0.0000    0.4510 ^ clkbuf_2_3_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0810    0.5320 ^ clkbuf_2_3_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_2_3_1_clk (net)
                    0.0264    0.0000    0.5320 ^ clkbuf_2_3_2_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0913    0.6233 ^ clkbuf_2_3_2_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_2_3_2_clk (net)
                    0.0379    0.0000    0.6233 ^ clkbuf_3_7_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0851    0.7084 ^ clkbuf_3_7_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_3_7_0_clk (net)
                    0.0264    0.0000    0.7084 ^ clkbuf_3_7_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0913    0.7996 ^ clkbuf_3_7_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_3_7_1_clk (net)
                    0.0379    0.0000    0.7996 ^ clkbuf_4_14_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0953    0.8950 ^ clkbuf_4_14_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_4_14_0_clk (net)
                    0.0379    0.0000    0.8950 ^ clkbuf_5_28_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0851    0.9801 ^ clkbuf_5_28_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_5_28_0_clk (net)
                    0.0264    0.0000    0.9801 ^ clkbuf_5_28_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.2717    0.2543    1.2344 ^ clkbuf_5_28_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
     7    0.0492                                 clknet_5_28_1_clk (net)
                    0.2717    0.0000    1.2344 ^ clkbuf_leaf_104_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0323    0.1817    1.4161 ^ clkbuf_leaf_104_clk/X (sky130_fd_sc_hd__clkbuf_16)
     2    0.0038                                 clknet_leaf_104_clk (net)
                    0.0323    0.0000    1.4161 ^ 31002/CLK (sky130_fd_sc_hd__dfxtp_1)
                    0.0339    0.2874    1.7035 ^ 31002/Q (sky130_fd_sc_hd__dfxtp_1)
     1    0.0018                                 alu_add_sub[26] (net)
                    0.0339    0.0000    1.7035 ^ 29511/A1 (sky130_fd_sc_hd__mux2_2)
                    0.0322    0.1227    1.8262 ^ 29511/X (sky130_fd_sc_hd__mux2_2)
     1    0.0017                                 alu_out[26] (net)
                    0.0322    0.0000    1.8262 ^ 30967/D (sky130_fd_sc_hd__dfxtp_2)
                                        1.8262   data arrival time

                              0.0000    0.0000   clock clk (rise edge)
                              0.0000    0.0000   clock source latency
                    0.0225    0.0100    0.0100 ^ clk (in)
     1    0.0079                                 clk (net)
                    0.0225    0.0000    0.0100 ^ clkbuf_0_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0285    0.1019    0.1119 ^ clkbuf_0_clk/X (sky130_fd_sc_hd__clkbuf_16)
     2    0.0046                                 clknet_0_clk (net)
                    0.0285    0.0000    0.1119 ^ clkbuf_1_1_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0817    0.1936 ^ clkbuf_1_1_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_1_1_0_clk (net)
                    0.0264    0.0000    0.1936 ^ clkbuf_1_1_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0810    0.2746 ^ clkbuf_1_1_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_1_1_1_clk (net)
                    0.0264    0.0000    0.2746 ^ clkbuf_1_1_2_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0913    0.3659 ^ clkbuf_1_1_2_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_1_1_2_clk (net)
                    0.0379    0.0000    0.3659 ^ clkbuf_2_2_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0851    0.4510 ^ clkbuf_2_2_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_2_2_0_clk (net)
                    0.0264    0.0000    0.4510 ^ clkbuf_2_2_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0810    0.5320 ^ clkbuf_2_2_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_2_2_1_clk (net)
                    0.0264    0.0000    0.5320 ^ clkbuf_2_2_2_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0913    0.6233 ^ clkbuf_2_2_2_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_2_2_2_clk (net)
                    0.0379    0.0000    0.6233 ^ clkbuf_3_4_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0851    0.7084 ^ clkbuf_3_4_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_3_4_0_clk (net)
                    0.0264    0.0000    0.7084 ^ clkbuf_3_4_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0913    0.7996 ^ clkbuf_3_4_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_3_4_1_clk (net)
                    0.0379    0.0000    0.7996 ^ clkbuf_4_8_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0379    0.0953    0.8950 ^ clkbuf_4_8_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     2    0.0046                                 clknet_4_8_0_clk (net)
                    0.0379    0.0000    0.8950 ^ clkbuf_5_17_0_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.0264    0.0851    0.9801 ^ clkbuf_5_17_0_clk/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0023                                 clknet_5_17_0_clk (net)
                    0.0264    0.0000    0.9801 ^ clkbuf_5_17_1_clk/A (sky130_fd_sc_hd__clkbuf_2)
                    0.4715    0.3897    1.3697 ^ clkbuf_5_17_1_clk/X (sky130_fd_sc_hd__clkbuf_2)
    11    0.0868                                 clknet_5_17_1_clk (net)
                    0.4715    0.0000    1.3697 ^ clkbuf_leaf_170_clk/A (sky130_fd_sc_hd__clkbuf_16)
                    0.0547    0.2382    1.6079 ^ clkbuf_leaf_170_clk/X (sky130_fd_sc_hd__clkbuf_16)
    12    0.0225                                 clknet_leaf_170_clk (net)
                    0.0547    0.0000    1.6079 ^ 30967/CLK (sky130_fd_sc_hd__dfxtp_2)
                              0.0000    1.6079   clock reconvergence pessimism
                             -0.0273    1.5806   library hold time
                                        1.5806   data required time
-------------------------------------------------------------------------------------
                                        1.5806   data required time
                                       -1.8262   data arrival time
-------------------------------------------------------------------------------------
                                        0.2456   slack (MET)


Startpoint: resetn (input port clocked by clk)
Endpoint: mem_la_read (output port clocked by clk)
Path Group: clk
Path Type: max

Fanout       Cap      Slew     Delay      Time   Description
-------------------------------------------------------------------------------------
                              0.0000    0.0000   clock clk (rise edge)
                              0.0000    0.0000   clock network delay (propagated)
                              4.9460    4.9460 ^ input external delay
                    0.0182    0.0064    4.9524 ^ resetn (in)
     1    0.0049                                 resetn (net)
                    0.0182    0.0000    4.9524 ^ input101/A (sky130_fd_sc_hd__buf_6)
                    0.0587    0.1008    5.0532 ^ input101/X (sky130_fd_sc_hd__buf_6)
     7    0.0234                                 net101 (net)
                    0.0587    0.0000    5.0532 ^ 15304/A (sky130_fd_sc_hd__clkbuf_4)
                    0.0934    0.1738    5.2269 ^ 15304/X (sky130_fd_sc_hd__clkbuf_4)
     6    0.0265                                 12638 (net)
                    0.0934    0.0000    5.2269 ^ 17093/C (sky130_fd_sc_hd__nand3_4)
                    0.1284    0.1274    5.3543 v 17093/Y (sky130_fd_sc_hd__nand3_4)
     4    0.0282                                 13857 (net)
                    0.1284    0.0000    5.3543 v 18867/B1 (sky130_fd_sc_hd__a21oi_4)
                    0.0799    0.1239    5.4782 ^ 18867/Y (sky130_fd_sc_hd__a21oi_4)
     1    0.0023                                 net199 (net)
                    0.0799    0.0000    5.4782 ^ output199/A (sky130_fd_sc_hd__clkbuf_2)
                    0.1052    0.1596    5.6378 ^ output199/X (sky130_fd_sc_hd__clkbuf_2)
     1    0.0177                                 mem_la_read (net)
                    0.1052    0.0000    5.6378 ^ mem_la_read (out)
                                        5.6378   data arrival time

                             24.7300   24.7300   clock clk (rise edge)
                              0.0000   24.7300   clock network delay (propagated)
                              0.0000   24.7300   clock reconvergence pessimism
                             -4.9460   19.7840   output external delay
                                       19.7840   data required time
-------------------------------------------------------------------------------------
                                       19.7840   data required time
                                       -5.6378   data arrival time
-------------------------------------------------------------------------------------
                                       14.1462   slack (MET)



![](img/pcts2.png)

![](img/pcts5.png)
















    
