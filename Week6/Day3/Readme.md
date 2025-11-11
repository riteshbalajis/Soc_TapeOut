# Day 3 - Design Library Cells Using Magic 



### Pin Configuration in Floorplan

The default setting for pin configuration:

    set ::env(FP_IO_MODE) 1


- `0` → Matching mode  
- `1` → Random equidistant mode (default)

Pins are placed with equal distance when using the default value.

When other values are set for `FP_IO_MODE`, the changes in pin placement become visible.

#### Default

![](img/pin_config_def.png)
#### After changed

![](img/changed_pinconfig.png)

- the pins are place not in uniform spaced manner

### 16-Mask CMOS Process Steps

1. **Selecting Substrate**  
   The process begins with choosing a high-quality silicon wafer as the base material.  
   This substrate acts as the foundation on which all CMOS devices are built.

2. **Creating Active Region for Transistors**  
   Active areas are defined using oxide layers and photolithography.  
   These regions are where the transistor channels will be formed.

3. **N-Well and P-Well Formation**  
   Doping is performed to create separate regions for NMOS and PMOS transistors.  
   N-well is used for PMOS devices and P-well for NMOS devices.

4. **Formation of Gate**  
   A thin gate oxide layer is grown, and polysilicon is deposited on top.  
   The gate structure is patterned to control transistor switching.

5. **LDD (Lightly Doped Drain) Formation**  
   Light doping is introduced near the gate edges to reduce electric field stress.  
   This improves device reliability and minimizes hot-carrier effects.

6. **Source and Drain Formation**  
   Heavily doped regions are implanted to form low-resistance source and drain terminals.  
   These define the main conducting terminals of the transistor.

7. **Steps to Form Contact**  
   An insulating layer is added, and openings are etched to reach the active regions.  
   Metal contacts are deposited to provide electrical connection to gate, source, and drain.

8. **Higher-Level Metal Formation**  
   Multiple metal layers are added for signal routing across the chip.  
   Each layer is separated by dielectric and connected using vias for interconnection.


## Lab: Layout - Ngspice

- gitclone cell design

    git clone https://github.com/nickson-jose/vsdstdcelldesign.git 

- copy tech file from openlane to vsdstdcelldesign

- command 

    magic -T ./lib/sky130A.tech sky130_inv.mag &

![](img/inv_mag.png)

![](img/inv_mag1.png)

- this is the layout of cmos inverter

![](img/inv_mag_s1.png)
![](img/inv_mag_S2.png)


- by clicking **s** on three time on a certain connects the entire connection can be seen by highlighted white line

![](img/inv_what.png)

-  what can be used to see information about  the part selected

![](img/pmos_what.png)
![](img/nmos_what.png)

- the poly cross n-diff it is NMOS and if poly cross p-diff it is PMOS

### Extract to Spice

    extract all

    ext2spice cthresh 0 rthresh 0

    ext2spice

![](img/extract_ng.spice)

- after these commands the extension file and spice file is created 

![](ls_afextract.png)

- Open spice file
![](img/inv_spice.png)

![](img/cell_size.png)

- by seing the single cell size the time scale is changed to 0.01u
- and by technology file the name of pmos and nmos also changed

- Updated spice file
![](img/spice_mod.png)

### Inverted Output

    plot y vs time a

![](img/ng_cons.png)

![](img/inv_out1.png)

### Analysis of Output

**Rise Transistion Time**:

    tr=tout(80%)-tout(20%) (at rise 0 to 1)

vdd=3.3v
80%= 2.64v
20%= 0.66v

- At 80%

![](img/rtrans_w80.png)

![](img/rtrans_v80.png)

- At 20%

![](img/rtrans_w20.png)

![](img/rtrans_v20.png)

tr= 2.24547e-9 - 2.18212e-9

tr= 0.06335 ns

**Fall Transistion Time**:

    tf=tout(80%) - tout(20%) (at fall 1 to 0)

vdd=3.3v
80%= 2.64v
20%= 0.66v

![](img/ftrans_w80.png)
![](img/ftrans_w20.png)

![](ftrans_v.png)

tf=8.09516ns - 8.05132 na

tf= 0.04384 ns

**Rise Delay**:

    trd=tout(50%)-tin(50%) (at rise 0 to 1)

![](img/rdelay_w.png)
![](img/rdelay_v.png)

trd=2.23097 ns - 2.1499 ns 
trd=0.08107ns

**Fall Delay**

    trd=tout(50%)-tin(50%) (at fall 1 to 0)

![](img/fdelay_w.png)
![](img/fdelay_v.png)

tfd=4.07763 ns - 4.0499ns
tfd=0.02773 ns







