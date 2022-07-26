SystemVerilog Fundamentals
=================

## Timing Control
----
### Delay Control
The delay control is a way of adding a delay between when the simulator encounters the statement and when it executes.
Delay control is achieved by specifying the waiting time to execution when the statement is encountered. The symbol **#** is used to specify the delay.

We can specify the delay based timing control in three ways:

* Regular delay control - It will be specified on the procedural assignment left as a non-zero number.
* Intra - assignment delay control -  In this case, delays will be specified on the assignment operator's right-hand side. The right-hand side expression will be evaluated at the current time, and the assignment will occur only after the delay.
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

## Implicit Port Connection - .name
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

## Implicit Port Connection - .*
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
  slowbus #(.WIDTH(8) bus8 (clk) // 8 bit data
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

## Interface Methods
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
An immediate assertion is an instantaneous boolean check - single cycle. 

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
