# Day 3/4 — Clocks, Resets, and FSMs

Goal: Understand the #1 source of RTL bugs

You must understand:
- Clock edges (posedge)
- Reset types:
    - Synchronous
    - Asynchronous
- FSM structure:
    - States
    - Transitions
    - Outputs

### Dff
- A D flip-fop is a circuit that stores a bit and is updated periodically, at the (usually) positive edge of a clock signal.

- D flip-flops are created by the logic synthesizer when a clocked always block is used. 
- A Dff is the simplest form of "block of combinational logic followed by a flip-flop" where the combinational logic portion is just a wire.

### Alwaysblock1
Since digital circuits are composed of logic gates connected with wires, any circuit can be expressed as some combination of modules and assign statements. 

Procedues (always and initial) provide an alt. syntax for describing circuits.

For synthesizing hardware, two types of always blocks are relevant:
- Combinational: always @(*)
- Clocked: always @(posedge clk)

Combinational always blocks are equivalent to assign statements, thus there is always a way to express a combination circuit both ways. 

**The syntax for code inside a procedural block is different from code that is outside.**

Time to compare two lines of code 
```
assign out1 = a & b | c ^ d;

always @(*) out2 = a & b | c ^ d;
```
`assign` - This is a continuous assignment, pure combinational logic, whenever any input changes, the output updates automatically. 

`always @(*)` 
- `always` this block keeps running forever in simulation
- `@(*)` re-run this block whenever any signal it depends on changes.

Thus, this is a procedural assignment, not a wire connection. However, they're equivalent as they recompute output whenever input changes; both synthesize to the same combinational logic. 

The critical difference is what can be assigned. 
`assign` - wire
`always @(*)` - register (verilog term, not a flip-flop)
    - Do not mean register hw, it just mean "assigned inside a procedural block."

`assign` and `always @(*)` are two syntaxes for the same combinational hardware. Use `assign` for simple expressions, `always @(*)` for complex logic.

```
module top_module(
    input a, 
    input b,
    output wire out_assign,
    output reg out_alwaysblock
);

	assign out_assign = a & b;
	
	always @(*)
		out_alwaysblock = a & b;
		
endmodule
```
### Alwaysblock2
- Clocked always blocks create a blob of combinational logic just like combinational always block, but also creates a set of flip-flops ("or registers") at the output of the blob of combinational logic. Instead of the outputs of the blob of logic being visible immediately, the outputs are visible only immediately after the next (posedge clk).

### Blocking vs. Non-Blocking Assignment
Three types of assignments in verilog:
- Continuous assignments (`assign x = y;`). Can only be ued when not inside a procedure ("always block").
- Procedural blocking assigment (`x = y;`). can only be used inside a procedure.
- Procedural non-blocking assignment (`x <= y;`). Can only be used inside a procedure.

**In a combinational always block, use blocking assignments.**

**In a clocked always block, use non-blocking assignments.** 

A full understanding of why is not particularly useful for hw design and requires a good understanding of how verilog simulators keep track of events. 

```
// Combinational logic
always @(*) begin
    x = y;      // blocking assignment
end

// Sequential (clocked) logic
always @(posedge clk) begin
    x <= y;     // non-blocking assignment
end
```

### Alwaysblock2
```
// synthesis verilog_input_version verilog_2001
module top_module(
    input clk,
    input a,
    input b,
    output wire out_assign,
    output reg out_always_comb,
    output reg out_always_ff   );
	assign out_assign = a ^ b;
    always @(*) out_always_comb = a ^ b;
    always @(posedge clk) out_always_ff <= a^b;
endmodule
```