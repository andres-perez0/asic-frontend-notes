# Day 1
- Infrastructure 
    - Everything that enables designers to design, without designing the chip itself.
- Continuous Integration (CI)
    - Essentially an automation verification pipeline.
    - Whenever RTL changes:
        - CI automatically runs checks
        - CI decides what to rerun
        - CI reports pass/fail

## What is RTL?
Imporved Answer:
Register Transfer Level (RTL) is a cycle-accurate hardware description of a digital design that specifies:
- Registers (state elements like flip-flops)
- Combinational logic between those registers
- How data moves between registers on clock edges

RTL is typically written in Verilog or VHDL and is the source of truth for ASIC design. It is detailed enough to be:
- Simulated to verify functional correctness
- Synthesized into gates during logic synthesis
RTL  describes what the hardware does each clock cycle, not how it is physically laid out on silicon.

## Who uses RTL?
RTL/Microarchitecture Designers: Write and modify RTL to implement functionality
Verification Engineers: Use RTL to build testbenches, assertions, and simulations
Synthesis Tools: Convert RTL into gate-levels netlists
Formal Verification tools: Prove properties about RTL behavior
Frontend Infrastructure & CI Systems: Complite, simulate, lint, and validate RTL automatically

## Why simulation is critical?
Simulation is critical because its the main way to verify functionality before fabrication.

In ASIC design: Bugs found after synthesis or layout are expensive, Bugs found after fabrication are often catastrophic

Simulation allows designers to:
- Validate RTL behavior across clock cycles
- Check corner cases and reset behavior
- Detect protocol, FSM, and timing-related logic bugs early

Simulation provides fast feedback at high abstraction levels, making it the most cost-effective bug detection stage in the flow.

## What happens after RTL?
Simulation & Verification -> Logic Synthesis -> Static Timing Analysis -> Physical Design -> Signoff & GDS Generation

## What happens when RTL changes
When RTL changes, it affects the entire ASIC flow. 
As an RTL modification can trigger:
- Re-simulation (functionality verification)
- Re-synthesis (gate-level netlist changes)
- Re-timing and constraint re-evaluation
- Potential changes in the GDS (placement/routing)

- Because the ASIC flow is interdependent, even small RTL changes can invalidate previous verification results, cached CI artifacts, timing closure assumptions. 
- A reminder to catch errors early at RTL and simulation stages. 

## Why frontend CI matters?
Frontend CI matters because it automatically verifies RTL correctness and prevents regressions as the design changes.

Frontend CI:
- Runs checks whenever RTL changes
- Detects functional, lint, and build failures early
- Prevents broken RTL from propagating downstream
- Saves massive compute and engineering time

Essentially discovers RTL bugs early so that they won't be expensive fixes down the line. 