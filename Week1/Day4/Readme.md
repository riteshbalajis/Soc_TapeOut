# Day - 4  

## Introduction to Gate Level Simulation (GLS)  

Gate Level Simulation (GLS) is the process of simulating the **netlist** that is created by synthesizing the RTL code using a testbench.  

- The netlist is **logically equivalent** to the RTL code, so the same testbench can be reused.  

### Why GLS?  
- To verify the **logical correctness** after synthesis of RTL into a netlist.  
- In case of timing circuits, to ensure that the **timing goals of the design are met**.  

### GLS Workflow  
![GLS Workflow Diagram](path/to/gls_workflow.png)  

---

## Example of GLS  

## RTL expression:  

    ```verilog
    assign y = (a | b) & c;

### Equivalent netlist:

    or  u_or  (.a(a), .b(b), .y(i));
    and u_and (.a(i), .b(c), .y(y));


- or and and are gates, and this information is provided by the gate-level model.

### Gate Level Models

- Timing-Aware Model → Verifies both functionality and timing.

- Functional Model → Verifies only the functionality.

### Why Verification is Still Needed?

Even though the netlist is logically equivalent to the RTL, verification is done because there can be synthesis and simulation mismatches.

## Synthesis and Simulation Mismatch

### Reasons:

### Missing sensitivity list:
The simulator works only on activity (i.e., changes in input signals).

**Example** :

// Example will be added here


### Blocking and Non-Blocking Statements

- Blocking: Executes the given statements one by one, in order.

- Non-Blocking: Executes all the right-hand side expressions first, then assigns them to the left-hand side.

**Example1** :




- Non-blocking statements are always used in sequential circuits.

**Example2** :
