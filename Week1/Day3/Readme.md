
# Introduction to Optimization

         i) Combinational Logic Optimization
        ii) Sequential Logic Optimization

## Combinational Logic Optimization

- The process of reducing the number of logic gates and the number of inputs to those gates in a circuit

### Methods :
- Constant Propogation
- Boolean Logic Optimization
 

### Constant Propogation:-

Process of Replacing a constant value(0/1) to the variable 

Example: y=(a+(bc)+(da))'  (CMOS Logic - 10 Mos used)

- if a=gnd (logic 0)

-  y=bc' (CMOS Logic - 4 Mos used)

### Boolean Logic Optimization:-

The boolean expression is optimized using K-Map,Quine–McCluskey etc...

Example: 



## Sequential Logic Optimization:

### Methods-

        1. Basic - Sequential Constant Propogation
        2. Advances - state Optimization, Retiming, Sequential Logic Clonning
### Sequential Constant :-

When a ouput of a block is constant value(0/1) independent of the change in inputs of the circuit then the ouput is sequential constant 

#### Example:

![Logic Diagram](diagram.png)

The output from the flip flop will be always 0.

The expression y becomes  y=0.a' -> y=0' -> y=1(always) ---> Sequential Constant

#### Example (not sequential constant):

![ Logic diagram ](diagram.png)

in this the output of Flip Fliup depends on the set . If set=1 then the output q immediately (asynchronous set) becomes one.But when the set=0 it waits for posedge to follow the d (0) to output. So we can describe y=set -->(Wrong). 

![Logic Diagram ](wave.png)


## Advanced Optimization:

### State Optimization : 
- reducing number of states in fsm to improve the harware implementations

### Clonning:      
-  When a single gate's output is connected to the inputs of many other gates (a high fanout), it can cause a significant signal delay . To reduce this identical copy of that gate/ff is created and divided among the multiple gates/ff.

### Retiming:
-  It is a technique in digital circuit design that moves registers (flip-flops) across combinational logic to reduce the clock cycle time.

![Retiming](img)

## Lab on Optimization:

### command
        opt_clean -purge
