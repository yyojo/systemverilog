SystemVerilog Fundamentals
=================

## Basics
----
### Timing Control
<ins>**Delay Control**</ins>

The delay control is a way of adding a delay between when the simulator encounters the statement and when it executes.
Delay control is achieved by specifying the waiting time to execution when the statement is encountered. The symbol **#** is used to specify the delay.

We can specify the delay based timing control in three ways:

* Regular delay control - It will be specified on the procedural assignment left as a non-zero number.
* Intra - assignment delay control -  In this case, delays will be specified on the assignment operator's right-hand side. The right-hand side expression will be evaluated at the current time, and the assignment will occur only after the delay.
* Zero delay control -  Zero delay control statement specifies zero delay value to the left-hand side of a procedural assignment. This method is used to ensure the statement is executed at the end of the simulation time. It means, zero delay control statement is executed after all other statements in that simulation time are executed.

<ins>**Event Control**</ins>

The event expression allows the statement to be delayed until the occurrence of some simulation event, which can change of value on a net or variable or an explicitly named event triggered in another procedure. Any change in a variable or net can be detected using the **@** event control.

An event controls the execution of a statement or a block of a statement. Value changes on variables and nets can be used as a synchronization event to trigger the execution of other procedural statements and is an implicit event.
The event is based on the direction of change like towards 0, which makes it a negedge and change towards 1 make it a posedge.
* A negedge is a transition from 1 to X, Z or 0 and from X or Z to 0
* A posedge is a transition from 0 to X, Z or 1 and from X or Z to 1

A transition from the same state to the same state does not suppose to be an edge. The edge event can be detected only on the LSB of a vector signal or variable.
If an expression evaluates to the same result, then it cannot be considered as an event. There are different types of event-based controls.

* Regular event control -  Execution of statement will happen on signal changes or at positive or negative transitions of signals. For example, posedge of a clock, and the negedge of reset, etc.
* Named Event Control - The event keyword can be used to declare a named event that can be triggered explicitly. An event cannot hold any data, no time duration, and can be made to occur at any particular time. A named event is triggered by the -> operator by prefixing it before the named event handle. A named event can be waited upon through the @ operator.
* Event OR control -  The transitions of signal or event can trigger statements' execution. The or operator can wait until any one of the listed events is triggered in an expression. The comma (,) can be used instead of the or operator.


### Data Types and Literals

The Verilog datatypes have 4-state values: 0, 1, X, Z. SystemVerilog adds 2-state value types: 


<img width="988" alt="Screen Shot 2022-07-21 at 09 26 57" src="https://user-images.githubusercontent.com/109002901/180144279-e933bcee-43e1-4ffd-8233-64a8f0651a67.png">

In Verilog reg data type is confusing - It can synthesize to a register or net depending on the usage. SystemvVerilog defines register data types as variable, so **var logic** replaces Verilog's **reg**.The keyword **logic** defines that the variable or net (wire) is 4-state data type.

```sv
logic clock;
logic [15:0] reg;
logic [7:0] mem8x32 [0:31]
```

Notice that 4-state variables initializes to X and 2-state variables are initialized to 0. So assigning a 4-state value to a 2-state type turns hign impedance and unknown into zero, and it can't be recovered. In addition, if there is a failure when initialzing design and 4-state variable represent the design state, it shows the design state as X - showing that the design could not be initialize. However in 4-state value it shows the design state as 0 - hides failure to initialize design.

<ins>**Connectivity Characteristics**</ins>

Assigning a SystemVerilog variable:
* In any number of **initial** or **always** blocks
* From a single continiuous assignment
* From a single module out port
* From a single primitive output

SystemVerilog has restrictions on assignments to variables. These restrictions aim to prevent multiple drivers on a signle variable.
* You cannot combine procedural assignments with continuous assignments or module output drivers on the same variable.
* You cannot have multiple continuous assignments or multiple outpout ports drive the same variable.
* Only net types can have multiple drivers.
* Special procedural blocks (always_ff , always_comb. always_latch) don't allow more than a single driver to variables (in regular always block it's allowed).

Thus you can declare most signals to be variable. As the var keyboard is optional, you can declare most signals as **logic**.
```sv
module two (output logic y, // y is logic - assigned in a procedural block.
            input logic a, b); // a, b connected to single module inputs.
  always @ ( a or b )
    y = a && b;
endmodule

module two (output logic y, // y is logic - assigned by a single continuous assignment.
            input logic a, b); // a, b connected to single module inputs.
  assign y = a || b;
endmodule

module test; 
logic a ,b;
logic and_out, or_out; // and_out and or_out are logic - single instance outputs
one m1 (and_out, a, b);
two m2 (or_out, a, b);
initial begin
  a = 0; // a is logic - assigned in an initial block.
  b = 0; // b is logic - assigned in an initial block.
 end
 endmodule
 
 module test;
 logic a, b;
 wire y; 
 one m1 (y, a, b);
 two m2 (y, a, b);
 endmodule
```

<ins>**Unsized Literals**</ins>

An unsized literal is a number with a base specifier but no size specification. 

```sv
logic [5:0] bus; // Unsized literals are also un-based.

bus = '0; // 000000
bus = '1; // 111111
bus = 'z; // zzzzzz
bus = 'x; // xxxxxx
```

<ins>**Time Literals**</ins>
Time literals aren nu mbers wrriten in integer or fixed-point format, followed without a space by a time unit (fs, ps, ns, us, ms, s). It scaled to the current time unit and rounded to the currenct time precision. SystemVerilog also have a 1step delay value (equals to one simulation precision unit , the smallest resolution of all the timescale/ timeprecision declarations in the code).

SystemVerilog introduces the **timeunit** and the **timeprecision** declarations, which are belong to the scope of the module and overwrite the timescale compiler directive for that specific module.


```sv
'timescale 1ns / 100ps // time unit 1ns , precision 100ps

module test;
logic a, b, c, sel , clk;
initial 
begin 
  #20ns sel = 0; // blocking assign.
  #5.18 sel = 1; // rounded to 5.2ns.
  b = #1step c; // blocking assign using 1step (eqauls here to 100ps) , after evaluating the right operand,
                //  the simulator. will block the statement until one simulation time precision elapses, 
                // then it will make the assignent into b.
end

always @(posedge clk)
  #5.1ns a <= a + 1; // nonblocking assign
endmodule
```
### Procedural Statements and Procedural Blocks
For any named block, SystemVerilog allows to repeat the name after the block **end , join or join variant keyword**. SystemVerilog allows a matching name to be specified with the block **end**.

```sv
module test;
initial
  begin : block_A
    •••
    begin : block_B
      •••
    end: block_b
  end : block_A
endmodule : test
```

 In SystemVerilog unnamed block can declare local variables.
 * The variable is visible in the block where it is declared and in any nested blocks.
 * Hierarchical references to the variable cannot be used.

```sv
initial begin // unnamed block
  integer i; // local declaration in unnamed block
  while (i <=10) begin // unnamed block
    $display("i : %d", i);
    if (i < 5) begin // unnamed block
      $display("Number is: %d", i)
    end
    i = i + 1;
  end
end
```

<ins>**For Loop Statements**</ins>

SystemVerilog allows to declare a for loop variable within for statement.
* Variables are visible only in the loop
* The same identifier name cen safely used in multiple loops.

```sv
initial begin
  for (int i = 0 ; i < 10 i = i+1)
    •••
```

<ins>**Foreach Loop Statements**</ins>

This loop iterated over all the elements of an array. 
* Loop variable does not have to be declared
* Only visible inside the loop
* Are read only

```sv
int arr [7:0];
foreach (arr[i]) // i is the loop variable
  arr[i] = i*2;
  
int mat [7:0] [2:0];
foreach (mat[k,l])
  mat[k][l] = k*l;
```

<ins>**While and Do...While Loops Statements**</ins>

The **while** loop excutes a group of statements untill **expression** become false.
* In **while** loops, the expression is checked at the beginning.
* In **Do...While** loops the expression is checked after statements execute.
In **Do...While** loops the statement block executes at least once.
```sv
while (condition) begin
  •••
end
  
do begin 
  •••
end while(condition)
```
<ins>**Repeat Loop Statements**</ins>
A forever loop runs forever

```sv
foever begin
  •••
end
```

<ins>**Forver Loop Statements**</ins>

A given set of statement can be executed N number of times with a **repeat** consruct.

```sv
repeat (N) begin
  •••
end
```

<ins>**Jump Statements**</ins>

SystemVerilog adds the break and continue keyword to control execution of any kind of loop statement.
* **break**
  * Terminates the execution of loop immmediately
  * Usually under conditional control

* **continue**
  * Jumps to the next iteration of a loop
  * Usually under conditional control
```sv
repeat (N) begin
  •••
  if (condition)
    break;
  end
  
foreach (arr[i]) begin
  if (condition)
    continue;
  •••
end
```

<ins>**Case Statements**</ins>

* **priority case**
  * Modifier to the case statement
  * Synthesis: Equivalent to the **full_case** attribute in Verilog
  * Simulation Compile-time / Run-time violation report (probably warning) if no case item expressiong match the case expression
  * Overlapping branches are permitted

```sv
initial begin 
  op = 0;
  
  priority case (op) // First match is excuted
  0 : $display ("Found to be 0");
  0 : $display ("Again found to be 0);
  1 : $display ("Found to be 1);
  endcase
end
```

* **unique case**
  * Modifier to the case statement
  * Synthesis: Equivalent to the **full_case** and **parallel_case** attribute in Verilog
  * Simulation Compile-time / Run-time violation report (probably warning) if no case item expressiong match the case expression
  * No overlapping allowed - Runtime warning if multiple breanches found for a case value
  * One branch must be executed - Runtime warning if no branch found for a case value. Use 

```sv
initial begin 
  op = 0;
  
  unique case (op) // Runtime warning - No overlapping allowed
  0 : $display ("Found to be 0");
  0 : $display ("Again found to be 0);
  1 : $display ("Found to be 1);
  endcase
end

  initial begin 
  op = 0;
  
  unique case (op) // Runtime warning - One branch must be executed
  1 : $display ("Found to be 1");
  2 : $display ("Found to be 2);
  endcase
end
```

<ins>**If-Else if-Else Statements**</ins>

* **priority if**
  * Implies full logic
  * Modifier can be applied to if statements with the same effects
  * Branch conditions are checked sequentially. First match executes
  * Compile-time / Run-time violation report (probably warning) if no branch taken (use unconditional else) 
  * Overlapping branches are permitted

```sv
priority if (contition1) // First match is excuted
  •••
else if (condition2)
  •••
else if (condition3)
  •••
else
  •••
end
```

* **unique if**
  * Implies parallel logic
  * Branch conditions are checked simultaneously. Exactly one branch must match
  * Compile-time / Run-time violation report (probably warning) if no branch taken (use unconditional else)
  * Compile-time / Run-time violation report (probably warning) if multiple branches can be taken

```sv
unique if (contition1) // Exactly one match must match
  •••
else if (condition2)
  •••
else
  •••
end
```

<ins>**Iff Qualifier**</ins>

The **iff** keyword qualifies a procedural event control. The event expression triggers only if the **iff** condition is true.
* **iff** has precedence over **or** - can add parenthesis for clarity
* Limited use in RTL code - latches and gated clocks.
* More useful for embbedded events in verification stimulus code

```sv
always @(trigger iff enable == 1) 
  y <= trigger // creates latch in synthesis
  •••
```

<ins>**Blocks**</ins>
The Verilog **always** block can synthesize to combinational, latched or sequential logic, depending upon you coding style's sensitivity list. SystemVerilog adds implementation specific procedural blocks - **always_comb, always_latch, always_ff**, these block reduce design ambiguty by clearly indicating the hardware intent for a prodecural block.

* **always_comb**
  * SystemVerilog procedural block 
  * Any variable assigned in an **always_comb** cannot be assigned by another procedure
  * Cannot contain further blocking timing or event control 
  * Sensitive to changes in any input to a called function
  * Automatically executed once at time 0 without waiting for an event - after all **initial** and **always** blocks have executed
  
  ```sv
  always_l // no warnings or errors
    if (sel == 1) 
      op = a;
    else
      op = b;

  ---------------
  
  logic op;
  always_comb 
     if (sel1) 
       op = a;
     else
       op = b;
  always_comb // error - op variable assigned by another procedure
     if (sel2) 
       op = a;
     else
       op = b;    
  ```
  * **always_latch**
  * SystemVerilog adds a specialized procedutal block for modeling latched logic
  * Any variable assigned in an **always_lathc** cannot be assigned by another procedure
  * Cannot contain further blocking timing or event control 
  * Sensitivw to changes in any input to a called function
  * Automatically executed once at time 0 without waiting for an event - after all **initials** and **always** have executed
  
  ```sv
  always_latch // no warnings or errors
    if (gate == 1)
      op <= a;
  
  ---------------
  
  always_latch
     if (en1) 
       op <= a;

  always_comb // error - op variable assigned by another procedure
     if (en2) 
       op <= b;  
  ```
  
