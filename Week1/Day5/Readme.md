# Day 5

## IF Statement :
- It is a priority based logic ,checks the conditon one by one in order if one satsifies the if block gets terminates and come end of if block.

        if <cond1> 
        ...
        c1
        ...
        else if<cond2>
        ...
        c2
        ...
        else if <cond3>
        ...
        c3
        ...
        else
        ...
        e
        ...

-This Block can be represented as 
![General Block of If](img/if_flow.png)

##Caution of If Satatement 
### Inferred Latch

-This inferred latch occurs when there is missing of condition in if statement which occurs.

### Example1(Lab):

Rtl Code:

    module incomp_if (input i0 , input i1 , input i2 , output reg y);
    always @ (*)
    begin
        if(i0)
            y <= i1;
    end
    endmodule
![Block of Incomp_If](img)

### Gtk_Wave(incomp_if):

![](img/incomp_case_vcd.png)

- when i0=0 the wave will be constant output of previous value which will include a latch in the circuit.

![Synthesis Information ](img/incomp_case_synth.png)

-from this we can see that d latch is included in the netlist.

### Netlist:

![Netlist](img/incomp_case_netlist.png)

---

### Example2(Lab):

Rtl Code:

    
    module incomp_if2 (input i0 , input i1 , input i2 , input i3, output reg y);
    always @ (*)
    begin
        if(i0)
            y <= i1;
        else if (i2)
            y <= i3;

    end
    endmodule
![Block of Incomp_If2](img/incomp_if2_vcd.png)

### Gtk_Wave(incomp_if2):

![](img/incomp_if2_vcd.png)
---

![](img/incomp_if2_vcd2.png)

- when i0==0 or i2==0 the wave will be constant output of previous value which will include a latch in the circuit.

![Synthesis Information ](img/incomp_if2_synth.png)

-from this we can see that d latch is included in the netlist.

### Netlist:

![Netlist Generation](img/incomp_if2_c.png)

---
![Netlist](img/incomp_if2_netlist.png)

---

-Counter shoulde contain inferred latch because of the principle of counter as it should increment from previous value so, in that case it is good to have inferred latch.

![circuit of counter](counter.png)

**Note**:
- whatever the statementassignes inside if/case satement must be of reg variable.

## Case Statement:
- In Verilog, a case statement is used for multi-way branching based on an expression.
- It checks all conditions one by one, and even if one matches, it continues checking the next conditions.

### Limitations

### 1. Inferred Latch:
- This arises due to the missing of conditon inside case statement which occurs then the circuit will use previous output for that case. So, Latch is used to store the previous Output.

### Example1(lab):

Code:

    module incomp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
    always @ (*)
    begin
        case(sel)
            2'b00 : y = i0;
            2'b01 : y = i1;
        endcase
    end
    endmodule

Gtk_wave:

    ![incomp_case](img/incomp_case_vcd)

- when sel=2'b10 the output y goes to contsant value of previous output.So, latch is invoked at the circuit.

    ![incomp_case](img/incomp_case_vcd2)

- when sel=2'b11 the output y goes to contsant value of previous output.So, latch is invoked at the circuit.

[Synthesis Details](img/incomp_case_synth.png)

- in this d-latch is used

### Netlist:

![Netlist Generation](img/incomp_case_c.png)

![Netlist](incomp_case_netlist.png)

---

### 2. Partial Assignment:
- This arises due to the missing assignment of input signal to output.

### Example(lab):

Code:

    module partial_case_assign (input i0 , input i1 , input i2 , input [1:0] sel, output reg y , output reg x);
    always @ (*)
    begin
        case(sel)
            2'b00 : begin
                y = i0;
                x = i2;
                end
            2'b01 : y = i1;
            default : begin
                    x = i1;
                y = i2;
                end
        endcase
    end
    endmodule

Gtk_wave:

    ![incomp_case](partial_case_assign_wave.png)

- when sel=2'b01 the x goes to contsant value of previous output.


![Synthesis Details](img/partial_case_c.png)


### Netlist:

![Netlist Generation](img/partial_case_assign_netlist.png)

![Netlist](img/partial_case_netlist.png)

---

### 3. Overlapping Case:
- This arises due to the overlaping of case conditons that is if multiple case block conditons are true then it get colapsed.

### Example1(lab):

Code:

    module bad_case (input i0 , input i1, input i2, input i3 , input [1:0] sel, output reg y);
    always @(*)
    begin
        case(sel)
            2'b00: y = i0;
            2'b01: y = i1;
            2'b10: y = i2;
            2'b1?: y = i3;
            //2'b11: y = i3;
        endcase
    end

    endmodule

Gtk_wave:

    ![bad_case](img/bad_case_vcd.png)

- here sel=2'b11 the output y goes to contsant value 1 because of mutliple case condition are true.

### Code get Synthesised

![Synthesis Details](img/bad_case_c.png)

- in this d-latch is used

### Netlist:



![Netlist](img/bad_case_netlist.png)

### After Synthesis (Gtk Wave Form):

![synthesised case ](img/bad_case_good_wave.png)

---












