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

### Bitwise vs logical
- various boolean operations (like `|` or `||`) 
- A bitwise operation between two N-bit vectors replicates the operation for each bit of the vector and produces a N-bit output, while a logical operation treats the entire vector as a boolean value (true=non-zero, false = zero) and produces a 1-bit output. 

### Reduction Operators
- Convert vectors to scalars.
- They operate on all of the bits in a vector to convert the answer to a single bit.
```
module reduction_operators();
    initial
        begin
            $display("AND  Reduction of 4'b1101 is: %b", &4'b1101);
            $display("AND  Reduction of 4'b1111 is: %b", &4'b1111);
        end
endmodule
```
Would print out 0 (0 & 0 & 0 & 0) and 1 (1 & 1 & 1 & 1).

### Concatentation
{a, b, c}; verilog must compute total_width = width(a) + width(b) + width(c); Therefore any size must be known at compile time.

### Connecting Signals to Module Ports
1. By Position
When instantiating a module, ports are connected left to right according to the module's declaration. For example:
```
mod_a instance1 (wa, wb, wc);
```
- One drawback of this syntax is that if the module's port list changes, all instantiations of the module will also need to be found and changed to match the new module.

2. By Name
- Connecting signals to a module's ports by name allows wires to remain correctly connected even if the port list changes. This syntax is more verbose, however. 
```
mod_a instance2 (.out(wc), .in1(wa), .in2(wb));
```
- Note how the ordering of the ports do not matter in this method. 
- Note the period immediately preceding the port name in this syntax.

## HDLBits Circuits
### Multiplexers
Create a one-bit wide, 2-to-1 multiplexer. When sel=0, choose a. When sel=1, choose b. 2-to-1 Multiplexer is simply:
```
input a, b, sel;
output out;

assign out = (sel & b) | (~sel & a);
assign out = (~sel) ? a : b; // with ternary operator
```
For a bus multiplexer
```
module top_module (
	input [99:0] a,
	input [99:0] b,
	input sel,
	output [99:0] out
);

	assign out = sel ? b : a;
	
	// The following doesn't work. Why?
	// assign out = (sel & b) | (~sel & a);
	
    // Correct way to do the previous method
    // assign out = ({100{sel}} & b) | (~{100{sel}} & a);

endmodule
```

The following does work as while '&' does work on vector arguments, the inputs widths do not match up (sel = 1-bit width vs b/a = 100-bit width). This now changes out assignment as different tools will handle the midmatch differently. Potentially with the sel being implicityly extended in an unintended way. 

Mux9to1v
My solution
```
module top_module( 
    input [15:0] a, b, c, d, e, f, g, h, i,
    input [3:0] sel,
    output [15:0] out );
    always @(*) begin
        case(sel)
            3'b000: begin
            	out = a;
            end
            3'b001: begin
            	out = b;
            end
            3'b010: begin
            	out = c;
            end
            3'b011: begin
            	out = d;
            end
			3'b100: begin
            	out = e;
            end
            3'b101: begin
            	out = f;
            end
            3'b110: begin
            	out = g;
            end
            3'b111: begin
            	out = h;
            end
            4'b1000: begin
                out = i;
            end
            
            default:out = {16{1'b1}};
        endcase
    end
endmodule
```
Improved Solution
```
module top_module( 
    input [15:0] a, b, c, d, e, f, g, h, i,
    input [3:0] sel,
    output [15:0] out );

    always @(*) begin
        out = '1; // '1 is a special literal syntax for a number with all bits set to 1. '0 are all 0s. 'x are all unknown. 'z are all high-impedance. 

        case(sel)
            4'h0: out = a; // h's are used over b since they're a lot more common. 
			4'h1: out = b;
			4'h2: out = c;
			4'h3: out = d;
			4'h4: out = e;
			4'h5: out = f;
			4'h6: out = g;
			4'h7: out = h;
			4'h8: out = i;
        endcase
    end
endmodule
```

Create a 4-bit wide, 256-to-1 multiplexer. The 256 4-bit inputs are all packed into a single 1024-bit input vector. sel=0 should select bits in[3:0], sel=1 selects bits in[7:4], sel=2 selects bits in[11:8], etc. 

My initial solution was similar to 
```
assign out = in[sel * 4 + 3 : sel*4]
```
This is illegal in verilog because the sel is variable and this the sel *4 is arithmetic on a variable. Verilog requires part-select bounds to be constant expressions. 
What I want is a **variable part-select**. Which means we want to identify that start_index and width. 
```
assign out = in[sel*4 +: 4]
// in[start_index +: width]
```
As we know our output is going to be 4 bits wide and the starting going to be some multiple of 4 (counting 0). 
So let's run through it. Sel = 0
in[0*4 +: 4]
start index = 0 with a width of four from that point so 0 + 4 = 3 (index counting not literal arthimetic). 

