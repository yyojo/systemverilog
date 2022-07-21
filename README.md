SystemVerilog Fundamentals
=================

## Basics
----
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
  b = #1step c; // blocking assign using 1step (eqauls here to 100ps) , after evaluating the right operand, the simulator.
                // will block the statement until one simulation time precision elapses, then it will make the assignent into b.
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
* In while loops, the expression is checked at the beginning.
* In Do...While loops the expression is checked after statements execute.
In Do...While loops the statement block executes at least once.
```sv
while (condition) begin
  •••
end
  
do begin 
  •••
end while(condition)
```

<ins>**While and Do...While Loops Statements**</ins>
The **while** loop excutes a group of statements untill **expression** become false.
* In while loops, the expression is checked at the beginning.
* In Do...While loops the expression is checked after statements execute.
In Do...While loops the statement block executes at least once.
```sv
while (condition) begin
  •••
end
  
do begin 
  •••
end while(condition)
```



