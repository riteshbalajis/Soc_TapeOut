# Week 4 

##  Install NGSpice  (Linux)

Follow these commands to build and install the latest NGSpice from source.


Update system packages

    sudo apt update

Install required dependencies

    sudo apt install git build-essential libxaw7-devlibreadline-dev libtool automake autoconf bison flex -y

Clone the NGSpice repository

    git clone https://git.code.sf.net/p/ngspice/ngspice
    cd ngspice

Prepare build configuration

    ./autogen.sh
    ./configure --with-x --enable-xspice

Compile the source

    make

Install NGSpice system-wide

    sudo make install

Verify installation

    ngspice -v

![](img/ngspice.png)


### Why ID vs VDS Plot?
The ID vs VDS plot helps us understand how the drain current (ID) changes with drain-to-source voltage (VDS) for different gate voltages (VGS).  
It shows the NMOS transistor’s behavior in **different regions of operation** — cutoff, linear (ohmic), and saturation — which is essential for designing and analyzing circuits.

### NGSpice Code:

![](img/200_idvds_c.png)

![Execution](img/day1_c.png)

### Graph Description
Below is the simulated **ID vs VDS** plot of an NMOS transistor.  
Here, **VGS is increased in steps of 0.2V** for each curve while measuring the corresponding drain current (ID).

![NMOS ID vs VDS](img/200_idvds.png)
![Cut Off Region](img/200_idvdszoom.png)

### Observation / Analysis
- At low VDS, the transistor operates in the **linear region**, and ID increases almost linearly with VDS.  
- As VDS increases further, ID gradually **saturates**, showing the **saturation region** where the current becomes nearly constant.  
- Increasing VGS shifts the curves upward, indicating **higher current flow** for larger gate voltages.  
- The plot clearly demonstrates the **dependence of drain current on both VGS and VDS**, helping us visualize transistor behavior.

---

#Effect of Velocity Saturation on MOSFET Characteristics

## Objective
To analyze how **velocity saturation** affects the **drain current (ID)** characteristics of a MOSFET by:
-  Studying **ID–VDS** and **ID–VGS** curves for transistors with different **W** and **L** values but the **same W/L ratio**.
---

## Given Data
| Case | Channel Length (L) | Channel Width (W) | W/L Ratio |
|------|--------------------|------------------|------------|
| **1** | 2 µm | 5 µm | 2.5 |
| **2** | 0.15 µm | 0.39 µm | ≈ 2.5 |

> Both devices have approximately the same W/L ratio, allowing comparison of current behavior independent of geometry ratio.

---

## 1. ID–VDS Characteristics

![](img/day2_idvds_c.png)

### Id vs Vds (2 µm | 5 µm
![](img/200_idvds.png)

### Id vs Vds (0.15 µm | 0.39 µm )
![](img/150_idvds.png)

### Observation
- Even with identical W/L, the **ID–VDS curves** differ significantly between long and short channel devices.

#### **For Longer Channel (L = 2 µm):**
- Exhibits **square-law behavior**:  

- The transition from **linear** to **saturation region** is **smooth and gradual**.

- The device follows the **long-channel MOSFET model** accurately.

#### **For Shorter Channel (L = 0.15 µm):**
- Shows **early saturation** — current flattens out at lower VDS values.
- **Saturation current is higher**, but the **rate of increase with VDS** is smaller.
- Indicates **velocity saturation** — carrier velocity stops increasing linearly with the electric field.
- Thus, **current depends linearly on (VGS – VTH)** instead of quadratically.



---

## 2. ID–VGS Characteristics (at Constant W/L)

![](img/150_idvgs_w.png)

### Id vs Vgs (2 µm | 5 µm
![](img/200_idvg.png)

### Id vs Vgs (0.15 µm | 0.39 µm )
![](img/150_idvgs.png)

### Observation
- Even when W/L is constant, the **ID–VGS curves** differ between devices.
- For low VGS → **Quadratic dependence** (square-law region).  
- For high VGS → **Linear dependence**, due to velocity saturation.



---

## 3. Key Comparative Observations

| Parameter | Long Channel (2 µm) | Short Channel (0.15 µm) |
|------------|---------------------|---------------------------|
| **Region transition** | Gradual | Early saturation |
| **ID–VDS curve** | Quadratic rise | Linear rise, early flattening |
| **ID–VGS curve** | Square-law | Velocity-limited (linear) |
| **Dominant effect** | Mobility-limited | Velocity saturation |
| **Impact of scaling** | Predictable current | Deviation from ideal model |

---

## 4. Conclusion

- As **channel length decreases**, **electric field strength** increases rapidly, causing **velocity saturation**.  
- The MOSFET no longer follows the **square-law model**, and current becomes **linearly dependent** on \((V_{GS} - V_{TH})\).
- **Short-channel devices** reach saturation earlier and exhibit **reduced transconductance** and **higher output conductance**.
- This demonstrates the necessity of using **advanced MOSFET models** (like BSIM or velocity-saturation-inclusive models) for deep-submicron technologies.

---

## 5. Summary

| Aspect | Long Channel Behavior | Short Channel Behavior |
|--------|------------------------|------------------------|
| **ID–VDS** | Quadratic → smooth saturation | Early saturation due to vₛₐₜ |
| **ID–VGS** | Square-law increase | Linear increase |
| **Cause of deviation** | Mobility dominance | Velocity saturation |
| **Practical significance** | Accurate for older tech nodes | Must include short-channel effects |

---

- This experiment confirms that **keeping W/L constant does not guarantee identical behavior** when **L scales down**.   **Velocity saturation** becomes a dominant limiting factor, influencing both **transfer (ID–VGS)** and **output (ID–VDS)** characteristics.
