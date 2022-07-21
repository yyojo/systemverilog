Systemverilog Fundamentals
=================

## Basics
----
### Data Types and Literals
The Verilog datatypes have 4-state values: 0, 1, X, Z. SystemVerilog adds 2-state value types: 
<img width="988" alt="Screen Shot 2022-07-21 at 09 26 57" src="https://user-images.githubusercontent.com/109002901/180144279-e933bcee-43e1-4ffd-8233-64a8f0651a67.png">

The keyword **logic** defines that the variable or net (wire) is 4-state data type.

### Program Structure 
```sv
logic clock;
logic [15:0] reg;
logic [7:0] mem8x32 [0:31]
```
