SystemVerilog Fundamentals
=================
## Table of Contents

* [Timing Control](#timing-control)
* [Data Types and Literals](#data-types-and-literals)
* [Procedural Statements and Procedural Blocks](#procedural-statements-and-procedural-blocks)
* [Operators](#operators)
* [User-Defined Data Types and Structures](#user-defined-data-types-and-structures)
* [Hirearchy and Connectivity](#hirearchy-and-connectivity)
* [Static Arrays](#static-arrays)
* [Tasks and Functions](#tasks-and-functions)
* [Interfaces](#interfaces)
* [Simple Verification Features](#simple-verification-features)
* [Clocking Blocks](#clocking-blocks)
* [Random Stimulus](#random-stimulus)
* [Basic Classes](#basic-classes)
* [Polymorphism and Virtuality](#polymorphism-and-virtuality)
* [Class-Based Random Stimulus](#class-based-random-stimulus)
* [Interfaces in Verification](#interfaces-in-verification)
* [Covergroup Coverage](#covergroup-coverage)
* [Queues and Dynamic and Associative Array](#queues-and-dynamic-and-associative-array)
* [Assertion Based Verification](#assertion-based-verification)
* [Introduction to SystemVerilog Assertions](#introduction-to-systemverilog-assertions)
* [Interproccess Synchronization](#interproccess-synchronization)


## Timing Control
----
### Delay Control
The delay control is a way of adding a delay between when the simulator encounters the statement and when it executes.
Delay control is achieved by specifying the waiting time to execution when the statement is encountered. The symbol **#** is used to specify the delay.

We can specify the delay based timing control in three ways:

* Regular delay control - It will be specified on the procedural assignment left as a non-zero number.
* Intra-assignment delay control -  In this case, delays will be specified on the assignment operator's right-hand side. The right-hand side expression will be evaluated at the current time, and the assignment will occur only after the delay.
* Zero delay control -  Zero delay control statement specifies zero delay value to the left-hand side of a procedural assignment. This method is used to ensure the statement is executed at the end of the simulation time. It means, zero delay control statement is executed after all other statements in that simulation time are executed.

### Event Control
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


## Data Types and Literals
----
The Verilog datatypes have 4-state values: 0, 1, X, Z. SystemVerilog adds 2-state value types: bit, byte, shortint, int and long int: 

<img width="988" alt="Screen Shot 2022-07-21 at 09 26 57" src="https://user-images.githubusercontent.com/109002901/180144279-e933bcee-43e1-4ffd-8233-64a8f0651a67.png">

In Verilog reg data type is confusing - It can synthesize to a register or net depending on the usage. SystemvVerilog defines register data types as variable, so **var logic** replaces Verilog's **reg**.The keyword **logic** defines that the variable or net (wire) is 4-state data type.

```sv
logic clock;
logic [15:0] reg;
logic [7:0] mem8x32 [0:31]
```

Notice that 4-state variables initializes to X and 2-state variables are initialized to 0. So assigning a 4-state value to a 2-state type turns hign impedance and unknown into zero, and it can't be recovered. In addition, if there is a failure when initialzing design and 4-state variable represent the design state, it shows the design state as X - showing that the design could not be initialize. However in 4-state value it shows the design state as 0 - hides failure to initialize design.

### Connectivity Characteristics
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

### Unsized Literals
An unsized literal is a number with a base specifier but no size specification. 

```sv
logic [5:0] bus; // Unsized literals are also un-based.

bus = '0; // 000000
bus = '1; // 111111
bus = 'z; // zzzzzz
bus = 'x; // xxxxxx
```

### Time Literals
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

## Procedural Statements and Procedural Blocks
----
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

### For Loop Statements
SystemVerilog allows to declare a for loop variable within for statement.
* Variables are visible only in the loop
* The same identifier name cen safely used in multiple loops.

```sv
initial begin
  for (int i = 0 ; i < 10 i = i+1)
    •••
```

### Foreach Loop Statements
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

### While and Do...While Loops Statements
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

### Repeat Loop Statements
A given set of statement can be executed N number of times with a **repeat** consruct.

```sv
repeat (N) begin
  •••
end
```

### Forever Loop Statements
A forever loop runs forever

```sv
foever begin
  •••
end
```

### Jump Statements
SystemVerilog adds the break and continue keyword to control execution of any kind of loop statement.
* **break**
  * Terminates the execution of loop immediately
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

### Case Statements
* <ins>**priority case**</ins>

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

* <ins>**unique case**</ins>

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

### If-Else if-Else Statements

* * <ins>**priority if**</ins>

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

* <ins>**unique if**</ins>

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

### Types of Blocks
The Verilog **always** block can synthesize to combinational, latched or sequential logic, depending upon you coding style's sensitivity list. SystemVerilog adds implementation specific procedural blocks - **always_comb, always_latch, always_ff**, these block reduce design ambiguty by clearly indicating the hardware intent for a prodecural block.

* <ins>**always_comb**</ins>

  * SystemVerilog procedural block 
  * Any variable assigned in an **always_comb** cannot be assigned by another procedure
  * Cannot contain further blocking timing or event control 
  * Sensitive to changes in any input to a called function
  * Automatically executed once at time 0 without waiting for an event - after all **initial** and **always** blocks have executed
  
```sv
always_comb // no warnings or errors
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
* <ins>**always_latch**</ins>

  * SystemVerilog adds a specialized procedutal block for modeling latched logic
  * Any variable assigned in an **always_latch** cannot be assigned by another procedure
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

always_latch // error - op variable assigned by another procedure
   if (en2) 
     op <= b;  
```
  
* <ins>**always_ff**</ins>

  * SystemVerilog adds a specialized procedutal block for modeling reagistered logic
  * Any variable assigned in an **always_ff** cannot be assigned by another procedure
  * Contains one and only one event control
  * Cannot conation any block timing - for example can't have #delay inside the block
  
```sv
always_ff @ (posedge clk or posedge rst)
  if (rst)
    op <= 1'b1;
  else
    op <= ip;
```

## Operators
----
### Assignments Types
* Blocking Assignments -  Blocking assignments statements are assigned using **=** and are excuted one after the other in a procedural block. However, this will not prevent exexution of statements that run in parallel block.
* Non-blocking Assignments - Non-blocking assignments allows assignments to be scheduled without blocking the execution of the following statements and is specified by a **<=** symbol. It's important to note that the same symbool is used as a relational operator expressions and as an assignment operator in the context of a non-blocking assignment.

### Assignment Operators
Operators that join an operation along with a blocking assignment to the first operand of the operator.

![Screen Shot 2022-07-23 at 20 32 39](https://user-images.githubusercontent.com/109002901/180616360-c9eceb33-75af-4fb0-a83c-2a8d52b3d3f2.png)

You can't use them in any block where you are making an assignment to  storage device, and there is any chance that there may be a race between this block make the assignment and some other block reading the value. You should always use a non-blocking assignment. 
Assignment operators are **blocking assignments** , therefore they are suitable for RTL combinational logic, temporary variables in RTL sequential code or testbench.

### Pre- and Post- Increment/Decrement Operators
These combine a blocking assignment and an increment or decremnet operator.
* Pre-form **++variable , --variable** adds or subtracts and then uses the new value.
* Post-form **variable++ , variable--**  uses the new value and then adds or subtracts.

```sv
initial begin
  b = 1;
  a = b++; // post a = 1, b = 2
  a = ++b; // pre a = 3 , b = 3
  a = b--; // post a = 3, b = 2
  a = --b; // pre a = 1 , b = 1
end
```
  
### Wildcard Equivalence Operator
The wild equality operator **==?** and inequality operator **!=?** treat X and Z values in a given bit position as a wildcard. A wildcard bit matches any bit value (0, 1,Z, or X) in the value of the expression being compared against it.
* An X, Z, or ? in the right operand matches any value in the left
* Asymmetric - only right side can have wildcatd bits

```sv
a = 4'b0101;
b = 4'b01XZ;
  
if (a == b) // unknown
  •••
if (a === b) // false
  •••
if (a ==? b) // true
  •••
if (a !=? b) // flase
  •••
if (a ==? 4'b?1?1) // true
  •••
```
Remember that Verilog has two (in)equality operators:
* Logical (in)equality **==** 
* Identity (in)equLITY **===**

The difference is that equivalence sign can have an unknown value. (if a == b and b has some unknowns -> unknown, but if  a === b and b has some unknowns in it , the result will be true if a also has unknowns in that particular bit positions, else result is false).

### Set Membership Operator (inside)
**inside** does a comparison:
  * True if expression value is contained within a values list
  * List values can be variables (including arrays) and ranges
  * List values can hve wildcards
  * List values can have overlap

```sv
if (variable indside {2'b01 , 2'b10}) // equivalent to 
                                      // if ((a == 2'b01) || (a == 2'b10))
  •••
  
int bar [1:0] = '[5,6];
if (variable indside {bar , [2;0]}) // equivalent to 
                                    // if (variable inside {bar[1] . bar[2] , 0 , 1 , 2})
  •••
  
if (variable indside {2'b0?}) // equivalent to 
                                    // if (variable inside {2'b00 , 2'b01 , 2'b0X , 2'b0Z})
  •••
```
You can also use **inside** with the **case** statement:

```sv
bit [3:0] a;
case (a) inside
  0 , 2 : $display ("0 or 2);
  [3:5] : $display ("3 or 4 or 5");
  1 , [6:7] : $display ("1 or 6 or 7");
  default : $display ("value > 7");
endcase
```
### Operators Precedence and Associativity

<img width="1929" alt="Screen Shot 2022-07-24 at 09 23 13" src="https://user-images.githubusercontent.com/109002901/180635092-88ef5fa7-d268-47c3-bb9f-153cc82f2def.png">

### Array Assignment Patterns
* Define a list of values for an assignment to array elements and structure fields
* It is equivalent to individual assignments - un-sized literals are allowed
* Patten "keys" can be used - arrays element index , **default** keyword
* There must be a pattern value for every array element

```sv
int arr [3:0];

arr = '{0,1,2,3}; // forward apostrophe before {•••} used for array assignment patterns
/* equivalent to 
arr[3] = 0;
arr[2] = 1;
arr[1] = 2;
arr[0] = 3;
*/

arr = '{3:1 , default : 0} // index:value
/* equivalent to 
arr[3] = 1;
arr[2] = 0;
arr[1] = 0;
arr[0] = 0;
*/
```

## User-Defined Data Types and Structures
----
### User-Defined Types
* Primitive Data Types - Built-in types not constructed from other data types (bit , integer, real ,logic)
* User-Defined Types - SystemVerilog's data types can ve extended with user-defined types using **typedef**

User-defined data types are:
* Types named after **typedef**
* Types constructedfrom other types such as vectors and arrays
* Enumerate (**enum**) types for user-defined value sets
* Structure (**struct**) types to bundle multiple variables into one object
* **union** types to store different data types in the same space
* Placing **__t** after typedef is a valuable convention for reminding you that this is a user-defined type
* The declaration must be visible in the scope where the names is used
* The type name can be used anywhere, the type declaration is legal
* An un-named type is known as an anonymous type

```sv
typedef logic [7:0] vec; // typedef is used to give user-defined name to existing
                         // logic data type
•••
vec vec1, vec2; // the named data type can be used now to declare other variables

logic [16:0] wordvar // anonymous type
```

### Enumerated Types
An enumerated type is set of integral named constants.
* It is a type with user-defined values - value names must be unique in the current scope
* Declared with the **enum** keyword - named or anonymous
* Enumerated types are strongly types
* Should only be directly assigned - a named balue of the type , a variable of the same type
* Naming the type (**typedef**) permits typecasting - **type_name' (value)**


```sv
typedef enum {idle , start , pause , done} state_t; // named enum type
state_t state, next_state; // idle = 0 ; start = 1 ; pause = 2 ; done = 3

enum {idle , start , pause , done} astate;

state = idle;
state = next_state; 
state = 2'b00; // not legal direct assignment of Integral value to enum variable is not allowed
state = 2; // not legal direct assignment of Integral value to enum variable is not allowed
state = state_t'(2); // pause
state = state_t'(8) // no name
a_int = state * 2; // 16
```
* Default base type of an **enum** is **int**
* Default values increment from zero in declaration order
* An enumerate variable can hold any value from the base type - even if there is no matching enumerate value name. Casting does not check value
* An enumerated type is the base type with mnemonics for specific values
* An enumerate variable in expression is replaced by its base type value

<ins>**Assigning Enumerate Value Encodings Explicitly**</ins>

Explicit encoding can be included in an enumerated type declaration, either typed or anonemous which allows one-hot , grey or other encodings to be defined for enumerated values.
* You can define explicit encoding for values - allows one-hot coding 
* You can mix explicity and implicit value encoding - implicit values increment from previous explicit value
* Value encodings must be unique - compliation error is repoorted if encodings overlap

```sv
typedef enum {idle = 1 ,
              start = 2 , 
              pause = 4 , 
              done = 8} state_t;

enum {s0 = 0 , s1 , s2 = 6 , s3} encs; // s1 = 1 - index 1 , s3 = 3 , index 3 

enum {p0 = 0 , p1 ,p2 = 1 , p3} bad_encs // p2 duplicates p1 encoding
```

You can use range notation for simple value name sequences and can mix ranged and unranged values.

```sv
typedef enum {go , R[3:5] , stop} seq_t;
// equivalent to 
typedef enum {go , R3 , R4 , R5 , stop} seq_t;
```

<ins>**Assigning Enumerate Base Type Explicitly**</ins>

* You can specifiy the enumeration base type - base type in by default
* This allows you to constrain the enumerate value range
* The default value encoding will match the specified base type
* Explit value encoding mush match type and length
* An explicit base typ makes enumerates easier and safer to use

```sv
typedef enum bit[1:0] {idle , start , pause , done} state_t; 
// idle = 2'b00 ; start = 2'b01 ; pause = 2'b10 ; done = 2'b11
// enum declaration with explicit base type and implicit encoding

typedef enum bit[3:0] 
             {idle = 3'b000 ,
              start = 3'b001 , 
              pause = 3'b010 , 
              done = 3'b011} state_t;
// enum decleration with explicit base type and explicit encoding
```

<ins>**Enumerated Base Type vs. Enum Initial Value**</ins>

* The initial vaule of an enum variable depends on its base type - 0 for default int type and any 2-state type and X for explicit 4-state type
* You can explicity encode a value with X - typically to indicate an illegal value

```sv
enum bit[2:0] {DOG , CAT} animals; 
initial begin 
  $display ("value %b", animals);
  $display ("name %s" , animals.name()); 
  // value - 000 , name - DOG

enum logic[2:0] {RED , BlUE} colors; 
initial begin 
  $display ("value %b", colors);
  $display ("name %s" , colors.name()); 
  // value - xxx , name - EMPTY STRING
 
enum logic[2:0] {Jack = 3'bX , Nick = 3'b001} names; 
initial begin 
  $display ("value %b", names);
  $display ("name %s" , names.name()); 
  // value - xxx , name - Jack
```

<ins>**Enumerated Type Access Methods**</ins>

<img width="786" alt="Screen Shot 2022-07-24 at 14 47 06" src="https://user-images.githubusercontent.com/109002901/180645513-9d22bc10-1b23-48d8-a7a6-92f55897ac9b.png">

```sv
typedef enum {YELLOW , RED ,BLUE , GREEN} colors;
colors color = colors.first();

repeat(color.num() + 1) begin 
  $display("%s = %0d" , color.name() , color);
  if (color = color.last())
    $display("----");
  color= colors.next();
end
```

### Structures
A structure represent a collection of data types that can be refrenced as a whole, or the individual data types that make up the structure can be referenced by name.
* Declare using **struct** keyword
* Use the "dot" notation to access individual fields
* You can make pattern assignments to structures - orded or keyed , but not both in the same pattern. Keys can be by name, type, default or a mix of these
* You can declate arrays of structures
* Structures can be nested

```sv
typedef struct { logic id,par ;
                 logic [3:0] address ;
                 logic [7:0] data;
                } frame_t;

frame_t frame1;
logic [7:0] data_in;

// individual field access 
frame1.id = 1'b1;
data_in = frame1.data;

// ordered assignment pattern
frame1 = '{1'b1 , 1'b0 , 4'b1 , 8'b1010}

// named assignment pattern
frame1 = '{id:0 , par:1 , address: 0 , data: 1};

frame two_frame_arr [1:0];
// nested ordered assginment pattern
two_frame_arr = '{'{0,0,0,225} , '{1,1,1,0}};
```

Structure can use either ordered or keyed assignment patterns for assigning values to structure elements.
Order of precedence:
1. Name - assign fields by name
2. Type - assgin remaning fields of the specific type
3. Default - assign remaing fields using default. Value must be type compatible with all remaining fields

All fields mush have an assignment.

```sv
typedef struct { logic id,par ;
                 logic [3:0] address ;
                 logic [7:0] data;
                } frame_t;

frame_t frame1;

// assignment by order
frame1 = '{1'b1 , 1'b0 , 4'b1 , 8'b1010}

// assignment by name 
frame1 = '{id:0 , par:1 , address: 0 , data: 1};

// assignment by name and type
frame1 = '{id:0 , address: 0 , logic : 1};

// assignment by name, type and default
frame1 = '{id:0 , logic : 1 , default : 0};
```
<ins>**Packed Structure**</ins>

Packed structure feilds are packed together in memory without gaps ans can be use as a whole arithmetic and logical operators.
* Structures can be packed
* All fields must be packable - integrals values only and any fields that is array or structure mush also be packed
* A packed structure can be treated as a one dimentional vector - can still use dot notation to access individual structure fields
* If any field is 4-state then the whole structure is stored as 4-state - conversions are done automatically upon read/write
* You cannot slice unpacked structure
* Avoid mixing 2-state and 4-state types

<img width="1025" alt="Screen Shot 2022-07-24 at 16 05 50" src="https://user-images.githubusercontent.com/109002901/180648339-a9602a6d-ffa3-438c-844f-1a54578db495.png">

```sv
typedef struct packed { logic id,par ; // all fields a 4-state
                 logic [3:0] address ;
                 logic [7:0] data;
                } frame_t;

frame_t frame1;
logic [3:0] buff;
initial begin
  buff = frame1.address; 
  buff = frame1[11:8]; // same field as frame1.address
```

### New Type Declaration Regions
New Type Declaration Regions are spaces used to share parameters, data, type, task, function, sequence, property and checker declaration among multiple SV modules interfaces, programs and checkers
* You can declare ports of user-defined types, but you mush declare types before you use them
* SystemVerilog has two new name spaces - packages and Compliation Unit Scope (CUS) (to be avoided)

<ins>**Compliation Unit Scope (CUS)**</ins>

* New scope for declarations is outside of design elements (modules, interfaces, etc.)
* Default scope is a single file - tools options can change this, for example: defining scope as multiple files compiled at same time
* Scope is unnamed - cannot make hierarchical references to declarations
* Local declarations override declarations in CUS - use **$unit::** to access CUS declrations
* Scope of compliation unit is tool-specific and can change between compile sessions. Local declarations override CUS declarations.

```sv
// declaration CUS
typedef enum {start , done} status_t;

module top // NOT RECOMMENDED
  (input status_t status,
  output logic [7:0] out);
  •••
```
<ins>**Packages**</ins>
* New design element similar to module - must be cimpiled separately and must be compiled before elements that reference the package
* They contain declarations to be shared between elements (type, variables, subroutines..)
* You can **import** package declarations - explicit - specifically namrf and implicit - all using wildcards ( * )
* Or directly access a declaration using the resolution operator **::** - does not require import

```sv
// package declaration
package mytypes;
  typedef enum {start , done} status_t;
  •••
endpackages : mytypes
```

```sv
import mytypes::status_t; // explicit import
module top1 (input status_t status);
  •••

module top2 import mytypes::* ; // wildcard import
  (input logic [7:0] in,
  output status_t status);
  
  status_t status_task;
  •••
  
module top3 (•••);
  mytypes::status_t status
  •••
```

## Hirearchy and Connectivity
----
Verilog has to ways to connect ports:
1. Ordered Port Connections - have the risk of incorreclty ordered connections
2. Named Port Connections - safer but ver verbose (.port_name(signal_name))

SystemVerilog adds to options to simplify port connections: **.name** and ** .* **.

### Implicit Port Connection - .name
* Where signal and port names match, just use **.clk = .clk(clk)**
* It can be mixed with a named connection - for ports where names do not match
* Signals used in **.name** must be explicity declared

```sv
module count (
  input logic clk, rst, ld, 
  inputy logic [7:0] data, 
  output logic [7:0] cnt' 
  output logic val);
  •••
endmodule

// .name port connection
count c1 (.clk , .rst , .ld , .data , .cnt , .val);

// named equivalent
count c2 (.clk(clk) , .rst(rst) , .ld(ld) , .data(data) , .cnt(cnt) , .val(val));

// .name and named
count c3 (.data , .clk , .rst(reset) , .ld(load), .cnt , .val(val));
```

### Implicit Port Connection - .*
* Where all signal and port names match, just use ** .* **
* It automatically connects ports to signals of the same name
* There is safety of named connection without the verbositiy - can lose some port list readability
* It can be mixed with a named connection - for ports where names do not match
* Signals used in ** .* ** must be explicity declared

```sv
module count (
  input logic clk, rst, ld, 
  inputy logic [7:0] data, 
  output logic [7:0] cnt' 
  output logic val);
  •••
endmodule

// .* port connection
count c1 (.*);

// .name equivalent
count c2 (.clk , .rst , .ld , .data , .cnt , .val);

// .* and named
count c3 (.* , .rst(reset) , .ld(load));
```

### Rules for Using .name and ,*
* In an instatiation:
  * Ordered connections with **.name** or ** .* ** cannot be mixed
  * **.name** and ** .* ** connections cannot be mixed
  * Named and **.name** connections can be mixed
  * Named and ** .* ** connections can be mixed

* Named Connections are required for:
  * Port/signal name mismatches
  * Port/signal width mismatches
  * Unconneceted ports

```sv
count c1 (clock , reset , .*); // error - ordered and .*

count c2 (.clk , .rst , .*); // error - .name and .*

count c3 (.data , .clk , .cnt , .val , .rst(reset) , ld(load)); // .name and named

count c4 (.* , .rst(reset) , .ld(load));
```

<img width="1870" alt="Screen Shot 2022-07-24 at 17 41 23" src="https://user-images.githubusercontent.com/109002901/180652229-6f27121a-c84d-461e-9d36-8f559cb1f3e1.png">

### Explicit Import 
An explicit impot only imports the symbols speciciallty ferenced by the import.
* Explicit import directly loads declaration into module
* Declaration must be unique in the current scope
* Only the sybols specifically referenced are imported
* An explicit import in a CUS follows compliaiton unit rules - local declarations override CUS declaration (even for explicit import). CUS should be avoided if possible

```sv
package pac;
  localparam int a = 10;
  typedef enum {start , stop} mode_t;
endpackage : pac

module top (•••);
  import pac::a;
  import pac::mode_t;
  
  logic [7:0] a; // error - local a clashes with pac::a
  mode_t mode;
  •••
  if (mode == stop); // error - unkown identifier (use pac::stop)
```

However in CUS import: 

```sv
import pac::a;
module top (•••);  
  logic [7:0] a; // local a takes precedence
  •••
```

### Wildcard import
A wildcard import allows al identifiers declared within a package to be imported provided the identifier isnot otherwise defined in the importing scope. Wildcard import makes declaration candidate for import:

* Local or explicity imported declarations can override wildcard imported declarations 
* Package declarations still visible through resolved name
* Local or explicity imported declarations take precedence over wildcard imports

```sv
package pac;
  localparam int a = 10;
  typedef enum {start , stop} mode_t;
endpackage : pac

module top (•••);
  import pac::*; 
  
  logic [7:0] a; // local a overrides with pac::a
  mode_t mode;
  •••
  if (mode == stop); // all package declerations visible
  •••
  $display ("%h", pac::a); // resolved name to access package declaration
```

### Import and Export Statements
* The import is a statement that is equivalent to data declaration
* Declration are only visible from the import onwards
* Fewer visibility issues
* Better readability
* Import do not chain - importing P1 into P2 and then P2 into top does not make P1 declarations visible in top. P1 mush be separately imported into top or add an export to P2 to make P1 visible as part of P2 ** export P1::* ** (rarely used)
* Place import at the beginning of the desgin element

```sv
package P1;
  int a1 = 5;
  •••
endpackage : P1

package P2;
  import P1::*;
  int a2 = a1;
  •••
endpackage : P2

module top (•••); 
  logic [7:0] d = a2; // error - a2 undefined
  
  import P2::*;
  initial begin
    $display("%0d" , a2);
    $display("%0d" , a1); // error - a1 undefined
endmodule
```

### Ambiguity and Resolved Names
How to resolve ambiguous references due to: "Multiple Declarations in separare packages using same name"
Solotions:
* Try to avoid duplicate names
* Structure packages carefu;;y - only required declaration visible
* Use resolved names
* Use explicit imports

```sv
package P1;
  int a1 = 5;
  •••
endpackage : P1

package P2;
  int a1 = a8;
  •••
endpackage : P2

// solution 1 
module top (•••); 
  import P1::* , P2::*;
  logic [7:0] d = a1; // error - ambiguous
  logic [7:0] e = P2::a1 // OK resolved
endmodule

// solution 2
module top (•••); 
  import P1::* , P2::*;
  import P2::a1; // explicity importing a1 from package you want to always infer
  logic [7:0] d = a1; // OK resolved
endmodule
```

### Importing Packages in the Module Header
How import packages declarations that may be used in ANSI C module port lists ? **import** inside module is insufficient. There are several solutions:
* Use resolved name - inefficient for multiple declarations
* Move import to COS - but we're trying to avoid using this scope
* Import package as part of the module header - imported before parameter and port lists and import clause must be terminated with a semicolon


```sv
package P1;
  typedef enum {start,stop} mode_t;
endpackage : P1

// solution 1 
module top (
  input mode_t mode, // error - mode_t undefined
  •••);
  import P1::*; 
  •••
endmodule

module top import P1::*;( // import in module header, must include semicolon
  input mode_t mode, 
  •••);
  •••
endmodule
```

## Static Arrays
----
### Arrays
* SystemVerilog allows arrays of any type including : events, structures and enumerates
* Standard, Verilog-like arrays are defined as unpacked - index declared ather name and elements are stored separately
* Assignment patterns can be used to assign unpacked arrays

```sv
int mat [2:0] [1:0]; // 2D array of int 

typedef enum bit [1:0]; {s0 ,s1, s2, s3} seq_t;
seq_t seq5 = [4:0]; // array of enums

logic [7:0] unpack [1:0]; // unpacked array of bytes

// logic [7:0] [1] equivalent to unpack [1]
// logic [7:0] [0] equivalent to unpack [0]

// array assignments 
unpack = '{8'h00 , 8'hff};
unpack = '{default , 8'hff};
```

#### Unpacked Multidimensional Arrays
The term unpacked array is used to refet to the demensions declared after the data identifier name.
* Unpacked dimensions are stored separately
* You can read and write any slice from one element to a whole array - mush match size ,layout and type. Indexing precedence: left to right 
* Assignment patterns are useful for loading unpacked arrays - keys or ordered , can be nested and also contain expressions, for example: repetition

```sv
int mat1 [2:0] [1:0] // 3x2 array of int
int mat2 [1:3] [2:1] // 3x2 array of int
int mat3 [15:0] [0:255] 
•••
mat1 = mat2; // both 3x2 arrays of int

mat1[2] = mat[3] // both 2x1 arrays of it 

mat1 = '{'{2,1} , '{5,4} , '{3,2}}; // nested orders assignment pattern
// mat1[2][0] = 2
// mat1[2][1] = 1
// mat1[1][0] = 5

mat2 = '{3{'{1,2}}}; // pattern expression
// mat2[1][2] = 1
// mat2[1][1] = 2
// mat2[2][2] = 1

mat3 = '{default:100} // keyed assignment
// mat3[0][0] = 100
```

### Packed Arrays
A packed array is mechanism for subdeviding a vector into subfields, witch can ve conveniently access as array alements.
* Packed arrays - index decalred before name, stored as a single vector. they also can be operatred upon and access as a single vector and indexing precedence from left to right
* Only signel-bit data types can be packed - bit , logic ,reg , and recursively, other packed objects of these types
* Packed arrays obey assignment rules for padding and trunction
* A packed array is equivalent to a vector with predefined bit fields
* Generally, only fully packed arrays are synthesizable (not mixed packed and unpacked)

```sv
logic[1:0] [7:0] packarr; // packed array of bytes
logic[7:0] short = 8'hAA; 
logic[23:0] long = 24'h5555555
•••
packarr = 16'h0'
packarr = packarr << 1;
packarr = ~packarr;
•••
packarr = short; // 16'h00aa
packarr = long // 16'h5555
```

packarr look like this:

<img width="665" alt="Screen Shot 2022-07-25 at 12 40 47" src="https://user-images.githubusercontent.com/109002901/180747543-80e6ffb5-f8da-4ba0-93fe-350c47453ba1.png">

## Tasks and Functions
----
In Verilog :
* Tasks - any number of input, output ot inout and i may contain timing; @ (•••) , delay , wait
* Functions - one ore motre input arguments only, no timing allowd (function must completee in zero time)
* Returns single vaulue - vector , real or integer

* Static Subroutines - only one copy exists and concurrent calls can overwrite each other's arguments and variables
* Automatic Subroutines - new copy for every call, concurrent calls each have their own set of arguments and variables, arguments assignment must be blocking

### Optional begin / end and Named Ends
* You can omit the **begin** and **end** keywords
* You can repeat the subroutine name with **endtask** or **endfunction** - SystemVerilog verifies that it is the same name 

```sv
function [7:0] flip (input [7:0] word
  •••
endfunction : flip
```

### Void Functions
* You can daclare functions that do not reaturn a value - define return type **void**
* You can call a a void function as a statement - all restrictions still apply 
* void can also be used to discard a function return value - by casting the return type to **void**

```sv
function void printer (input int errs);
  if (errs = 0)
    $display ("No Errors");
  else
    $display ("%0d Errors", errs);
endfunction
endfunction : flip
```

### Function Output Arguments 
* You can provie **input , output** and **inout** function arguments - argument default is **input** of type logic, **output/inout** arguments let functions return multiple values, **output/inout**  arguments let void functions return values
* Function becomes a general-purpose synthesizable subroutine

```sv
function void adder (input [7,0] a,b 
                       output carry); // the output is change by reference
                       
  logic [7:0] result;
  {carry, result} = a + b;
  
 end function
 
 logic [7:0] a, b, sum;
 logic cry;
 
 sum = adder(a , b, , cry) // sum = result, cry = carry (changed by reference)
```

```sv
function void adder (integer a,b 
                       output integer sum); // the output is change by reference
  sum = a + b;
endfunction

always @ (a , b) // return nothing, sum changed
  add (a ,b ,sum)
```

### Default Argument Values
* You can specify default values for subroutine arguments
  * SystemVerilog uses defauld values for those not paases - with rules
  * Must have unambiguous mapping of actuals to formals
  * It is an error for a formal parameter to not have an unambiguous value 

```sv
// default argument values in task definition 
task read (int j = 0 , int k , int data = 1);
•••
endtask

int val = 21;

// invocation of task with default arguments 
read ( , 5); // equivalent to (0, 5, 1)
read (2 , val); // equivalent to (2, 21, 1)
read (2); // eror - k not defined
```

### Argument Binding by Name
* You can bind aubroutine arguments by name instead of by position
  * Similar to port connection by name 
  * Simplifies calling subroutines having many argumnents
  * Simplifies calling subroutines having default argument

```sv
// default argument values in task definition 
task read (int j = 0 , int k , int data = 1);
•••
endtask

int val = 21;

// invocation of task with default arguments and name passing
read (.j(4) , .k(val) , .data(7); // equivalent to (4, 21, 7)
read (.j(2) , .l(val));  // equivalent to (2, 21, 1)
read (.k(3)); // equivalent to (0, 3, 1)
read (.data(7) , .j(4) , .k(3));
```

### Optional Argument List 
* You can omit passing arguments if all arguments have default values
* If there are no required arguments, you can even omit the parentheses for :
   Tasks calls 
   Void fumction calls
   Non recursive calls to class methods

```sv
// default value for every argument
task read (int j = 0 , int k = 2 , int data = 1);
•••
endtask

// invocation of task with defaukt arguments 
read(); // equivalent to (0,2,1)
read // also legal in SystemVerilog
```

### Explicit return
* Executing a **return** statement immediattely exit a subroutine
  * Allows easier structuring of subroutines
* In a function you can include an expression to return a function value

```sv
function integer mult (input integer num1, num2);
  if ((num1 == 0) || (num2 == 2) begin 
    $display ("Zero multiply");
    return 0; 
  end 
  else
    mult = num1 * num;
endfunction
```

### Argument Passing by Value
For the code shown below, will this work as expected? 
* This is the default method of argument passing
* Its is the same as in Veriloog 2021 but can also pass new data types: array, structures, unions, classes, etc.
* Values copied in on call - subroutines then unaware of input changes
* Values copied out on return - Environment meanwhile unaware of output changes
* Issue for testbench tasks

```sv
logic reg; 
logic ack;
logic [7:0] data;

task cpu_drive_bad (input logic [7:0] write_data,
                    input logic data_valid, 
                    output logic data_read, 
                    output logic [7:0] cpu_data);

  #5 data_read = 1;
  wait (data_valid == 1); // The wait statement blocks the current process until the expression becomes true.
                          // input changes not detected
  #20 cpu_data = write_data;
  wait (data_valid == 0);
  #20 cpu_data = 8'hzz; // only last output updates seen
  data_read = 0;
  
endtask : cpu_drive_bad
•••
cpu_drive_bad (8'hff, ack, req, data); // fornal arguments mapped to actuals in call
•••
```

## Solution 1 - Variable Access by Side-Effect 
* This is a traditional solution to the argument passing issue 
  * Task directly read/writes variables
  * Only literals are passed as arguments 
* Variables mush be of same scope as task declaration
  * Task cannot be declared in a package
  * Reuse of tack is limited 
* It is used extensively for class methods

```sv
logic reg; 
logic ack;
logic [7:0] data;

task cpu_drive_side (input logic [7:0] write_data);

  #5 reg = 1; 
  wait (ack ==1);
  #20 data = write_data; // inputs read directly
  wait (ack ==0);
  #20 data = 8'hzz;
  req = 0; // outputs driven directly
  
endtask : cpu_drive_side
•••
cpu_drive_side (8'hff); // only literals in call
•••
```

## Solution 2 - Argument Passing by Reference
* Use pass by reference by declaring arguments using **ref**
  * Use **ref** instead of parameter direction
  * Only variables not net
* It creates a link between actual and formal arguments
  * Changes in inputs can be seen in task
  * Changes in outputs are immediately updated
* Task mush be declared as automatic

```sv
task cpu_drive_ref (input logic [7:0] write_data,
                    ref logic data_valid, 
                    ref logic data_read, 
                    ref logic [7:0] cpu_data);

  #5 data_valid = 1; // all output updates seen
  wait (data_read ==1);
  #20 cpu_data = write_data; // inputs changes directly
  wait (data_read ==0);
  #20 cpu_data = 8'hzz;
  data_valid = 0; 
  
endtask : cpu_drive_ref
•••
cpu_drive_ref (8'hff, req, ack , data); // fornal arguments mapped to actuals in call
•••
```

### Using const with Reference Arguments
* It can be used only with automatic (not static) subroutines - risk of outdated reference
* Without direction, arguments may be wrriten in error - use **const_ref** to prevent the subroutine from modifying argument 
* Its is also useful for efficiency gains - when arguments occupy large amounts of memory

```sv
task automatic create_crc (ref logic [255:] payload [1:0] •••);
  payload = '0; // var = 'num - fill the bus with num , no matter the length
endtask
  
task automatic create_crc (const ref logic [255:] payload [1:0] •••);
  payload = '0; // constant argument cannot be modified
endtask
```

## Interfaces
----
A SystemVerilog interface is, at its most basic level, simply a bundle of external nets and/or variables noramally shared by one or more modules by virtue of port connections. SystemVerilog provides the interface construct to:
* Encapsulate communication between hardware blocks
* Provide a mechanism for grouping together multiple signals into a single unit that can be passes around the design hierarchy
* Enable abstraction in RTL design

### More Facts on Interfaces
You declare an interface as a heiratchical opject, mush like a module:
* You instantiate it in a module, like any other heirarchical block
* You use the interface like a type in port lists and port maps

A simple interface is just named bundle of nets or variables:
* Similar to a **struct** type
* You can reference the nets or variables where needed

Interfaces can also contaion module-like features for defining signal relationships:
* Continuous assignments, tasks, function, initial/always block, etc.
* Can further instantiate interfaces
* Can be generated or arrayed
* Cannot declare or instantiate module specific items: modules, primitives ,specify blocks and configurations

### How to Bind Interface Instance to Module Portsof Interface Type
SystemVerilog port mapping rules also apply to interfaces.
* You can map by position (order) or name
* You can use **.name** mapping if instance name matches port name
* You can use ** .* ** mapping id all names match


### How to Access Items Through Module Ports Bound to the Interface Instance
For a module with an **interface** **port** you access interface items using the port name as a prefix.

```sv
interface ifa; 
  logic req, start, gnt, rdy;
  logic [1:0] mode;
  logic [7:0} addr;
  wire [7:0] data;
  •••
endinterface : ifa
```

```sv
module memory (input bit clk , ifa bus); // pass the interface as port bus
  reg [31:0] mem [0:31];
  logic read, write;
  assign read = (bus.gnt && (bus.mode == 0) );
  assign write = (bus.gnt && (bus.mode == 0) ); // access the interface item using module port name
  always @(posedge clk)
    if (write)
      mem[bus.addr] = bus.data;
    assign bus.data = read ? mem[bus.addr] : 'z;
endmodule
```

### How to Access Interface Items Through the Interface Instance Identifier 
For a module with an **interface** **instance** you access interface items using the port name as a prefix.

```sv
interface ifa; 
  logic req, start, gnt, rdy;
  logic [1:0] mode;
  logic [7:0} addr;
  wire [7:0] data;
  •••
endinterface : ifa
```

```sv
module top;
  logic clk;
  
  ifa bus(); // decalre interface instance bus
  
  memory mem (clk, bus);
  cpucore cpu (clk, bus);
  •••
  always @ (bus.rdy) // Access interface signal using instance name
    if (bus.mode == 0)
      •••
endmodule : top
```

### How to Define Interface Ports
An interface can have its own ports: 
* Connected like any module port
* Used to share an external signal

```sv
interface ifa (input clk); // define clk port in interface declaration
  logic req, start, gnt, rdy;
  logic [1:0] mode;
  logic [7:0} addr;
  wire [7:0] data;
  •••
endinterface : ifa
```

```sv
module memory (ifa bus);
  •••
endmodule

module cpucore (ifa bus);
  •••
endmodule

module top;
  logic clk = 0;
  
  ifa bus(clk); // connect clk port of top module to interface port duting instantiation
  
  memory mem (bus);
  cpucore cpu (bus);
  •••
endmodule : top
```

### How to Define Interface Parameters
You can parameterize interface just like a module 

```sv
// you can define them as port of interface header
interface fastbus #(WIDTH1 = 16, WIDTH2 = 8) (input clk);
  logic [WIDTH1-1:0] data;
  logic [WIDTH2-1:0] address;
  •••
endinterface

// you can define them inside interface with parameter type
interface slowbus (input clk);
  parameter WIDTH = 16;
  logic [WIDTH-1:0] data;
  •••
endinterface

module test;
  fastbus #(8,5) bus8x5 (clk); // 8-bit data and 5-bit address
  fastbus #(8) bus8x8 (clk); // you can override parameters duting interface instantiation
  slowbus #(.WIDTH(8)) bus8 (clk) // 8 bit data
endmodule
```

### Modport
To restrict interface access within a module, there are **modport** lists with direcetions declared within the interface. The keyword **modport** indicates that the directions are declared as if inside the module.

Modports create different views of an interface 
* Specifiy a subset of interface signals accessible to a module
* Specify direction information for those signals

You can specify a modport vire for a specific module in to ways:
* In the module declaration
* In the module instatiation

```sv
interface mod_if'
  logic a, b, c, d;
  modport master (input a, b, output c, d);
  modport slave (input c, d, output a, b);
  modport subset (input b, output a);
endinterface
```

### Seletcting the Interface Modport by Qualifying the Module Port Interface Type
An interface modport can be selected unsing the module declaration port of interface type.
* busmaster decalares interface port mbus - type is mod_if and modprt is master
* busslave decalares interface port sbus - type is mod_if and modprt is slave
* testbench instantiates interface and module as before

```sv
interface mod_if'
  logic a, b, c, d;
  modport master (input a, b, output c, d);
  modport slave (input c, d, output a, b);
endinterface
```

```sv
module busmaster (mod_if.master mbus);
  •••
endmodule

module busslave (mod_if.slave sbus);
  •••
endmodule

module testbench;
  mod_if bus();
  busmaster M1 (.mbus(bus));
  busmaster S1 (.mbus(bus));
  •••
endmodule
```

### Selectiong the Interface Modport by Qualifiying the Module Port Interface Binding
An interface modpot can be selected during the port mapping of module intantiation.
* busmaster declares interface port mbus of the mod_if 
* busslave decalred interface port sbus of type mod_if
* testbench maps the mbus port of M1 to modport master of bus2
* testbench maps the sbus port of S1 to modport slave of bus2

```sv
interface mod_if'
  logic a, b, c, d;
  modport master (input a, b, output c, d);
  modport slave (input c, d, output a, b);
endinterface
```

```sv
module busmaster (mod_if.master mbus);
  •••
endmodule

module busslave (mod_if.slave sbus);
  •••
endmodule

module testbench;
  mod_if bus2();
  busmaster M1 (.mbus(bus2.master));
  busmaster S1 (.mbus(bus2.slave));
  •••
endmodule
```

### Interface Methods
A subroutine defined within and interface is called an **interface method**.
* Writing/reaing interface signals can be handled by tasts - abstract interface communication 
* However everymodule must contaion copies of the interface - maintainability issues
* Solution is to declare tasks as part of the interface - accessible to any module connected to the interface 

```sv
interface ifa (input clk);
  logic req., start, gnt, rdy;
  logic [1:0] mode;
  logic [7:0] addr;
  wire [7:0] data;
endinterface : ifa
```

```sv
module cpucore (ifa bus);

  task read (input address);
    @ (posedge clk);
    bus.addr = address;
    •••
  endtask
  •••
  read(8'hff);
  •••
endmodule
```

### Accessing Interface Methods in Module
* Same syntax and statements as in the module task definition 
* Declare once - visible to any module connected to interface 
* Call methods through interface port or instance

```sv
interface ifa (input clk);
  logic req., start, gnt, rdy;
  logic [1:0] mode;
  logic [7:0] addr;
  wire [7:0] data;
  
  task read (input byte raddar , output byte rdara);
    @ (posedge clk);
    addr = radder;
    rdata = data
    •••
  endtask
endinterface : ifa
```

```sv
module cpucore (ifa bus);
  •••
  bus.read (addr,data);
  •••
endmodule
```

### Declare a Module Port of a Generic Interface Type 
You can use the **interface** keyword rather than a specific interface type in your module declaration's list of prts. This serves as a placeholder for an interface type and you can select the interface when instantiating the module.

You can declare a module port of a generic interface type:
* This defers actual interface slecetion until module instantiation
* Must use Verilog 2001 ANSI-stye port declaration

```sv
interface ifa; 
  logic req, start, gnt, rdy;
  logic [1:0] mode;
  •••
endinterface : ifa
```

```sv
module memory (interface bus); // use a generic interface port
                               // in module defintition
  •••
endmodule

module cpucore (interface bus); // use a generic interface port
                               // in module defintition
  •••
endmodule

module top;

  ifa bus1();
  memory mem (.bus(bus1)); // Map specific interface during instatiations
  cpucore cpu (.bus(bus1));
endmodule : top
```

## Simple Verification Features
----
### Strings 
* **string** is a new datatype - dynamic arrays of bytes that grows/shrinks automatically to hold contents with initial value **""** (empty string)
* It is indexed as a normal array - with 0 as the left most character
* It can be concatenated, replicated or compared
* It can be assigned a string literal 

```sv
string message = "test";

initial begin  
  if (pass)
    message = {message, "passed"};
  else
    message = {message , "failed"};
  $display ("%s" , message);
  $display ("%c" , message); // "t"
  
  string repsrt;
  repstr = {2{"go "}}; // "go go "
  repstr[2] = "t"; // "gotgo "
```
### String Operators

<img width="1744" alt="Screen Shot 2022-07-26 at 11 01 35" src="https://user-images.githubusercontent.com/109002901/180955171-a729a704-08dd-4976-9ccd-4ad2e764a4b2.png">

### String Methods

<img width="1697" alt="Screen Shot 2022-07-26 at 11 03 03" src="https://user-images.githubusercontent.com/109002901/180955449-9b930f7d-e042-444b-a09b-be51cd53cdea.png">
<img width="1688" alt="Screen Shot 2022-07-26 at 11 04 14" src="https://user-images.githubusercontent.com/109002901/180955679-49b60c33-e349-471a-83e0-80f16f77669a.png">

### Immediate Assertions
*  A procedural **assert** statement is similar to an **if** statement and ignored by synthesis
*  When executed, assertion verifias a boolean expression - pass if expression evaluates to 1 and fail if expression evaluates to 0, Z ,X
*  reports error message on failure - you can change this behavior
*  You should label you assertions - label is used in the failure message and helps to manage assertions


```sv
module test;
  •••
  always @(negedge clk)
    A1: assert ( ~(wr_en && rd_en) );
  •••
endmodule

// Verilog equivalent 
always @(negedge clk)
    if (wr_en && rd_en)
      $display ("error");
  •••
```

### Action Blocks
Assertions can be execute statements when they pass or fail:
* Called action blocks 
* Can contain any SystemVerilog constructs
* Has verification functionality only 
* **else** block is executed on assertion failure - common to omit pass block
* Assertion label can be accessed in the action block via the **%m** format specifier - included in the failure message

```sv
module test;
  •••
  always @(negedge clk) begin
    A1: assert ( ~(wr_en && rd_en) );
      $display("%m : success");
    else begin 
      $display ("read/write fail");
      err_count++;
    end 
    
    A2 assert (valid);
      // pass block ommited
    else  
      $display ("valid inactive");  
  end 
  •••
endmodule
```

### Severity Levels 
Assertion failure can be graded with severity levels: **$info**, **$warning** , **$error** and default **$fatal** (terminates simulation)
* Severity level is reported 
* You can appent output information using syntax identical to that for $display
* This is output in addition to the standard failure

```sv
always @(negedge clk) begin
  A1: assert (valid);
    $display("%m : success");
  else begin 
    $warning("valid inactive");
```

### Immediate and Concurrent Assertions
An immediate assertion is an instantaneous boolean check - single 
. 

```sv
always begin : CHECK @ (posedge ce);
  repeat (16) begin
    @ (posedge clk or negedge ce);
      SP : assert (ce) 
      else begin 
        $error ("short ce pulse");
        disable CHECK;
      end
    end
    @ (posedge clk or negedge ce);
    LP :  assert (!ce)
    else $error ("long ce pulse");
end
```

Concurrent assertion describe behavior that spans over time - can span multiple cycles.

```sv
SPI1 : assert property (@(posedge clk) !ce ##1 ce |-> ce[*16] ##1 !ce);
```

## fork-join Enhancements: join_any , join_none 
Verilog **fork-join** completes when all spawned blocks complete - This blocks further execution until the **fork-join** completes.

```sv
fork
  a;b;c;
join
d;
```
SystemVerilog adds two join variants to control when **fork-join** completes:
* **join_any** completes the fork as soon as any of the blocks completes - other blocks left running

```sv
fork
  a;b;c;
join_any
d;
```

* **join_none** completes the fork immediately - all blocks left running 

```sv
fork
  a;b;c;
join_none
d;
```

### Proccess Control - disable fork, wait fork
* **disable fork** terminates all active descendants of the current proccess - use after **join_any** to ensure only one forked block completes. You can write **disable block_name** to terminates all active executions of block and functions or tasks that called from it, but does not terminate subproccess of the block (if there are **join-forks** in the block, they will not be terminated)

```sv
fork 
  fork 
    a; b; c;
  join_any 
  disable fork;
  d;
join
```
* **wait fork** stops further execution until all forked blocks complete - affects all forks in currenct scope

```sv
fork
  a; b; c; 
join_any

fork 
  d;
join_none
e;
wait fork;
f;
```

## Clocking Blocks
----
The **clocking block** construct identifies clock signals and captures the timing and synchornization requirements of the blocks being modeled.
<img width="1520" alt="Screen Shot 2022-07-26 at 13 56 27" src="https://user-images.githubusercontent.com/109002901/180990205-472c5f2a-3152-4cfe-9c8f-cb2c2e7a2b0c.png">

* Driving or sampling DUT port on active clock edges can lead to race conditions
* Clocking blocks are a verification construct to help avoid this
  * Driving testbench outputs via a clocking blocks adds a delay (skew) to transistors
  * A clocking block can also sample testbench inputs with a set delay
* Clocking blocks separate signal timing from signal function and allows stimulus to be written in terms of cycles and transactions only

### Clocking Block Declaration
* A clocking block can be declared in a module or interface 
* Clocking block declaration defines the clock event , signal direction (does not declare signals , direction is from testbench perspective) and input and outpout delay (skew) - explicitly or by default
* Outputs are driven skew units after the clock eveng
* Inputs are samples skew units before the clocking event

```sv
clocking cb @(posedge clk);
  default input #1ns
          output #3ns; // default skew for inputs and outputs
  
  input dout; // clocking block input
  output data; // clocking block outputs
  output #5sn enab; // output with explicit skew

endclocking
```

### Clocking Block Output Drive
For timing drive output via the clocking block - (cb.enab <=1;) - must use a nonblocking assignment
This schedulas assignment with skew:
* Wait for the clocking block event - use the current time if assigned at the same time as the event occurs, wait for specified skew delay and drive output
* Can synchronize to clocking block event  - @(cb);

```sv
clocking cb @(posedge clk);
  default input #1ns
          output #3ns; // default skew for inputs and outputs
  
  input dout; 
  output data; 
  output #5sn enab; 

initial begin
  @(cb); // sync to clocking event
    cb.data <= 1'b1;
    cb.enab <= 1'b1; // clocking block drives
  •••
```

### Clocking Block Input Samples
* Clocking block automatically sampels input: 
  * Input samples as specified skew before clocking event
  * Input unpdated obseved region

* For timing read input via clocking block - dreg <= db.out;
* This reads input from the last sample point - input sample from clocking value may be different from current value 
* Can synchronize to a change in input sample - @(cb.dout)

```sv
bit [7:0] dreg, dout;
clocking cb @(posedge clk);
  default input #1ns
          output #3ns; 
  
  input dout; 
  output data; 
  output #5sn enab; 
endclocking

initial begin
  @(cb); 
  // read last sample
  dreg <= cb.dout;  
  •••
```

### Input and Output Skews
You can specify skew:
* With a default value - separate default for inputs and for the outputs
* Explicitly in the signal identifier - override the default 

Skew can be: 
* A constant expression, parameter or number - must be positive and uses timescale of current scope
* An edge (posedge, negedge, edge)
* A time literal (including #1step - remember 1step means one simulation precision unit before the clocking evet 

```sv
clocking cb @(posedge clk);
  default input #1step
          output #3ns; // default skew for inputs and outputs
  
  input dout; 
  output data; 
  output #5ns enab; 
endclocking
```

if we have: 
```sv
clocking cb @(posedge clk);
  
  output #3ns one; 
  output negedge two; 
endclocking 

initial begin 
  @(cb);
  @(negedge clk);
  #1ns
  cb.one <= 1;
  cb.two <= 1;
```
Its mean that **synchronization point** is right after positive edge -> negative edge -> 1ns delay , than wait until the next clk positive edge, then 1ns to update value. For signal two , it wait to the next negative edge to upsate value.

### Multiple and Default Clocking Blocks 
* You can define multiple clocking blocks in a scope - for multiple clocks , different signals or different timming 
* One block in a scope can be defined as a default clocking block - add **default** to the beginning of the clocking block declaration, or use a default statement separate from the declaration
* Default clocking blocks allow cycle delays

```sv
clocking cb1 @(posedge clk1);
  default input #1step
          output #3ns; 
  input dout1; 
  output data1; 
endclocking

clocking cb2 @(posedge clk2);
  input #2 dout2; 
  output #2 data2; 
endclocking


default clocking cb3 @(posedge clk2);
  input #2 dout2; 
  output #2 data2; 
endclocking

// equivalent to 
clocking cb3 @(posedge clk2);
  input #2 dout2; 
  output #2 data2; 
endclocking

default clocking cb3
```

### Cycle Delay
* You can insert cycle delays for the default clocking block
  * ##N - positive number or identifier 
  * ##(expr) - positive integer expression
* Procedural delay always refers to the default clocking block - ##1 cb.data <= 1;
* Synchronous drive delay always refers to target variable's clocking - cb.data <= ##2 var; , this is not intra-aassignment delay and illeagal for clocking block signals

```sv
default clocking cb1 @(posedge clk);
  input #2 dout;
  output #3 reset, data;
endblocking

clocking cb2 @(negedge clk);
  output #3 enab, rdata;
endclocking 

initial begin 
  repeat (2) begin
    @(cb1);
    //wait 2 cb1 cycles 
    ##2; 
    // drive data after 1 cb1 cycle 
    ##1 cb1.data <= 2'b01;
  end
  // drive rdata with current dreg value after 3 cb2 cycles
  cb2.rdata <= dreg;
end   
```

## Handling Heirarchial Expressions in Clocking Blocks 
Clocking blocks can drive or sample via a hierarchical expression:
* Out-Of-Module References (OOMRs)
* Slices or concatenations of signals in current or other scopes

**Problem** - A clocking block signal cannot be a hierarchical expression
**Solution** - You must associate a clocking block signal with the hierarchical expression

```sv
clocking cb @(posedge clk);
  default output #3;
  // output top.dut.daya; - error - direct hierarchical expression is illegal
  // data associated with hierarchical expression
  output data = top.dut.data // correct - hierarchical expression associated with signal 
endclocking 
```

## Random Stimulus 
----
#### CPU Coverage Test Care
Declare opcode and register values as enumerated types. Assign values for driving into DUT. 
**Problem** - How to test, combinations of opcode, register and data ? 
* **Solution 1** - Using looped CPU instructions stimulus 
* **Solution 2** - Applying SV randomization and constraints 

This simplified 8-bit CPU has: 7 immediate instructions: ADDI, SUBI, ANDI, XORI, JMP, JMPC, CALL and 4 registers: REG0, REG1, REG2, REG3
in the following double byte format
<img width="1300" alt="Screen Shot 2022-07-26 at 17 22 10" src="https://user-images.githubusercontent.com/109002901/181029720-f14f406d-ea41-45b0-b3e6-7cf631b011b9.png">

```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;
typedef enum bit[1:0] {REG0, REG1, REG2, REG3} regs_t;

op_t opc;
regs_t regs;
logic[7:0] data;

initial begin 
  // generate stimulus
  opc = ADD1;
  regs = REG0;
  data = 5;
  // drive into DUT
  •••
```

### Solution 1 - Looped CPU Intruction Stimulus
Create a matrix using nested loops to index through combinations.
* Structued stimulus does not thoroughlty test CPU functonality
* It misses the most transitions between value combinations - what if AAD1 followed by JMP causes an out-of-range error

### Solution 2 - Employ SV Randomaization and ConstraintsThe human brain loves the patterns, however, bugs don't come in patterns. SystemVerilog provides a new set of structures for generating and constraining random variable.
* Random generation allows minimal code to generate mana data sets
* Constraints restricts the data set to meaningful data
* You can apply randomization with constraints to:
  * Local variables - "scope" randomization (this module)
  * Class properties - class-based randomization


### Pseudo Random Number (PRN) Generators
PRN Generators generate random sequences algorithmically which are controlled by initial values called as seeds
Pure random number sequences are useless for verification
* Sequence must be repeatable
* Allows debug fix debug cycle

Verificiation uses pseudo-random sequences - controlled by seed(s): 
* Using the same seed generates the same sequence
* Using a different seed generates a different sequence

### randomize() - Randomizing Scope Variables
* The **radomize()** function generates random values on variables - it return 1 on success, variables can be randomized and constraints can be met. Otherwise return 0
* You can randomize integral scalar and array variables
* **randomize()** is defined in a built-in package called std - automatically imported and singular, integral and variable restrictions

```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;
typedef enum bit[1:0] {REG0, REG1, REG2, REG3} regs_t;

op_t opc;
regs_t regs;
logic[7:0] data;

int ok;

initial begin
  repeat (7168) begin 
    // function randomizes variable pass as arguments
    ok = randomize(opc, regs, data); 
    @(posedge clk);
  end
end
```

### Random Stability 
Random stability is to localize the RGN (Random Number Generator) to threads and object. Because this makes the sequence of random values return by a thread or object independent of RNG in other threads or objects.
* Each design element instance RNG has the same initial seed (default 1) - can be change on command line **-svseed = x**
* Every instance and process has its own RNG
* Each process RNG is seeded with the next value from parent RNG
* Adding new proccesses after exitsting code maintains seed order and stability 

### Setting the Random Seed 
* You can manually seed a proccess RNG - can help with thread stability but very rare to use it 
* Use **srandom()** methods of the built-in **proccess** class (a proccess is a class) - **proccess:: self.srandom(seed)**
* Proccess object instances are automatically created

```sv
module randtest;
•••

// proccess 1
always @ (posedge clk); 
  begin : P1
    proccess::self.srandom(10); // manually seed P1 RNG
    ok = randomize(data1);
    •••
  end
  

// proccess 2
always @ (sel, d1, d2); 
  begin : P2
    ok = randomize(data1);
    proccess::self.srandom(20); // RNG of P2 reseeded during execution
    •••
  end
```

### Constraint Block 
The **constraint_block** is a list of expression statements that restrict the range of a variable or define relations between variables.
Use the **with** clause to attach a constraint block to the randomize method.


```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;
typedef enum bit[1:0] {REG0, REG1, REG2, REG3} regs_t;

op_t opc;
regs_t regs;
logic[7:0] data;

int ok;

initial begin
  ok = randomize(data) with { data >= 32 ; data <= 126; }; // relational - data between 32 and 126
  ok = randomize(opc) with { opc inside {[ADDI;ANDI] , JMP , JMPC}; }; // list - opc in range 
                                                                       // ADDI to ANDI or JMP or JMPC
  // distribution: REG0-REG1 twice as likely as REG2-REG3                                                                     
  ok = randomize(regs) with { regs dist { [REG0:REG1] := 2 , [REG2:REG3] := 1 }; 
end
```

### Conditional Constraints 
Conditional constraints allow you to select between different sets of constraints, depending on the value of another variable.
There are two ways of defining condirional constraintds: 
* Implication, using the **->** operator
* **if-else**, using the **if-else** construct

```sv
typedef enum bit[1:0] {SMALL, MEDIUM, LARGE, XL} mode_t;

mode_t mode;

int ok;

initial begin
  // if mode is SMALL, data constraint is <100
  ok = randomize(data) with {mode == SMALL -> data < 100;
  // if mode is LARGE, data constraint is > 100
                             mode == LARGE -> data > 100;};
  // if mode is neither SMALL or LARGE data is unconstrained
  
  // same constraints with if-else
   ok = randomize(data) with {if (mode == SMALL) data < 100;
  // if mode is LARGE, data constraint is > 100
                             else if (mode == LARGE) data > 100;};
```

### Random Weighted Case - randcase 
You can use **randcase** to randomly select statement(s) for execution.
* You can provide different probabilities (weights) for each branch
* Weights may be constant or variable expressions - non negative integral expressions only

```sv
repeat (50) begin 
  randcase
    20 : gen_atm();
    30 : gen_ethernet();
    10 : gen_ipv4();
    5 : gen_crc_errror(); // probability = 5/65 ~ 8%
  endcase
```

## Basic Classes
----
**Object Oriented Programming** is a programming paradigm that focuses on objects (as apposed to proccess).
Data oriented rather than procedure-oriented
* Focuses on object instead of actions
* Defines classes to characterize the objects 
* Classes decalre and own:
  * properties (attributes) - class member data mostly is private - hidden from "outside" code
  * behaviors (methods) - class member functions often are public - callable by outside code 

### Data Encapsulation in OOP
Data encapsulation means that each object hides its data from external access
* An object typically prohibits direct external access to its data
* Objects interact by "sending messages"
* A message can request the reciving object to change a data value or replay with a data value

### Class
A class is a type that includes data and subroutines (functions and tasks) that operate on those data.
* A class is a user-defined data type - declared in a module package interface or other design element
* Classes define data and subroutines (tasks/functions) that operate on the data
* Class objects can be dynamically created and deleted during simulation
* Used in OOP for testbenches and simulations models

```sv
module top;
  // class declaration in a module
  class myclass;
    int number
  endclass

package myclasses;
  // class declaration in a package
  class frame;
    logic [7:0] payload;
    bit parity;
  endclass
```

### Class Object
A class defines a data type. An object is an instance of that class. An object is used by first declating a variable of that class type (that holds an object handle) and then creating an object of that class (using the **new** function) and assigning it to the variable.

The class type declartion declares class members:
* Data itmes called properties or fields
* Task or function which operate on data items called methods

```sv
class myclass;
  int number; // class property
  
  task set (input int i); // class method
    number = i;
  endtask
  
  function int get(); // class method
    return number;
  endfunction  
endclass

myclass c; // declaring a variable of class my_class, c is called "handle"
c = new; // creating an instance of myclass with new.
         // now c is inferred as instance or object
```

### Variables of the Class Type
* A variable of a class type is a called a handle - the uninitialized value is null
* A class instance must be created for handle - using the **new** function. Assigns a pointer to an area of memory holding the class instance. A handle can be declared and created at the same tine 
* Object presists until it either is no longer used or assigned **null**
* There is automatic garbage collection


```sv
module top;
  class myclass;
    int number;
  endclass
  
  myclass c; // handle
  
  initial begin 
    c = new; // instance
    •••
  end
  
```

### Accessing Object Members by Using . Operator
* Class members are accessed using ** . **
* Class properties can be accessed directly or via class methods

```sv
class myclass;
  int number; // class property
  
  task set (input int i); // class method
    number = i;
  endtask
  
  function int get(); // class method
    return number;
  endfunction  
endclass

myclass obj1 = new; 
initial begin 
  obj1.number = 4; // direct access 
  
myclass obj2 = new;
initial begin 
  obj2.set(3); // method access
```

### External Method Declaration
Purely for convenience and readablitiy, you can declare the implementation of a class method outside of the class declaration using the **extern** keyword.

* Define the methods outside the class declaration
* Define the method prototype in the class prefixed by the keyword **extern** - first line in method, identifying type, name and arguments
* Implement the method outside the class declaration, but in same scope - link to prototype using the scope resolution operator **::** , class_name::method_name


```sv
class myclass;
  int number; // class property
  
  task set (input int i); // class method
    number = i;
  endtask
  
  extern function int get();
endclass

function int myclass()::get();
  return number;
endfunction
```

### Constructor Method
The **new** function which is used to create a class instance and to initialize and instance is called the class constructor.
Method **new** is a special class method called a constructor: 
* Defined by default for al classrs
* You can explicitly define it
* Just like any other class function except new has no return type

```sv
// default constructor
class defcon;
  int number;
endclass

// explicit constructor
class expcon;
  int number;
  
  function new();
    number = 5;
  endfunction
end class

expcon c1 = new; // c1.number = 5

// explicit contructor with arguments
class argcon;
  int number;
  
  function new (input int a)
    number = a;
  endfunction
endclass

argcon c2 = new(3); // c2.number = 3
```

### Current Object Handle this
* Keyword **this** is a handle to the current class instance - you can use it to reference class identifiers re-declared within a local scope. It is meaningful for nonstatic members
* Here , this.addr is used to access the class property addr - otherwise it is hidden by the function argument addr

```sv
class frame;
  logic [4:0] addr;
  logic [7:0] payload;
  
  function new (input int addter, dat);
    this.addr = addr;
    payload = dat;
  endfunction
endclass
```

### Static Class Properties
The static class properties are class properties which are created using the keyword **static**. A static property is shared by all instance of a class.

```sv
class frame;
  static int frmcount;
  int tag;
  logic [4:0] addr;
  lgoic [7:0] payload;
  logic parity;
  
  function new (input int add , dat);
    addr = add;
    payload = dat;
    frmcount++;
    tag = frmcount;
  endfucntion
endclass
```

### Static Class Methods
When methods are declared as static, they are subject to all the class scoping and access rules but behave like a regular subroutine that can be called outside the class, even with no class instantiation. 
Only static properties or other static methods can be access. Can be called even if no class instantiations exits
* From class name using resolution operator **::**
* From any class handle

```sv
class frame;
  static int frmcount;
  int tag;
  logic [4:0] addr;
  lgoic [7:0] payload;
  logic parity;
  
  function new (input int add , dat);
    addr = add;
    payload = dat;
    frmcount++;
    tag = frmcount;
  endfucntion
  
  static function int getcount();
  return (frmcount);
  endfunction 
endclass
```

```sv
frame f1,f2; // handles
int frames;
initial begin
  frames = frame::getcount(); // 0
  frames = f1.getcount(); // 0 
  
  f1 = new (1,2);
  f2 = new (3,4);
  frames = f2.getcount(); // 2 
end
```

### Class Aggregation
Aggregation refers to a class which collects a number of other classes into one object.
* A class property can be instnce of another class
  * Creates an aggregare or composite class
  * Relationship "has a"
  * Similar to module instantiation
* Constructors of class properties must be explicitly called
* Instance handles mush be chained to reach into hierarchy
* Basically, use other class instance as a property of a class

### Class Inheritance 
Class inheritance is extending a class by defining a derived clas that inherits members (does not inherit constructors) of the base class and may override inherited members and/or add a new members.

<ins>**Why inheritance?**</ins>

* Reusability makes the user more productive 
* Encourages use of proven software

<ins>**Terminology**</ins>

* Original class is called superclass (OO) or base clase (SV) 
* New class is called the subclass (OO/SV) or derived class
* SV provides only single inheritance - that is each subclass is derived from a single base class


```sv
class Base
  ••• // base functions
  ••• // base data
endclass
```

```sv
class Derived extends Base // include the base function and data
  ••• // more functions
  ••• // more data
endclass
```
If superclass constructor takes arguments, the derived class constructor has to take these arguments

```sv
class frame;
  static int frmcount;
  int tag;
  logic [4:0] addr;
  lgoic [7:0] payload;
  logic parity;
  
  function new (input int add , dat);
    addr = add;
    payload = dat;
  endfucntion
endclass

class badframe extends frame;
  function new();
    super.new(); // automatically inserted
    frm.count++;
    tag = frmcount;
  endfunction
endclass

class goodframe extends frame;
  function new(input int add,dat);
    super.new(add,dat); // automatically inserted
    frm.count++;
    tag = frmcount;
  endfunction
endclass
```
<img width="1849" alt="Screen Shot 2022-07-27 at 11 40 05" src="https://user-images.githubusercontent.com/109002901/181202518-3101d3b8-47e7-4d08-8ab0-843eef94e170.png">

### Multi-Layer Inheriance 
You can inheriance to multiple generations. Each new level inherits the members of the pervious levels and can override any of these members and can add new members. Notice that **super.super.new()** is not allowed , only **super.new()**

### Data Hiding and Encapsulation 
To restrict the access to class properties and methods from outside the class by hiding their names, SystemVerilog provides **local** and **protected** identifiers.

By default, class members are visible externally and in all sub-classes. Two keywords hide members: 
* **local** - Only visible inside the class
* **protected** - Visible inside the class and any sub-classes (can be inherited)

### What Are Parameterized Classes 
The SystemVerilog parameter mechanism is used to parameterize a class. It allows user to define a generic class whose objects can be instantiated to have different array sizes or data types.

* Classes can have parameters just like modules and interfaces
* Parameters can be types as well as values - a type parameter creates a "class template"
* Parameter values can ve overriden for individual instances - methods using type parameters must work for all expected type overrides
* Each new parameter value effectively creates a new class declaration or class specialization - each class specialization has a separate set of static properties

## Polymorphism and Virtuality 
----
### Polymorphism
Polymorphism allows the use of a variable of the superclass type to hold subclass objects and to reference the methods of those subclasses directly from the superclass variable. 
* An inherited class creates a new type - you need to easily change between sub-class types
* A class handle of a given type can be assigned any class extension instance - polymorphism, this introduces the conceopt of handle type, class type is used in delcaration
* It aims to look at the contents of the handle - class instance held in the handle.

### Copy a Sub-Class Instance to a Parent Handle 
A sub-class instance can always be directly copied to a parent class handle. However, by default, only parent class members are accessible from the handle - even thought it conttains a sub-class instance.

```sv
class frame;
  function void iam();
    $display("frame");
  endfunction
endclass

class tagframe extends frame;
  int tag;
  
  function void iam();
    $display ("tagframe");
  endfunction
endclass
```

```sv
frame f1;
tagfreame t1 = new(); // create an instance of sub-class

initial begin 
  f1 = t1; // copy subclass instance to superclass handle
  f1.iam(); // frame , only parent method visable - error
  f1.tag = 5; sub-class property not visible 
```

### Copy a Parent Instance to a Sub-Class Handle
It is never legal to directly assign a parent handle to a handle of one of its sub-classes. It is only legal to assign a parent handle to a sub-class handle, if the parent handle contains an instance of the given sub-class.
* A parent-class instance cannot be copied to a sub-class handle - unless the parent class handle conatins a subclass instance 
* Requires use of **$cast** - checks that the parent handle conatins a sub-class instance 

```sv
class frame;
  function void iam();
    $display("frame");
  endfunction
endclass

class tagframe extends frame;
  int tag;
  
  function void iam();
    $display ("tagframe");
  endfunction
endclass
```

```sv
frame f1;
tagfreame t1 = new(); 
tagframe t2;

initial begin 
  f1 = t1; // copy subclass instance to superclass handle
  f1.iam(); // frame , only parent method visable 
  $cast(t2,f1); // copy from f1 to t2 m checking with $cast
  t2.iam(); // sub-class (tagframe) method now visiable
```

### Using  $cast
Casting lets sub-class instances use resources defined for parent classes.
**$cast** is actually a subroutine - defined as both function and task 
* Syntax - **$cast(destination , source)** , if the source does not contain a matching instance for the destination - task gives a runtime error , function returns 0

### Advantages of Polymorphism 
Polymorphism Usage: 
* A base claass is used for handle array declaration
* Diffenect sub-class instances are then assigned to array elements 
* We can dynamically selet with sub-class instance to load into array.

Advantages:
* The type of frame is decided at the start of stimulus code
* All subsequent references can be maes to the base array variable
* More sub-classes can be easily added to the design

```sv
frame framearr[7:0];
tagframe tf;
errframe ef;

initial begin 
  foreach (framearr [i])
    randcase 
      2: begin 
         tf = new (0,0);
         framearr[i] = tf;
         end
      1: begin 
      ef = new (0,0);
      framearr[i] = ef; 
      end
   endcase
```

### Class Member Access in Polymorphism 
**Problem** - By default, resolution of class member access is always according to the type of handle, not its contents
**Solution** - Using virtual methods

* Class members are resolved by searching from the class handle type 
  * Through the inheritance hierarchy
  * Even if the handle contains a subclass instance 
* This blocks access to sub-class members for a sub-class instance in a parent class handle

```sv
class base; 
  function void iam();
    $display("Base");
  endfunction 
endclass

class parent extends base; 
  function void iam();
    $display("Parent");
  endfunction 
endclass

class child extends base; 
  function void iam();
    $display("Child");
  endfunction 
endclass
```

```sv
base b1; 
parent p1 = new();
child c1 = new();

initial begin
  b1 = p1;
  b1.iam(); // Base
  
  p1 = c1;
  p1.iam() // Parent
```

### Using Virtual Methods 
* Virtual methods direct methods resolution to the contents of a handle
* They allow access to methods of a sub-class instance when held in a parent class handle 
* When a method is declared as **virtual** it is automatically **virtual** in every sub-class

```sv
class base; 
  virtual function void iam();
    $display("Base");
  endfunction 
endclass

class parent extends base; 
  virtual function void iam(); // virtual keyword is optional
    $display("Parent");
  endfunction 
endclass

class child extends base; 
  virtual function void iam(); // virtual keyword is optional
    $display("Child");
  endfunction 
endclass
```

```sv
base b1; 
parent p1 = new();
child c1 = new();

initial begin
  b1 = p1;
  b1.iam(); // Parent 
  
  p1 = c1;
  p1.iam() // Child
```

### Resolve Class Method Access 
With polymorphism, when a methods is accessed off a class handle, how do you identify which class method is used ? 
1. Examine the class declaration of the handle type 
2. If the method is not virtual, then the call is directed to the handle class 
3. If the methods is virtual, then examine the contents of the handle 
4. If the handle contains a sub-class instance, the call is directed to the subclass
5. If the handle does not contain a sub-class instance, the call is directed back to the handle class

```sv
class base; 
  function void nvirt();
    •••
  endfunction

  virtual function void virt();
    •••
  endfunction
endclass

class parent extends base; 
  virtual functionvoid virt();
    •••
  endfunction
endclass
```

```sv
base b1; 
parent p1 = new();

initial begin 
  b1 = p1; 
  b1.nvirt(); // base 
  b1.virt() // parent 
```

### Abstract Classes and Pure Virtual Methods
A virtual class exists only to be inherited - it cannot be instantiated
* Also known as an "abstract" class

A virtual class (only) may have pure virtual methods: 
* Prototype only - no implementation
* Sub-class must provide implemetation

```sv
virtual class base; 
  pure virtual function void iam();
    $display("Base");
  endfunction 
endclass

class parent extends base; 
  virtual function void iam(); // virtual keyword is optional
    $display("Parent");
  endfunction 
endclass

class child extends base; 
  virtual function void iam(); // virtual keyword is optional
    $display("Child");
  endfunction 
endclass
```

```sv
base b1 = new(); // instance illeagal
parent p1 = new();

initial begin
  b1 = p1;
  b1.iam(); // Parent 
```

## Class-Based Random Stimulus
---- 
### Random Class Propertires
Random class properties are integral dlass properties that defined/declared as radom using rand or randc
* **rand** - random with uniform distribution
* **randc** - random cyclic randomly iterates through all values without repetition : 
  * When an iteration is complete, a new random iteration automatically starts 
  * Implementation may limit size

```sv
class randclass;
  rand bit [1:0] p1;
  // p1 example - 01 11 00 01 11 01 10 11 - equal probability, close repetition
  randc bit [1:0] p2;
  // p2 example - 01 11 10 00 01 00 11 10 - each repetition cycles through all value
  // without repetition
endclass
```

### Randomizing Class Objects : randomize()
Randomize the properties by calling the **randomize()** function.
* Every class has a built-in randomize virtual method
* You cannot re-declare this method

```sv
class randclass;
  rand bit [1:0] p1; // random variable
  randc bit [1:0] p2; // random cyclic variable
endclass

randclass myrand = new();
int ok;
initial begin 
  // randomizes all random variables in a class instance 
  ok - myrand.randomize(); 
  if (!myrand.randomize())
    $display ("myrand randomize failure");
end
```

### pre_randomize() and post_randomize()
**randomize()** automatically calls two "callback" functions:
* **pre_randomize()** - before radnomization
* **post_randomize()** - after successful randomization

if defined, these methods are automatically called on randomization

```sv
function void pre_randomize();
 •••
endfunction

function void post_randomize();
 •••
endfunction
```

```sv
class randclass;
  rand bit[1:0] p1;
  randc bit [1:0] p2;
  bit [1:0] parity;
  
  function void post_radomize(); // define post_randomize
    parity = p1 ^ p2;
  endfunction
endclass

randclass myrand = new();
int ok;

initial begin
  ok = myrand.randomize(); // randomize automatically calls post_randomize
```

### Controlling Randomization rand_mode()
Every random property has an enable switch called **rand_mode**
* Enabled by default (1)
* If disables (0) the property will not be randomized

Mode can be wrriten with task **rand_mode** - **task rand_mode(bit on_off)**
* Called off a random property, the task changes the mode of that property
* Called off an instance, the task changes the mode for all random properties of the instance 

Mode can be read with **function int rand_mode()**
Only random properties have rand_mode - calling method off a non-random property is a compile error

### Constraint Blocks
* Constraint can be embedded in classes using constraint blocks
* A block can contain any any number of any form of constraints - relational, list or distribution or conditional 

```sv
class randclass;
  rand bit [1:0] p1;
  rand bit [1:0] p2;
  
  constraint c1 { p1 != 2'b00;} // constraints p1 to the values 01 10 11 
  constraint c2 { p2 >= 64 , p2 <= 192; } // ; after each constraint expression
                                          // but not after constraint block
endclass

randclass myrand = new();

int ok;
initial begin
  // randomize p1 using constraint block c1 and p2 using constraint block c2
  ok = myrand.randomize();
  •••
end
```

### Constraint Inheritance 
Constraint are class members and are inherited just like other members.

### Constraint Expressions - Set Membership
The **inside** operator is particularly useful in constraint expressions. The operator can also be negated to generate a value outsine the set.

```sv
class randclass;
  rand bit [7:0] p1;
  constraint c1 {p1 inside {3,7,[11:20]};}
endclass

randclass myrand new();

int ok;
initial begin
  ok = myrand.randomize();
  •••
end
```

```sv
class not_inside;
  rand bit [7:0] p2;
  constraint c2 { !{p2 inside {1,3,5,7};}
endclass
```

### Constraint Expressions - Weighted Distributions 
* You can change distribution by defining weights for value using **dist** (default weight is 1). 
* There are two ways to assign weight: 
  * ** := ** assigns weight to the item or every value in a range
  
```sv
constraint c1 { p1 dist { [101:200] := 200 ); // 101 to 200 each get a weight of 200
```
  * **:/** assigns weight to the item or to a range as a whole 
  
```sv
constraint c1 { p1 dist {[26:30]:/ 1 ); // 26-30 each has a weight of 1/5
```  

### Constraint Expressions - Iterative Constraints
You can use a loop to apply separate constrainta to each array element 

```sv
class randclass;
  rand logic [3:0] arr [7:0]; 
  constraint c1 {foreach (arr[i] (i <= 4) -> arr[i] <= 1;} // iterative constraint
  constraint c2 {foreach (arr[i] (i >= 4) -> arr[i] >= 1;} // iterative constraint
```  

### Controlling Costraint with constraint_mode()
Every constraint block has an enable switch called **constraint_mode** - enabled by default (1) , if disabled (0), the constraint block will not be used.

* Mode can be written with task **constraint_mode**
```sv
task constraint_mode (bit on_off);
```

* Mode can be read with function **constraint_mode**
```sv
function int constraint_mode ();
```
Only constraint block have **constraint_mode**

### Randomization Procedure and Its Effects 
**Problem** - Randomization procedure can lead to unexpected results. For example , here you expect mode to be one haft the time and this is not the case.

**Solution** - Order of randomization can be defined:
* Use solve...before (apllies to rand variables only)
* Or use **randc** properties 

Randomization proceeds as follows:
1. All **randc** properties randomized simultaneously
2. Then all **rand** properties randomized simultaneously
3. Then constrains are checked
4. Cycle iterates until a solution is found ot the random space is exhausted 
5 

```sv
class randclass;
  rand bit[2:0] vect;
  rand bit mode;
  constraint c1 {mode -> vect ==0;} // expectation: mode to be 0 one half the time
                                    // mode is 0 one ninth of the time
endclass
// applying solve...before
class randclass;
  rand bit[2:0] vect;
  rand bit mode;
  constraint c1 {mode -> vect ==0;
                 solve mode before vect;} 
endclass
```
Unordered solution:
* **vect** and **mode** randomized simultaneously 
* Constraint applied
* All solutions equaliy likely
* Probability **mode** = 1 is 1/9

Ordered solution;
* **mode** randomized first 
* **vect** randomized based on constraints
* Probability **mode** = 1 id 1/9

### Randomization Failure 
* Warning on randomization failure
  * Identifies confliction constraints 
  * Identifies variables used in constraints
* In batch mode, simulation continues
  * GUI launches Constraints Debugger Window 

## Interfaces in Verification 
----
### Class-Based Testbench 
It is a way of creating testbench that are more efficient, easy to write and enables the user to verify design with high-abstruction level modeling. 

Why use class-based testbenches ?
* Easier to maintain, reuse, etc.
* Inheritance allows instance-specific customization of data and functionality - without affecting original code
* Separation of stimulus and enviroment - Allows test writers to drive enviroment with minimal knowledge of protocols, heirarachy, etc.

<img width="544" alt="Screen Shot 2022-07-27 at 16 58 40" src="https://user-images.githubusercontent.com/109002901/181265886-3ad9fc0a-4609-4d7c-9adb-adaffa250d45.png">

### Separation of Stimulus and Enviroment
Testbench interaction with a bus is in two layers:
* Sequence - defines progression of high-level stimulus (transactions)
* Verification Component (VC) - translate bewteen transactions and signal transitions

### Class Connections to DUT Ports 
**Problem** - class-based VCs drive DUT signals via interfaces. However, VCs should not be directly connected to an inteerface instance.
* Breaks reusability - if mycv is hardwired to bus1, cannot drive bus2
* Interface instances are static - myvc has to be declared local to the bus1 instance 

**Solution** - what we need is an inerface variable:
* Used ing class to access interface signals (via virtual interface)
* Assigned to a specific interface instance when class is instantiated

```sv
interface myif (input clk);
  logic [7:0] data;
  •••
endinterface 
```

```sv
module test;
  logic clk;
  myif bus1 (clk), bus2 (clk);
  comp DUT1 (bus1); // interface instance
  comp DUT2 (bus2); // interface instance
  
  class myvc;
    
    task write_data (input int datin);
      @ (negedge bus1.clk);
        bus1.data = datin; // hardwired to bus1
        •••
     end task
   endclass : myvc // class must be declared local to bus1
   
   myvc c1;
   
   initial begin
     c1 = new();
     c1.write_data(5); // any instance of myvc can only drive to bus 1
                       // breaks usability
   end
   
endmodule
```

### Virtual Interface
The solution is to use virtual interface. 
* An interface variable that can be connected to an interface instance
* Can be referenced inside classes 
* Allows access to interface signals using variable name as prefix
* Needs to be assigned to an actual interface

A virtual interface:
* Can be declared as a class property 
* Defualt value is **null** - must be assigned to an interface instance
* **interface** keyword in the declaration is optional - include for readability 

```sv
interface myif (input clk);
  logic [7:0] data;
  •••
endinterface 
```

```sv
class myvc;
  virtual interface myif vif; // property
  •••
  task write_data (input int datin);
      @ (posedge vif.clk);
        vif.data = datin; // accessed in method
        •••
     end task
```

### Virtual Interface Limitation 
**Problem** - nets and variabelscan be read via a virtual interface. But a virtual interfaces cannot write to nets: 
* As procedural assignment to a net (**wire**) is illegal
* Bidirectional connections must be declared as **wire**

**Solution 1** - Dual Representatiton 
**Solution 2** - Utilize a clocking block 

**Solution 1** is to define two representations for and instance bidirectional connection 
* **wire** (hdata_w) for mapping to DUT **inout** port and for reading via virtual interfaces
* **logic** (data) for writing from virtual interface
* An **assign** statement is needed to drive **logic** writes onto **wire**

<img width="657" alt="Screen Shot 2022-07-27 at 17 43 23" src="https://user-images.githubusercontent.com/109002901/181276805-f01d924a-5a26-4605-b1a2-314b5fe9c037.png">
```sv
interface hbus_if (input clk);
  wire [7:0] hdata_w;
  logic [7:0] hdata;
  assign hdata_w = hdata;
endinterface
```

```sv
virtual interface hbus_if vif;

initial begin 
  @ (posedge vif.clk);
  // vif.hdata_w <= 8'hff; // procedural assignment to wire illegal
    vif.hdata <= 8'hff; // logic used as intermediary
    dreg <= vif.hdata_w; // read from wire - ok 
```
### Utilizing a Clocking block
* Adding a clocking block to an interface allows better timing control
* You can access signals via the clocking block with a virtual interface - procedural assignment to net via a clocking block is allowed
* Clocking block defines signal direction from testbench perspective
* You can reuse this direction information in the interface modport
  * Use the **clocking** keyword in modport 
  * It can be mixed with the other direction or **import** information

```sv
interface hbus_if (input clk);
  wire [7:0] hdata_w;
  
  clocking cb1 @ (posedge clk);
    default input #1 output #3;
    input hdata_w;
    output hdata_2;
  endclocking 
  
  modport tb (clocking cb1, •••);
  
endinterface : hbus_if
```

```sv
virtual interface hbus_if vif;

initial begin 
  @ (posedge vif.clk);
    vif.cb1.hdata_w <= 8'hff; // procedural assignment to cb wire
    dreg <= vif.cb1.hdata_w;
  •••
end
```

## Covergroup Coverage
---- 
### Structural and Functional Coverage
Structural metrics (code coverage) are tool-instrumented monitors to check:
* Execution of design blocks, lines or statements
* Assignments of values to variables

Code coverage is: 
* Instrumented by tool - tool blindly focuses on individual items
* Less difficult to set up 
* More difficult to analyze 

Functional metrics are user-defined scenarios to check:
* Assignments of sequences of values to variables
* Stepping of the design through control states (transactions)

Functional coverage is a measure of what functionalities/features of the design have been exercised by the tests. This can be useful in constrained random verification (CRV) to know what features have been covered by a set of tests in a regression.

Functional coverage is:
* Insturmented by user - user specifies scenarions, corner cases , protocols...
* More difficult to set up          

**Functional coverage complements but does not replace code coverage**

### Data-Oriented and Control-Oriented Functional Coverage
Data-oriented functional coverage defines a coverage model to capture value changes/signals transitions.
* Covergroups for data oriented functional coverage 

```sv
covergroup cg @ (posedge clk);
  cpa: coverpoint addr
  { bins low = { [0;'h0F] , 19};
    bins mid = { 16 , 17 ,18 {;
    bins higs = { ['h14 : 'hFF] }; }
  addrxvalid : cross cpa, valid;
endgroup
ch ch1 = new();
```

Control-oriented functional coverage is sequence of control states. 
* SystemVerilog Asserions for control-oriented functional coverage

```sv
property reg_gnt (cyc);
  @ (posedge clk)
    reg ##[cyc] gnt;
endproperty

Coverqgnt_3 : cover property (req_gnt(3));
Coverqgnt_4 : cover property (req_gnt(4));
Coverqgnt_5 : cover property (req_gnt(5));
```

### Data-Oriented Functional Coverage
Data-oriented functional coverage is user-instumented coverage using SystemVerilog cover groups to focus of design data. 
The following are some key aspects of data-oriented functional coverage:
* It is user specified - not automatically inferred from the design 
* You write a test plan capturing the functionality to be tested:
  * Based on design specification and independent of actual implementation - less likely to verify "what I built" rather than "what is        should have built.
  * Needs to capture how to test given feature , how to check the test passes and how to measure the test is successfully applied (coverage).
* Coverage model is defined based on the test plan
* The simulator counts how many times a variable hits the value or transitions defined in the coverage model

### Cover Group
A covergroup is userdefined type specifying a coverage model. the user can construct multiple instances of that type in different contexts.
* Define a coverage module using **covergroup** keyword and you can declare it in a package/interface/module/program/class
* A covergroup contains:
  * A **sampling event** whose coverage can be manually sampled
  * A **coverprint** for each variable to track which can also track integral expressions
* Define a variable covergroup name
* Instantiate the coverage model using **new** - one covergroup can have multiple instantiations in different contexts.

```sv
module example;
  logic clk;
  logic [2:0] address;
  logic [7:0] data;
  
  covergroup cg1 @(posedge clk);
    c1 : coverprint address;
    c2 : coverprint data;
  endgroup : cg1
  
  cg1 cover_inst = new();
endmodule
```

### Coverpoint and Automatically Created Cover Point Bins
A **cover point** is a user defined item in a coverage model specitying an expression to cover and optionally a condition guarding its sampling. A **coverage bin** is a tool-defined or user-defined counter associated with a cover point value set, a cover point value transition set, or a cover cross tuple set. **Automatic cover point bin** - by default, the tool automaticaly creates a unique bin for rach value.
* Each coverpoint generatres a series of counters (bins) - stored in a database
* By default - there is one for each coverprint value - up to a preset limit and named **auto[<value]>**
* At every sample point, the corresponding bin for the coverprint value is incremented.

```sv
module example;
  logic clk;
  logic [2:0] address;
  logic [7:0] data;
  
  covergroup cg1 @(posedge clk);
    c1 : coverprint address;
    c2 : coverprint data;
  endgroup : cg1
  
  cg1 cover_inst = new();
endmodule
```

### Cover Point Bin for Values (Explicit Bins)
* You can define the bins yourself - to track only a subset of values and to control whats values increment which bin 
* Use the **bins** keyword and provide a bin name and a list of value ot value ranges 
* With explicit bins, unlisted values are not tracked
* Each coverprint can have multiple **bins** clauses 

```sv
c1 : coverprint var1 {
  bins V = {1,2,5}; } // Bin V increments for var1 = 1, 2 or 5
  } // No semicolon at end of coverpoint with explicit bins
  
// value list examples
/* { [0:5] , 10} - values 0-5 and 10  
{ [0:5] , [9:14} - values 0-5 and 9-14
*/
```

### Explicit Scalar and Vector Bins 
You can define scalar or vector bins:
* Scalar bin 
  * A single bin
  * Incremented for all values in value range list 
* Vector bin
  * A unique bin for each value in the range list 
  * Incremented when variables takes the corresponding value
* You can mix scalar and vector bins for the same coverpoints

```sv
c1 : coverprint var1 {
  bins V = {1,2,5}; } // created a single bin cs.V
} 
  
cv : cover point var 1 {
  bins V[] = {1 ,2 ,5} ; [] for 3 values created 3 bins
  // cv.V[1] , cv.V[2] and cv.V[5]
}
```

Each coverpoint can define:
* Bin(s) for values that are illegal even if they appear in other bins 
* Bin(s) for values to be ignored even if they appear in other bins
* A single bin for all the values in range list
* An uncostrained array giving a perarate bin for each unique value in a range list
* A fixed number of bins for all values in a range list - duplicates retained and ignored and illegal values removed after distribution 
* A default bin for all remaining values

```sv
logic [3:0] var 1 {

ce : coverpoint var1 { 
  // 1 bin for illegal values {0,15}
  illegal_bins a = {0,15};
  // 1 bin for ignored values {13,14} 
  // (value 15 is illegal)
  ignore_bins  b = { [13:15]};
  // 1 bin for {2,3}
  bins c = {2 ,3}
  // 3 bins for d[1], d[2] , d[3]
  bins d [ = { [0:2] , 2, 6] }; // bin for each unique value
  // 2 bins - e[0] = 
  bins e[2] = { [9:11] . 9, [12:15] };
  // 1 bin for {4,5,7,8}
  bins f = default;
```
**Notice!**
The illegal_bins will report error in simulation immediately when its bins hit.
The ignore_bins won't show error in simulation.
Your testbench should never hit illegal_bins. If it does, your testbench or design has a problem and all coverage is meaningless.

### Cover Cross
A cover cross is a user-defined item in a coverage model specifying a cross-product of cover points to cover and any optionally a condition guarding is sampling.

You can track cross-products of:
* Coverpoints within the covergroup
* Other scope variables:
  * Creates implicit coverpoint 
  * Participates in cover cross
  * No separate coverage for variables

Use the **cross** keyword and provide:
* A cross name (optional) 
* List of coverpoints and/or variables 

Bins are created for every cross-combination

```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;
typedef enum bit[1:0] {REG0, REG1, REG2, REG3} regs_t;

op_t opc;
regs_t regs;

covergroup cg @(posedge clk);
  c1 : coverpoint opc;
  c2 : coverpoint regs;
  opcxregs = cross c1,c2;
endgroup
```

### Automatically Created Cover Cross Bins 
A coverage bin is a tool defined or user-defined counter associated with a cover point value set, a cover point value transition set, or a cover cross tuple set. For cross-products the tool automatically creates a unique bin for each tuple.


```sv
logic [1:0] a;
logic [3:0] b;
logic c;

coverage cg @(poseedge clk);
  bcp : coverpoint b {
    bins b1 = { [9:12] }; // one bin b1
    bins b2[] = { [13:15] }; // 3 bins b2[13] , b2[14] b2[15]
    bins restofb[] = default; // not in cross 
    }
 cpp : coverpoint c;
 // implicit coverpoint for a 
 axbxc : cross a , bcp , ccp; // 32 bins = a(4)* bcp(4)$cpp(2)
```

### Defining a Cover Cross Bin (Explicit Cross bin and Select Expressions)
You can explicit cross coverage bins:
* **binsof** - selects specific bins from a coverpoint
* **intersect** - filters bin selection to specified value 
* You can use **!** , **&&** , **||** on resulting slections

```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;
typedef enum bit[1:0] {REG0, REG1, REG2, REG3} regs_t;

op_t opc;
regs_t regs;

covergroup cg @(posedge clk);
  c1 : coverpoint opc;
  c2 : coverpoint regs;
  opcxregs = cross c1,c2 { 
    bins x1 = 
      binsof (c1) intersect {[ADDI : XORI]} && 
      binsof (c2) intersect {REG0 , REG3};}
endgroup
```

### Defining Cover Cross with illegal and Ignored Cross Bins
* Use **ignore_bins** to excludes bins from a cross - even if selected elsewhere in the same cross.
* Use **illegal_bins** to specify illegal bins - even if selected or excluded elsewhere in the same cross.

```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;
typedef enum bit[1:0] {REG0, REG1, REG2, REG3} regs_t;

op_t opc;
regs_t regs;

covergroup cg @(posedge clk);
  c1 : coverpoint opc;
  c2 : coverpoint regs;
  opcxregs = cross c1,c2 { 
    bins x1 = ! binsof(c2) intersect {[REG2:REG3]};
    ignored_bins x2 = binsof(c1) intersect {[JMP:JUMPC]};
    // x1 - select all bins nuo of REG2 or REG3 but ignore all JMP or JUMPC bins
endgroup
```

### Defining Easier Cover Cross Bins 
Complex cross expressions can be avoided using dedicated coverpoints:
* Define multiple coverpoints for required values or ranges
* Cross coverpoints directly


```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;
typedef enum bit[1:0] {REG0, REG1, REG2, REG3} regs_t;

op_t opc;
regs_t regs;

covergroup cg @(posedge clk);
  c1 : coverpoint opc;
  c1nj : coverpoint opc {
    bins opnj = {[ADDI:XORI] , CALL};}
    
  c2 : coverpoint regs;
  c201 : coverpoint regs {
    bins reg01 = {REG0,REG1};}
    
  oxpr : cross cl1nj, c201 ; // one bin opxr.<opnj,re01>
  // select all bins not of REG2 or REG3 but ignore JMP , JUMPC bins
endgroup
```

### Cover Groups in Classes 
To define the cover group in the class definitiaon is an intuitive way to define the coverage model for the class.
1. Define the class member cover group instance
  * The declaration create and instance variable of the anonymous cover group type - you cannot create another instance of that type
  * The cover group definition can acess any class properties - even if **protected** or **local**
2. Construct the cover group instance 
3. Define cover group smapling 
  * For a design or test component class object, define a sampling event 
  * For a data class object, upon communicating the data object call the cover group method **sample()**.

```sv
class covlass;
  logic [2:0] address;
  logic [7:0] data;
  
  covergroup cg1;
    c1 : coverpoint address 
    c2 : coverpoint data;
  endgroup cg1;
  
  function new();
    cg1 = new(); // a class covergroup insance is not separatly named
  endfunction
endclass

covclass one = new()
  one.cg1.sample()
```

### Defining a Cover Point Bin for Value Transitions 
Coverpoints can also track transitions:
* Single value change
* Sequence of values
* Multiple changes between arrays of values

It also allows sequence syntax similar to SystemVerilog Assertions 


```sv
typedef enum bit[2:0] {ADDI, SUBI, ANDI, XORI, JMP, JUMPC. CALL} op_t;

op_t opc;

covergroup cg; 
  c1 : coverpoint opc { bins adsu ] (ADDI => SUBI); // 1 transition
    bins suan = (ADDI => SUBI => ANDI); // 1 sequence transition
    bins su3 = (ADDI , SUBI => ANDI); // ADDI => ANDI , SUBI => ANDI
                                      //  2 transitions
    bins sud = (ANDI[*3]) ;}; // ADDI => ADDI => ADDI 
```

### How to Specify Coverage Options
SystemVerilog provides options to control the behavior of :
* Covergroups 
* Coverpoints
* Crosses

There are two types of options:
1. Type options 
  * Affect every instance (static)
  * Set with **type_option** 

2. Instance-specific options 
* Can be set on idividual instances
* Set with **option**


```sv
// type option
int a,b;
covergroup cg1 @(posedge clk);
  c1 : coverpoint a {
    type_option.comment = "a";}
  c2 : coverpoint b;
endgroup : cg1

cg1:type_option.comment = "ab";
```

```sv
// instance option
int a,b;
covergroup cg1 @(posedge clk);
  c1 : coverpoint a {
    option.auto_bin_max = 10;
  c2 : coverpoint b;
endgroup : cg1

cg1 one = new();
one.c2.option.auto_bin_max = 256;
```

**Type-Specific type_option Fields**
<img width="1935" alt="Screen Shot 2022-07-28 at 15 26 30" src="https://user-images.githubusercontent.com/109002901/181504592-30f500eb-1f72-4644-903c-deecb78fc5ae.png">

**Instance-Specific option Fields**
<img width="1932" alt="Screen Shot 2022-07-28 at 15 27 11" src="https://user-images.githubusercontent.com/109002901/181504704-6d3a7bbd-e4c5-47f3-82aa-604c121728cf.png">

**Covergroup Methods**
<img width="1838" alt="Screen Shot 2022-07-28 at 15 27 28" src="https://user-images.githubusercontent.com/109002901/181504748-7abce994-862e-44db-8e3c-7e9c833b25f7.png">

cg = allowed for covergroup
cp = allowed for coverprint
cc = allowed for cover cross

## Queues and Dynamic and Associative Array
----
### Dynamic Arrays
* Use a dynamic array when an array size must change during simulation - declared by leaving an unpacked dimension unsized **[]** . 
* A dynamic array doesn't exist until it is explicity created during runtime
* Use new to create array or change size 
  * new[size] - create array initialized to default values.
  * new[size] (array) - create array initilized from existing array 
* Dynamic array built-in methods - **size()** - returns array size and **delete()** - clears elements ans sets size to 0

```sv
logic [7:0] dynarr[]; // declare dynamic array 8-bit logic

int index;

initial begin 
  dynarr = new[8]; // create array of 8-bit logic
  for (int = 0 ; i <8 ; i++)
    dynarr[i] = i+1;
  index = dynarr.size() // 8
  
  // resize array keeping values
  dynarr = new[16] (dynarr);
  index = dynarr.size(); // 16
  
  // delete array
  dynarr.delete();
  
  int mat [][];
  
  mat = new[32]; // initialize rows
  foreach (mat[i]) begin
            mat[i] = new [32]; // initialize columns 
        end 
end
```

### Associative Arrays 
Use an associative array when the data space is unbounded or sparsely populated.
Declare the array by specifying a type instead of a size for its one dimension.
* The key can be any nonreal type for which the equality operator is defined
* Array elements do not exist until you assigned to them 
* Array elements are "pairs" of associated key (address) and data values
* Elements are stored by key, ordered by key
* 

```sv
bit [3:0] aa1 [int] // associative array of 4 bit 2-state 
                    // with index type of int 
logic [7:0] aa2 [string] // associative array of 8 bit logic 
                         // with index type of string   
int aa3 [myclass] // associative array of 4 bit 2-state 
                  // with index type of myclass
bit aa4 [byte] // associative array of bit
               // with index type of byte 
```

**Associative Array Methods**

<img width="1785" alt="Screen Shot 2022-07-28 at 17 20 49" src="https://user-images.githubusercontent.com/109002901/181538113-720f761f-3583-457e-93ef-db945b7aa3d3.png">

### Queues
* Queues are dynamically sized, ordered collection of elements of a declared type
* A queue is declared by specifiying **$** instead of a size for its one dimension 
* A qeueu supports access to all its elements as well as insertion and removal at the beggining or the end of the queue
* Each element is identified by a number defining its position in the queue - **0** represent the first location and **$** represents the last location

```sv
integer q_integer[$]; // queue of integers - queue of unlimited size
logic [15:0] q_logic[$]; // queue of 16-bit logic
int q_int[$:2000]; // queue of int - max size of 2000
time q_time[$:10] // queue of time - max size of 10
```

**Queue Methods**

<img width="1867" alt="Screen Shot 2022-07-28 at 17 34 20" src="https://user-images.githubusercontent.com/109002901/181553177-d06a77e8-5a17-4d50-b158-4de904b22a39.png">

You can delete with empty queue assignment

### Array Manipulation Methods
Array manipulation methods are built-in methods that support searching, ordering, and reducing of array. These methods apply to any unpacked array, with the following exceptions: 
* Locator methods cannot be applied to assoiative arrays that use the wildcard index type
* Ordering methods cannot be applied to associative arrays

* Locator 
  * Search array for elements or indexed that satisfy an expressiong - attached using **with**
  * Return a queue of those elements or indexes

```sv
// extract all elements from array_int greater than 5
q_int = array_int.find with (item > 5);
```

<img width="1447" alt="Screen Shot 2022-07-28 at 17 48 05" src="https://user-images.githubusercontent.com/109002901/181566075-77337a05-cacb-4b3d-aeb5-d50f7d7591ab.png">

* Ordering 
  * Reorder an array 
  * Cannot be applied to associative arrays

```sv
// sort q_int in ascending order 
q_int.sort;
```

<img width="1118" alt="Screen Shot 2022-07-28 at 17 48 34" src="https://user-images.githubusercontent.com/109002901/181566267-28c4f028-4306-4f43-83c9-6e625accfc6c.png">

* Reducation 
  * Reduce an array of integral values to a single value

```sv
// extract xor reduction of all elements in q_int 
var_int = q_int.xor;
```

<img width="1081" alt="Screen Shot 2022-07-28 at 17 49 50" src="https://user-images.githubusercontent.com/109002901/181566792-e069b326-8911-4393-941a-7f21cf89d6cb.png">

## Assertion Based Verification 
----
### Assertion 
An assertion is a directive to an EDA tool to verify that a property is always true.
* A property is a description of design behavior
* We instruct the tool what to do with the property using verifiction directives - **assert**, **cover** , **assume** and **restrict**
* An assertion is a check that a property is always true 
* During verification, assertions continually observe: 
  * If a specific condition accurs or 
  * If a specific sequence of events occurs
* Assertions offer a way to signicantly enchance productivity for designers - find bugs earlier and more easily 

<img width="725" alt="Screen Shot 2022-07-28 at 18 27 46" src="https://user-images.githubusercontent.com/109002901/181577056-5200a5dc-19f5-4d12-92ef-6154ad12bd1b.png">

<ins>**Why Use Assertions**</ins>

Improve observability 
* Automatically and constanly checks behavior
* Can detect hard-to-find bugs embedded deep in the design 
* Isolates problem close to source 
* Can be applied non-intrusively to legacy designs

Improves verification efficiency 
* Can concisely and unambiguously document design intent 
* Encapsulation and reuse 
* Can detect bugs earlier in the design cycle 
* Trap and exit on errors, saving simulation cycles
* Writing assertions helps designers to better understand their design and do more debugging

Assertions can be embedded in code, to "travel with" design form
* Project to project - ideal for IP
* Tool to Tool - ideal for design methodology interity 

<ins>**Who Writes Assertions and Why**</ins>

<img width="1898" alt="Screen Shot 2022-07-28 at 18 39 06" src="https://user-images.githubusercontent.com/109002901/181579626-834f7360-5f22-4815-a5c9-8f676dc4b8ce.png">

<ins>**Issues with Assertions**</ins>

* Assertions must be constructed with care:
  * Incorrectly specified assertions can give misleading results
  * Debugging an assertion can be difficult 
* How do you know when enough assertions have been written
* It is easy to shoot yourself in the foot using overly complex SVA constructs - the syntax "sugared" - simplaicity of the syntax hids complex overlapping behaviors
* Qulity of test stimulus is critical
  * The simulator can make only those checks which the test exercises
  * Coverage metrics reveal which checks are exercised 
* Checking assertions consumes CPU cycles

### Assertions-Based Verification
Assertion-based verification is the structured use of assertions to describe an verify design properties.
* Assertions monitor and report - expected behavior and forbidden behavior
* Assertions are used by static verification tools - no test vector, formal (mathematical) proof. Dynamic verification tools - dependent upon simulation test effectiveness (as measured using functional coverage)
* ABV also encompasses SVA constructs for defining coverage

<img width="494" alt="Screen Shot 2022-07-28 at 23 45 51" src="https://user-images.githubusercontent.com/109002901/181634318-3940deb0-e4dd-489a-9c8f-48356da443d2.png">

### HDL-Based Assertions
You have been "asserting" correct design condition all along using: **if (!good_condition) $display (error_message);** .

<ins>**Why is ABV Better Than HDL-Based Assertions**</ins>
ABV goes well beyond HDL-based assertions:
* Standard language constructs to concisely express complex temporal behaviors (PSL and SVA)
* Standard tool support taking advantage of new constructs
  * Failure messaging
  * Statistics gathering
  * Debug features 
* Definition of functional coverage

## Introduction to SystemVerilog Assertions 
----
### Concurrent Assertions
Concurrent assertions describe behavior that spans over time. Unlike immediate assertions, the evaluation model is based on a clock so that a concurrent assertion is evaluated only at the occurrence of a clock tick.
* Concurrent assertions can be embedded in functional code and ignored by synthesis
* They are supported by mainstrean simulator are formal verification tool
* You can define and assert the functional properties of a design with concurrent assertions:
  * Boolean expressions about some behavior design
  * What it should or should not do 
  * Can be single od multiple cycle
  * Uses Verilog syntax
  * Mathematically precise and computationality efficient - sufficiently expressive to specify "real world" design properties 


### Concurrent Assertions Structure

<img width="1797" alt="Screen Shot 2022-07-31 at 09 02 58" src="https://user-images.githubusercontent.com/109002901/182012517-30671527-5bce-4d94-af2f-dab7b6e92296.png">

### Concurrent Assertions Placement
* Properties are declared as normal SystemVerilog declarations - usually in a module or interface
* Properties are asserted as concurrent statement - usually in a module of interface 
* Also legal inside an always or initial block - strong recommendation not to do this

Test expression is evaluated at clock edges based on values in sampled variables
Sampling of variables is done in the preponed region and evaluation of the expression is done in the observed region of the simulation scheduler.
It can be placed in a procedural, module, interface, or program block
It can be used in both dynamic and formal verification techniques

```sv
module mod (input logic clk, output logic en1, en2);

  always @ (negedge clk)
    begin 
      (en1,en2) <= sel[1:0];
    end
  
  'ifndef USING_OLD_TOOL
    property RW_CHK_CLK;
      @ (negedge clk) en1 || en2;
    endproperty 
    
   A1 : assert property (RW_CHK_CLK); 
   'endif
endmodule
```

### Defining a Simple Property 
* Define a design behavior as a Verilog Boolean expression
* Any expression allowed in an **if** condition can be used ,including functions and out of module references 
* Subject to normal Verilog restrictions, particulary - case sensitivity and naming rules 

### Naming and Asserting the Property 
You have two options: 
* Declare and instance the property on the same line 

```sv
ASSERT : assert property ( @ (posedge clk) en1 || en2);
```

* Declare and instance the property in separate statements - gives more flexability and property can have arguments 

```sv
property RW_CHK;
  @ (posedge clk) en1 || en2;
endproperty
// Always label assertions 
ASSERT1 " assert property (RW_CHK); // parentheses for named property required
```

### Clocking the Property 
* Properties are evaluated upon standart Verilog clocking events
  * **@identifier** or **@(event expression)**
  * Can alsu use SystemVerilog enhancements (such as **iff**
* All assertions must be clocked 
* Clocking expression does not need to be a design hardware clock or periodic 
* Good clocking expression can be greatly simplify properties 
* A default clock can be specified 

```sv
property RW_CHK;
  @ (posedge clk iff VALID) en1 || en2;
endproperty

property HW_CHK;
  @ (negedge addr_en) addr <= 7;
endproperty
```

### Assertion Evaluation
Assertions are sempled, clocked and evaluated at strictly deifined poinrs in the simulation cycle.
* Property variables are sampaled in the preponed region 
* Properties are evaluated in observed region using preponed values 


### Assertion Binding 
Binding allow to specify one or more instantiations of an assertions module , interface program ot checker without modifying the code of the target with **bind** construct.
* The **bind** statement can be placed literally anywhere 
* Instantiates an assertion module in instance(s) of hierarchical design module 

<ins>**Advantages**</ins>

* No changes to design module
* Easier to modify and update
* Enhances reusability

* Can bind to all of specific instances of a design module:
  * To bind to every instatiations of **mone** 
```sv
bind mone mone_prop...
```
  * To bind only to instatiations **m1** of **mone** 
```sv
bind mone:m1 mone_prop...
```

```sv
// assertion module
module mone_prop(input a,b,clk);
  property P1;
    @ (posedge clk) $ros(a) |=> b;
  endproperty
  A1 : assert property (P1);
endmodule
```

```sv
// design module 
module mone (input logic clk, reg , 
  output logic ack);
  
  logic half, empty,full; // internal signals 
  ~~mone_prop mp1 (reg , ack, clk);~~ // moved to testbench
endmodule
```

```sv
// testbench module 
module top;
  
  // instantiate design modules 
  mone m1 (clk , req1 ack1);
  mone m2 (clk , req2, ack2);
  
  // design module , assertion module , instantiation name , design module var
  bind mone mone_prop mp1 (reg,ack, clk, hald ,empty ,full); // signal names in design
                                                           // module scope
  
endmodule
```

<ins>**Application**</ins>

Connecting assertion moudle port signals to internal signals of the design module using **bind** .

* Internal signals o design module van be connected to port signals of assertion module
* If names don't match, the **.name** notation can be used for connection

### Sequence Expression
A sequence expression describes a series of one or more cycles of design signal states (each cycle desribed by a boolean expression).
* Simple boolean properties are instantaneous - signal pass/fail test at the evaluation point
* Multi-cycle properties are described using sequences: 
  * Series of boolean equations
  * Evaluated in successive clocking cycles
  * Each cycle separated by ##N
* Sequences are building blocks of properties 
* There is a wide range of operators and features to construct complex sequences - if-then implication , repetition , composition operators

```sv
@ (negedge clk)
a ##1 b ##1 c;
```

<img width="715" alt="Screen Shot 2022-07-31 at 10 38 53" src="https://user-images.githubusercontent.com/109002901/182015339-6a69e036-fab4-47f3-9809-4d0f975ca4dc.png">

### Defining a Cycle Delay 

<img width="2064" alt="Screen Shot 2022-07-31 at 10 41 16" src="https://user-images.githubusercontent.com/109002901/182015422-53adc4ab-ada2-4911-80f9-c4e056587b9f.png">

### Implication |-> : Same Cycle 
* Conditional properties are deifned with implication operators:
  * If **expra** is true, then **exprb** must occur
  * if **expra** is not true, then **exprb** is not checked
* For the same cycle implication **exprb** must be true in the same cycle as **expra**
* For implication properties:
  * **expra** is the antecedent ot enabling condition
  * **exprb** is the consequent or fulfilling condition
  * Both can be either sequences or boolean conditions

```sv
expra |-> exprb
```

```sv
property SCI;
  @ (negedge clk)
    // check the condition only when req is high and ack is low 
    (reg && !ack) |-> bsy;
endproperty
```

<img width="830" alt="Screen Shot 2022-07-31 at 10 46 55" src="https://user-images.githubusercontent.com/109002901/182015617-5a8a918e-fc70-4d17-9e79-a9a6e324e1fb.png">


### Implication |=> : Next Cycle 
* For the next cycle implication, **exprb** must be true in the cycle after **expra** completes.
* Otherwise, same rules apply:
  * If **expra** is true, then **exprb** must occur
  * if **expra** is not true, then **exprb** is not checked
  * Both can be either sequences or boolean conditions
* If a variable is not included in a condition, its value is "don't care":
  * **bsy** is not checked in enabling condition 
  * reg and ack are not checked in the fulfilling condition 

```sv
expra |=> exprb
```

```sv
property NCI;
  @ (negedge clk)
    (reg && !ack) |=> bsy;
endproperty
```
<img width="875" alt="Screen Shot 2022-07-31 at 10 54 01" src="https://user-images.githubusercontent.com/109002901/182015834-1847e4a5-46ee-4dd1-a3f6-29734daab8bd.png">

<ins>**Pictorial View of Assertion Property Evaluation States**</ins>

An assertion can be in one of several states:

<img width="1740" alt="Screen Shot 2022-07-31 at 12 48 56" src="https://user-images.githubusercontent.com/109002901/182020644-8b332dcf-960a-4f92-a0bb-aa7909a849d6.png">

### Disabling Properties with disiable iff
* To terminate an assertion when a condition is true use **disable iff**
* if **expr** is true at any time, then the property is terminated 
* Disable expression must be bracketed ()
* Useful for cancelling multi-cycle sequence properties - for example,burst on a bus interrupted by reset 
* A default disable can be specified

```sv
property NAME;
  @(clocking) disable iff (expr)
    the_property;
endproperty
```
```sv
property ABORT;
  @ (negedge clk) disable iff (rst)
    a |=> b ##1 c ##1 d;
endproperty
```
<img width="828" alt="Screen Shot 2022-07-31 at 13 03 39" src="https://user-images.githubusercontent.com/109002901/182021199-f10a985a-7c78-4dd5-be66-58ecc8be19d2.png">

### Cycle Delay Repetition 
* **##** can be used with any constant integral expression to define a number of cycle delays.
* After expr1, count N evaluation points, after which **expr2** must be true
* ##0 is allowed (and useful) for sequences - zero delay ("fusion")

```sv
property A3B_NEXT;
// if a is low then b is high 4 cycles later
  @ (negedge clk) !a |=> ##b;
```

### Consecutive Sequence Repetiton
You can specify multiple consecutive repetition using **[* N]** :
* Consecutive repetition operator
* **expr** repeats consecutively N times
* N must be a non-negativ integer constant expression

```sv
expr[*N]
```

```sv
// instead of this
property SEQ_COUNT;
  @ (negedge clk) !a ##1 !a ## 1 !a ##1 !a |=> a;
endproperty

// write this
property CONSEC;
// a is never low more than 4 cycles
  @ (negedge clk) !a[*4] |=> a;
endproperty
```

<img width="940" alt="Screen Shot 2022-07-31 at 13 21 46" src="https://user-images.githubusercontent.com/109002901/182021757-95f238cf-eb43-40d8-8c3d-39f1fd188c79.png">

### Consecutive Repetition with Ranges 
* You can specify a range for consecutive repetition
* **expr** repeats consecutively for:
  * At least **min** number of times 
  * At most **max** number of times 
* Range limits must be non-negative interger constant expressions:
  * **min** can be 0 - no repetitions
  * **max** can be $ (unlimited) - can repeat forever
  * **min** <= **max**

```sv
expr[*min:max]
```

```sv
property RANGE;
  @ (negedge CLK)
  // a goes low in between 2 and 4 cycles only 
    a ## 1 !a |=> !a[*1:3] ##1 a;
```

<img width="840" alt="Screen Shot 2022-07-31 at 13 32 13" src="https://user-images.githubusercontent.com/109002901/182022082-bece5f13-f452-46c3-8551-7089e78dad30.png">

### Additional SVA Features
SystemVerilog has many more advanced properties, sequence operators and features: 
1. Conditional property operators 
2. Nonconsectuive sequence repitition
3. Concurrent , alternate and overlapping 

<ins>**Advance SVA feature**</ins>

* Detectiong the end of a sequence 
* Parameterized sequences and properties
* Action blocks on property pass or failure
* Functional coverage 

The SystemVerilog Assertions class covers all the above including the following:
* Coding guidelines and recommendations
* Practical examples
* Formals Friendly SVA coding
* Use of Verilog helper code to simplify verification using SVA properties 


## Interproccess Synchronization
----
### Blocking Event Trigger - >
* Traditional Verilog event trigger is blocking - triggered instantaneoulsy in Active region when executed
* Synchronization can be difficult - proccess to be triggered must be waiting for the event when its occurs

```sv
event e;
integer i = 0;

always @e $display ("i:%0d",i);

initial begin 
  i <= 1;
  -> e; // blocking i:0
```

### Nonblocking Event Trigger - ->>
* You can trigger SystemVerilog events with nonblocking **->>** operator:
  * Executes without blocking 
  * Schedules an event for the NBA region
  * Helps prevent races between proccesses
  * Can include optional embedded event control


```sv
event e;
integer i = 0;

always @e $display ("i:%0d",i);

initial begin 
  i <= 1;
  ->> e; // nonblocking i:1
  
  ->> #3 e;
  ->> @ (posedge clk) e;
```

### Persistent Event Trigger - triggered
A proccess can wait for the **triggered** property of the event - the **triggered** state persists to the end of the time step

```sv
event e1, e2;

initial begin 
  $display ("fork");
  fork 
    @e1;
    -> e1;
    -> e2;
    @e2;
  join 
  $display("join);
  // output - fork
```

```sv
event e1, e2;

initial begin 
  $display ("fork");
  fork 
    @e1;
    ->> e1;
    -> e2;
    wait (e2.triggered);
  join 
  $display("join);
  // output - fork join
```

### Mailbox
* Mailboxes are a message-based synchronization mechanism
  * Used for passing messages where order is important (FIFO)
  * Intended for interpocedure communication and synchronization

* Mailboxes can block procedure execution
  * Writing to a full mailbox blocks until the mailbox is no longer full
  * Reading from an empty mailblock blocks untill the mailbox is no longer empty

* Mailboxes can be bound (to set size) or unbound - Unbounded mailboxes can never be full
* Mailboxes can be type-less or parameterized:
  * Each type-less (default) mailbox can contain data of different types 
  * Parameterized mailbox can contain only one type of data 
  * Mailbox connections 

```sv
mailbox hugebox = new; // mailbox of unlimited size

mailbox fourbox;;
fourbox = new(4); // mailbox of size 4 
```

<ins>**Mailbox Methods**</ins>

<img width="1901" alt="Screen Shot 2022-07-31 at 14 12 00" src="https://user-images.githubusercontent.com/109002901/182023511-17c90abb-cddf-495d-9a9d-24b3bf22eba7.png">

### Parameterized Mailboxes
You can type-parameterize a mailbox:
* Define type upon declaration **mailbox #(type) namebox = new;**
* Holds messages of that and equivalent types
* Compiler detects type mismatch

### Semaphore
* Semaphores are a key-based synchronization mechanism
* They are intended for process synchronization, mutal exclusion and controlling access to a shared or limited resource 
  * A procedure requests semaphore key(s) before accessing the resource
  * A procedure returns semaphore key(s) after accessing the resource
* Semaphores can block procedure execution - if the procedure requests more keys than the semaphore holds, the execution is blocked untill sufficient keys are returned
* Key management is the responsibility of the designer

```sv
semaphore keybox = new(5); // semaphore with 5 keys

semaphore sync;
sync = new(4); // semaphore with 4 keys
```

<ins>**Semaphore Methods**</ins>

<img width="1928" alt="Screen Shot 2022-07-31 at 14 25 08" src="https://user-images.githubusercontent.com/109002901/182023980-3e17d747-3c71-4fd4-93d8-c84dd5ebf913.png">

### Event Variables
A SystemVerilog event variable is a handle pointer to a synchronization queue.
* You can assign and compare these handles to each other - assignment causes both handles to point to the same queue

```sv
event e1, e2;
initial 
  fork 
    #1 e2 = e1; // e1, e2 both have 21 synchronization queue
    #2 @e1 $display ("e1 triggered");
    #2 @e2 $display ("e2 triggered");
    #3 -> e2; // trigger e2 (and also e1)
  join
// output : e1 is triggered , e2 is triggered
```

### Merging Events
Merging events merges only the event variables - both now point to the RHS synchronization queue.

```sv
event e1, e2;
initial 
  fork 
    #1 @e1 $display ("e1 triggered");
    #1 @e2 $display ("e2 triggered");
    #1 e2 = e1; // e1, e2 both have 21 synchronization queue
                // e2 synchronizaiton queue is lost 
    #3 -> e2; // trigger e2 (and also e1)
  join
// output : e1 is triggered 
```

### Reclaiming Events
You can assign **null** value to an event.
* Triggered a null event shall have no effect 
* The effect of waiting on null event is undefined

```sv
event e1;
initial 
  fork 
    #1 @e1 $display ("e1 triggered once");
    #2 -> e1 $display ("e2 triggered");
    #3 e1 = null; // e2 synchronizaiton queue is lost 
    #4 -> e1 $display ("e2 triggered twice"); // @e1 may not block
                                              // or may block forever
    #5 -> e1; // nothig happens
              // triggering null event e1 has no effect
  join
// output : e1 is triggered once
```
