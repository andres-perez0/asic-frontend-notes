Day 2 - Understand why hw is different from sw

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


## HDLBits Circuits
- Wire, GND, Nor, Another Gate, Two gates, More logic gates,

- Combinational means the outputs of the circuit is a function of only its inputs.

# Finish-Up HDLBits to get to Multiplexers
- Don't forget to answer the following
- Why latches are bad?
- Why clocks exist?
- Why reset metters in CI?

