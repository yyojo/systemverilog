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
The Verilog datatypes have 4-state values: 0, 1, X, Z. SystemVerilog adds 2-state value types: 


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
