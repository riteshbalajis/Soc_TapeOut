# BabySoC Fundamentals & Functional Modelling
# System on Chip

A **System on a Chip (SoC)** is an integrated circuit (IC) that combines most or all of the essential components of a computer or electronic system onto a single microchip.

Unlike older electronic circuits, where components like the CPU (Central Processing Unit), GPU (Graphics Processing Unit), and memory were mounted separately on a board, an SoC integrates all these components onto a single chip.

---

## Types of SoC

### Based on the Primary Processor Core

**Microprocessor-Based SoC:**  
Used for complex, general-purpose operating systems and high-level computation. Requires external memory (RAM) chips.  
*Examples:* Smartphones, Tablets, High-end Routers

**Microcontroller-Based SoC:**  
Used for less complex control and monitoring tasks. Focuses on low power consumption and real-time response.  
*Examples:* Internet of Things (IoT) devices, Smart home sensors, Embedded systems

**Application-Specific Integrated Circuit (ASIC) SoC:**  
Designed for the exclusive use of a single customer and a specific application. It integrates only the necessary components, making it highly efficient, fast, and power-optimized.  
*Examples:* AI/ML Accelerators, Networking SoCs

---

## Advantages

- **Power Efficiency:** SoCs are designed for optimal power usage, making them ideal for battery-powered devices.
- **Smaller Size and Weight:** Essential for compact devices like smartphones, tablets, and wearables.
- **Cost-Effectiveness:** Manufacturing a single integrated chip is more cost-effective than producing separate discrete components.
- **Performance:** In an SoC, components are placed close together on a single chip, enabling faster data transfer, lower power consumption, and improved overall performance.

---

## Challenges

- **Complex Design:** Designing SoCs is challenging and requires specialized tools, expertise, and significant development time, which can lead to longer time-to-market.
- **High Design Cost:** The initial design and development cost is very high. If the production volume is low, the cost per SoC will be high.
- **Inflexibility:** Once manufactured, the SoC cannot be easily modified or updated.

---

## Components of an SoC

- **Central Processing Unit (CPU):**  
  The primary core that executes instructions, runs the operating system, and manages general-purpose computing tasks. Advanced SoCs use multiple cores to balance performance and power efficiency.

- **Graphical Processing Unit (GPU):**  
  A parallel processing unit dedicated to rendering images, 3D graphics, and videos. It handles visual output and is crucial for gaming, multimedia, and smooth user interfaces.

- **Memory:**  
  Memory is a critical part of an SoC, used to store instructions and data for the processor and other components. SoCs typically include:
  - *On-chip memory (RAM/ROM/SRAM):* Fast and close to the CPU for quick access, used for temporary data storage and program execution.  
  - *Non-volatile memory (Flash, EEPROM):* Stores firmware, programs, or configuration data that must be retained even when power is off.  
  - *Cache memory:* Small, very fast memory located near the CPU to reduce data access time and improve performance.

- **Interconnect:**  
  The high-speed internal communication structure, often an Advanced eXtensible Interface (AXI) bus or Network-on-Chip (NoC). It enables efficient data exchange between all IP blocks (CPU, GPU, Peripherals, etc.).

- **Power Management Unit (PMU):**  
  Manages the chip’s power rails by controlling voltage and clock frequencies for different blocks. It is vital for maximizing battery life.

- **Peripherals / I/O Blocks:**  
  Peripherals allow the SoC to interact with the outside world. They include interfaces like GPIO, UART, SPI, I²C, timers, and ADC/DAC, enabling connections to sensors, displays, and other devices.

---

## VSD BabySoC

**VSD BabySoC** is a simplified System-on-Chip (SoC) designed for open-source chip design learning and experimentation. It integrates key digital and analog components to demonstrate how different IP blocks interact within a chip environment. The design operates across two voltage domains — **1.8V** and **3.3V** — to support both low-power logic and analog peripherals.

![](Week2/Lab/img/babysoc.png)
---

## Why VSD BabySoC

- Provides a minimal, easy-to-understand SoC framework for beginners in VLSI and chip design.  
- Helps learners explore interfacing between analog and digital blocks, such as the PLL, DAC, and processor cores.

---

## Parts of BabySoC

### 1. PLL (avsdpll_1v8)

- The **Phase-Locked Loop (PLL)** generates a stable, high-frequency clock signal for the processor and peripherals.  
- Operates at 1.8V and takes a reference clock from a crystal oscillator.  
- The PLL locks its output frequency to match the input reference frequency, ensuring timing stability across the SoC.

### 2. RISC-V Core (rvmyth)

- **rvmyth** is a simple, educational RISC-V processor core implemented in Verilog.  
- It forms the digital brain of the SoC, handling computation, instruction execution, and control logic.  
- The core communicates with peripherals like SPI, PLL, and DAC through standard digital interfaces.

### 3. DAC (avsddac_3v3)

- The **Digital-to-Analog Converter (DAC)** converts the digital output from the RISC-V core into an analog signal.  
- Operates in the 3.3V domain for a higher output voltage range and analog compatibility.  
- The DAC uses reference voltages (VREFL and VREFH) to generate analog outputs proportional to the digital input values.

---

### Voltage Domains

- **1.8V Domain:** Digital logic and low-power analog circuits (PLL and rvmyth).  
- **3.3V Domain:** High-voltage analog components (DAC, SPI I/O).  

**Level Shifters (LS)** are used to interface signals safely between the two voltage domains.
