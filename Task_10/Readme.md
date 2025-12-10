# Caravel SoC Overview & Housekeeping SPI (HKSPI)

##  What is Caravel?

**Caravel** is a pre-designed SoC wrapper used in open-source ASIC projects.  
It includes a built-in RISC-V management SoC and provides an easy framework to integrate custom digital logic.

Caravel divides the chip into two main regions:

### **1. Management Area**
- Contains the RISC-V management core  
- Handles system control and IO configuration  
- Manages clocking, boot flow, and housekeeping features  

### **2. User Project Area**
- Region where custom logic is implemented  
- Isolated from the management core  
- Allows controlled interaction with management firmware  

---

##  Housekeeping SPI (HKSPI)

### **What is HKSPI?**
The **Housekeeping SPI (HKSPI)** is an SPI slave block that allows an external SPI master (MCU, FPGA, Raspberry Pi, etc.) to communicate with the Caravel SoC.

### **HKSPI Provides:**
- Configuration of management SoC registers  
- Reading system status  
- Control of chip-level features  
- Indirect interaction with the User Project  
- Pass-through mode for external flash access  

---

##  Standard 4-Pin SPI Interface

| Signal | Description | Caravel Pin |
|--------|-------------|-------------|
| **SDI (MOSI)** | Slave Data In | F9 |
| **SCK** | Clock | F8 |
| **CSB** | Chip Select (active low) | E8 |
| **SDO (MISO)** | Slave Data Out | E9 |

### **Timing Details**
- **SDI** is sampled on the **rising edge** of SCK  
- **SDO** changes on the **falling edge** of SCK  
- **SDO** is **tri-stated** when:  
  - `CSB = HIGH`  
  - During non-read operations  

---

##  HKSPI Communication Protocol

![](img/hk_signalling.png)


### **Transaction Steps**
1. **CSB pulled LOW** → enter **COMMAND** state  
2. **Command byte (8 bits, MSB first)** defines operation mode  
3. **Address byte (8 bits)** selects register address  
4. **One or more data bytes** are read/written  
5. **CSB pulled HIGH** ends the transaction  

---

## ⚙ Modes of Operation

![](img/hk_mode.png)

### **1. Streaming Mode**
- Activated when **command bits [5:3] = 000**  
- Allows unlimited consecutive bytes  
- Ends when CSB goes HIGH  

### **2. N-Byte Mode**
- Activated when **command bits [5:3] = 001 … 111**  
- Defines a fixed number of bytes to read/write  
- CSB may stay LOW across multiple commands  

### **3. Pass-Through Mode**
Used to directly access the SPI Flash.

Once enabled:
- RISC-V management CPU is **forced into reset**  
- External SPI master fully controls:  
  - SDI  
  - SCK  
  - Flash CSB  
- Flash output is forwarded to **SDO**  
- Mode ends when CSB is released (goes HIGH)

### HKSpi Reg Map:

![](img/hk_regmap.png)

---

# SKY130 Installation, Caravel Setup & HKSPI Testbench 


## Installing SKY130 PDK Using Volare

---

    ###  Create a Python Virtual Environment
        python3 -m venv ~/myenv

    ### Activate the Virtual Environment
        source ~/myenv/bin/activate

    ### Upgrade pip
        pip install --upgrade pip

    ### Install Volare
        pip install volare

    ### Set the PDK Installation Path
        export PDK_ROOT="directory to install sky130"

    ### List Available SKY130 Versions
        volare ls --pdk sky130

    ### choose the version from the previous list :
        volare enable --pdk sky130 <version_number>

## Clone the Caravel Repository

    git clone https://github.com/efabless/caravel.git


---

## Testbench Tasks

The testbench defines three primary tasks used to interact with the HKSPI interface.

### 1. write_byte
- Sends a byte to the SDI pin.
- Input: `odata[7:0]`
- Used for write operations over SPI.

### 2. read_byte
- Receives a byte from the SDO pin.
- Output: `idata[7:0]`
- Used for reading data from HKSPI registers.

### 3. read_write_byte
- Performs simultaneous SPI read and write operations.
- Useful for full-duplex behavior during SPI transfers.

---

## Timing Information

- Each of the tasks (write, read, or read-write) requires a total of **200 ns** to complete.
- The SPI clock toggles every **100 ns**:
  - Clock transitions from 0 to 1 after 100 ns.
  - Clock transitions from 1 to 0 after the next 100 ns.

---

## Test Scenarios

The testbench performs the following functional checks:

### 1. Reading Product ID (Address `8'h03`)
- The testbench reads from register address `0x03`.
- This register holds the Product ID.
- Expected read value: `0x11`.

### 2. Writing to External Reset Register (Address `8'h0B`)
- The testbench writes value `1` to address `0x0B` to assert an external reset.
- Then it writes value `0` to deassert the external reset.
- Confirms correct write behavior and register functionality.

### 3. Streaming Mode Register Read
- The testbench enables HKSPI streaming mode.
- In streaming mode, the register address auto-increments after each byte read.
- The testbench reads 18 consecutive registers.
- Each read value is compared against the default values defined in the HKSPI specification.
- Validates streaming mode functionality and sequential register access.

---
# Simulation of HKSPI


Inside the HKSPI testbench folder (`caravel/verilog/dv/caravel/mgmt_soc/hkspi`) there is a Makefile to run the testbench.

## Simulation Command

      make RTL=SIM


![](img/caravel_picro_giterror.png)

The error indicates that `DFFRAM.v` was not found in the `caravel_pico` directory.  
The GitHub repository for `caravel_pico` was referenced inside the Makefile.

To fix this, I cloned the repository locally and updated the Makefile with the correct directory path.

## Running Simulation

![](img/reg_out_error.png)

This error occurred because many variables were declared twice.  
A variable was declared as an `output` in the module header, and then declared again inside the module as a `reg`.  
To resolve this, I changed those variables to `output reg`.  



## Running Simulation Again

![](img/img.png)

The test failed because register `12` had the value `0x01` but it should be `0x00`.

Register 12 indicates a CPU trap.  
If the CPU encounters an error, the trap flag (register `0x0C` / 12th register) is set to `0x01`.

### CPU Trap Description

| Address | Bit | Description |
|---------|-----|-------------|
| `0x0C` | 0 | Set to 1 if the CPU stops due to an error and raises the trap signal. |

The trap signal can also be read via a GPIO pin, but since the GPIO state may be unknown, HKSPI provides a reliable way to read it.

---

# Debugging

### 1. Backtracking the Signal Source

The trap signal originates from `picorv32.v`.

![](img/cpu_state2.png)

![](img/cpu_state2.png)

The trap is asserted when the CPU encounters errors such as misaligned word accesses or similar exceptions.

### 2. Adding Display Statements in Testbench

I added multiple `$display` statements to observe register 12 at different stages:

1. **At the very beginning inside `initial begin`:**  
   The signal was `0xZZ`.

2. **After initialization (after `RSTB = 0`):**  
   The signal value was `0x00`.

3. **After reset is toggled (`RSTB = 1`, then `RSTB = 0` after a delay):**  
   The signal changed to `0x01`.

This is the point where `cpu_trap` becomes 1.

---

# Checking Other Tests

To confirm the rest of the testbench functionality, I forced register 12 to always return `0x01` to see if other tests pass.

## Modification

![](img/foreced_mod.png)

## Simulation Output

![](img/full_test.png)

All tests passed with this modification.  
The only issue is the behavior of register 12 (CPU trap).

---

# Waveform

### Command

    gtkwave hkspi.vcd


![](img/wave_c.png)
![](img/wave_1.png)

Below is the explanation of waveform sections for each test.

---

# 1. Reading Product ID

![](img/wave_2.png)

`8'h40` and `8'h03` are passed as `odata` to HKSPI.  
These represent the read-stream command and the Product ID register address.  
The `idata` returned by HKSPI is `0x11`, which is the Product ID.

![binary](img/wave_3.png)

---

# 2. Toggling External Reset (Register `8'h0B`)

![](img/wave_4.png)

First, `8'h80` (write-stream command) is sent, followed by `8'h0B` (CPU reset register), and then `8'h01` to assert reset.

Later the same register is set to `0`:

![](img/wave_5.png)

---

# 3. Reading All Registers (0 to 18)

![](img/wave_7.png)

`8'h40` is passed as the read-stream command and `8'h00` as the starting address.  
In stream mode, each following address increments automatically.  
Thus, HKSPI reads registers 0 through 18 sequentially and outputs them to `tbdata`.

The waveform clearly shows data being read from registers 0 to 18.

---


## Gate Level Simulation of Hkspi (GLS):

- Generating netlist file for housekeeping_spi.V

![](img/synth1.png)
![](img/synth2.png)
![](img/4_hkspinet_c.png)

### Synthesised Netlist:

![](img/4_hkspinet.png)


### Simulation:

*Errors:*


![](img/mgmt_e.png)

the mgmt_defines.v and mgmt_core_wrapper.v are present inside the caravel_pico directory , so updated the proper directory in caravel_netlist.v

![](img/mgmt_es.png)


![](img/dpll_e.png)

digital_pll.v not found error arise due to  same improper directory in caravel_netlist.v file. Changed the directory to proper gl directory for multiple includes present in it.

![](img/dpll_es.png)

### Simulation :

![](img/glsp1.png)
![](img/glsp2.png)

### Waveform:

![](img/glsp_wave.png)

### Comparison

| RTL Simulation | GLS Simulation |
|---------|---------|
| ![](img/rltp_wave.png) | ![](img/glsp_wave.png) |



