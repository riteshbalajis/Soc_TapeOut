# ⚡SoC Tapeout Course – Learning Journey  

Welcome to my **SoC (System-on-Chip) Tapeout Journey 🚀**.  
This repository documents the entire path from **idea → design → silicon** using open-source EDA tools.  
Every week adds new building blocks toward the final goal: **a working chip taped out!**  

---
add this 

## ╰┈➤ Week 0️⃣ – Introduction to SoC Tapeout Flow  

This week lays the foundation of the SoC design and tapeout journey.  
We start from the **application specification** and follow the step-by-step process until the chip can be fabricated.  
Along the way, we also set up the required **open-source tools**.  

---

### 📌 Task 1: From Application to Tapeout  

#### 🎯 Goal  
The ultimate goal is to **run the application on the chip**.  

#### 🔹 Step 1: Verify the Application / Specification  
- A **testbench** checks whether the given **application (specification)** is correct.  
- Two outputs are compared:  
  - `testbench -> gcc -> out1`  
  - `testbench -> specs (C model) -> out2`  
- If `out1 == out2`, then the **specification is verified** and **frozen**.  

👉 This stage is called **Chip Modeling**.  

---

#### 🔹 Step 2: Softcopy of Hardware  
- Once the specification is verified, we create a **softcopy of hardware** (RTL code in Verilog or VHDL).  
- At every stage in the design flow, the outputs are compared with the original specification to ensure correctness.  

---

#### ✅ Summary  
- Every step in the SoC design originates from the **application specification**.  
- At each stage (C model, RTL, gate-level, physical design), the output is compared against the application.  
- If outputs match, we proceed to the next stage.  
- This ensures **functional equivalence** all the way to the final tapeout.  

---

### 📌 Task 2: Software Installation  

To perform simulation, synthesis, and verification, we install the following **open-source tools**:  

#### 🔹 Installed Tools  
1. **Yosys** → Logic synthesis  
2. **Icarus Verilog (iverilog)** → RTL simulation  
3. **GTKWave** → Waveform visualization  

---

#### 🔹 Installation Commands  

```bash
# Update packages
sudo apt-get update

# Install Icarus Verilog
sudo apt-get install iverilog

# Install GTKWave
sudo apt-get install gtkwave

# Install Yosys
sudo apt-get install yosys
