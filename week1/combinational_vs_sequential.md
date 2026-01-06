Day 2&3 - Understand why hw is different from sw

HDLBits Messages:
- Compile Error — Circuit did not compile.
- Simulation Error — Circuit compiled successfully, but simulation did not complete.
- Incorrect — Circuit compiled and simulated, but the outputs did not match the reference.
- Success! — Circuit was correct

HDLBits uses verilog 
- Does not execute line-by-line, rather:
    - `assign` statements describe wires connected to logic
    - gates, muxes, and expressions describe physical hardware
    - everything happens in parallel

Basic Syntax
```
wire out;  // wire declaration

assign out = in; // continuous assignment

// multiple inputs
assign out = a & b; // and
assign out = a | b; // or
assign out = a ^ b; // xor

assign out = ~a; // not; inverting
```
- Generally `assign` is used with wire and creates combinational logic

Two ways of describing gates (one with operators above; second below)
```
and (out, a, b);
or  (out, a, b);
not (out, a);
```
```
assign out = sel ? a : b; // 2-to-1 multiplexer

assign out = {a, b}; // concatenation
```
Declaring wires
```
module top_module(
    input in;
    output out;
);

    wire not_in;

    assign out = ~not_in;
    assign not_in = ~in;

endmodule
```
### Vector1
Vectors must be declared: 
type [upper:lower] vector_name;
type - usually wire/reg can also include the input/output type

```
wire [7:0] w;       // 8-bit wire
reg [4:1] x;        // 4-bit reg
output reg [0:0] y; // 1-bit reg that is also an output port
input wire[3:-2] z; // 6-bit wire input (negative ranges allowed)
output [3:0] a;     // 4-bit output wire. Default type is 'wire'
wire [0:7] b;       // 8-bit wire where b[0] is the most-significant bit
```

The direction of a vector is whether the least significant bit has a lower index (little-endian, e.g., [3:0]) or a higher index (big-endian,e.g.,[0:3]). Once delcared, it'll always be used in that way. 

### Implicit nets
- A net represents a physical connection (aka a wire) between hw elements. So they do not store values and carry signals driven by a source.
- Implicit nets are often a source of bugs.
- Net-type signals can be implicity created by an `assign` statement or by attaching something undeclared to a module port. 
- *Implicit nets are always one-bit wires and causes bugs if you had intended to use a vector. Disabling creation of implicit nets can be done using the* `default_nettype none` *directive.*
    - This is done by making the second line of the following code an error, which makes a bug more visible. 

```
wire [2:0] a, c;    // Two vectors
assign a = 3'b101;  // a = 101
assign b = a;       // b = 1; implicity-created wire
assign c = b;       // c = 001 <- bug
my_module i1 (d,e); // d and e are implicitly one-bit wide if not declared. This could be a bug if the port was intended to be a vector. 
```
L1: declares two 3-bit wires a and c; with a/c[2] being the MSB and a/c[0]
L2: Continuous assignment drives `a` to `101`, so a[2]=1,a[1]=0,a[0]=1;
L3: b get implicity created as a wire (1-bit wide) and thus truncates it down to it's LSB (a[0]=1)
L4: given c is already declared as a 3-bit wire, it takes the 1 to 001

### Unpacked vs. Packed Arrays
- "packed" dimensions of the array, where the bits are "packed" together into a blob
- "unpacked" dimensions are delcared after the name. They are generlaly used to delcare memory arrays. 
```
reg mem1 [28:0];        // 29 unpacked elements, each of which is a 1-bit reg
reg [7:0] nem2 [255:0]; // 256 unpacked elements, each of which is a 1-bit reg
```

### What is a reg?
- Is an abstraction of a data storage element;
- can store a value
- keeps that value until changed

### Accessing Vector Elements
```
w[3:0]      // Only the lower 4 bits of w
x[1]        // The lowest bit of x
x[1:1]      // ...also the lowest bit of x
z[-1:-2]    // Two lowest bits of z
b[3:0]      // Illegal. Vector part-select must match the direction of the declaration.
b[0:3]      // The *upper* 4 bits of b.
assign w[3:0] = b[0:3];    // Assign upper 4 bits of b to lower 4 bits of w. w[3]=b[0], w[2]=b[1], etc.
```

### Writing modules in verilog
Question MT2015_q4 in Basic Gates
```
module top_module(input x, input y, output z);
    wire o1, o2, o3, o4;

    A ia1 (x, y, o1);
    B ib1 (x, y, o2);
    A ia2 (x, y, o3);
    B ib2 (x, y, o4);

    assign z = (o1 | o2) ^ (o3 & o4);

endmodule

module A (input x, input y, output z);
    assign z = (x^y) & x;
endmodule

module B (input x, input y, output z);
    assign z = ~(x^y);
endmodule
```


## HDLBits Circuits
- Combinational means the outputs of the circuit is a function of only its inputs.



# Finish-Up HDLBits to get to Multiplexers
- Don't forget to answer the following
- Why latches are bad?
- Why clocks exist?
- Why reset metters in CI?


