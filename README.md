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
An unsized literal is a number with a base specifier but no size specification. 
