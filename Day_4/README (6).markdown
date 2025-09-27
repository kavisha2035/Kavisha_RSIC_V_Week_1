# RTL Workshop: Day 4 - Gate-Level Simulation, Blocking vs. Non-Blocking, and Synthesis-Simulation Mismatch

Welcome to **Day 4 of the RTL Workshop!**  
This session introduces three key concepts in digital design:

- **Gate-Level Simulation (GLS)**  
- **Synthesis vs. Simulation Mismatches**  
- **Blocking vs. Non-Blocking Assignments in Verilog**  

We’ll cover the theory and provide **hands-on labs** to reinforce your understanding.

---

## 📑 Table of Contents
1. [Gate-Level Simulation (GLS)](#1-gate-level-simulation-gls)  
2. [Synthesis-Simulation Mismatch](#2-synthesis-simulation-mismatch)  
3. [Blocking vs. Non-Blocking Assignments](#3-blocking-vs-non-blocking-assignments)  
   - [3.1 Blocking (`=`)](#31-blocking)  
   - [3.2 Non-Blocking (`<=`)](#32-non-blocking)  
   - [3.3 Comparison](#33-comparison)  
4. [Labs](#4-labs)  
5. [Summary](#5-summary)  

---

## 1. Gate-Level Simulation (GLS)

**Gate-Level Simulation (GLS)** involves simulating a **synthesized netlist** (gates, flip-flops, muxes) rather than the RTL description to validate correct behavior post-synthesis.

### 🔹 Why GLS?
- Ensures RTL and synthesized netlist behave identically.  
- Verifies **timing behavior** using SDF (Standard Delay Format) annotations.  
- Validates scan chains, DFT logic, and inserted test points.  
- Catches synthesis-related bugs before physical design.  

### 🔹 When is GLS Done?
- **Post-synthesis**: After converting RTL to gates.  
- **Pre-tape-out**: To confirm design intent is preserved.  

### 🔹 Types of GLS
- **Functional GLS**: Ignores delays (zero-delay or unit-delay simulation).  
- **Timing GLS**: Includes real timing information via SDF annotations.  

---

## 2. Synthesis-Simulation Mismatch

A **synthesis-simulation mismatch** occurs when RTL simulation results differ from gate-level simulation or actual hardware behavior.

### ⚠️ Common Causes
1. **Non-synthesizable constructs**: Using `#delays`, `initial` blocks, or `$display`.  
2. **Incomplete coding**: Missing sensitivity lists, latch inference, or ambiguous conditions.  
3. **Tool differences**: Simulators and synthesizers may interpret ambiguous code differently.  

👉 **Golden Rule**: Write **synthesizable, deterministic RTL** with complete sensitivity lists and proper resets.

---

## 3. Blocking vs. Non-Blocking Assignments

Verilog supports two types of assignments inside procedural blocks:

### 3.1 Blocking (`=`)
- Executes **immediately** in sequential order.  
- Suitable for **combinational logic**.  

**Example**:
```verilog
always @(*) begin
  y = a & b;
end
```

### 3.2 Non-Blocking (`<=`)
- Updates are **scheduled** and applied at the end of the time step.  
- Essential for **sequential logic** (e.g., flip-flops).  

**Example**:
```verilog
always @(posedge clk) begin
  q <= d;
end
```

### 3.3 Comparison
| Aspect            | Blocking (`=`)               | Non-Blocking (`<=`)         |
|-------------------|-----------------------------|-----------------------------|
| Execution         | Immediate, sequential       | Deferred, parallel updates  |
| Usage             | Combinational logic         | Sequential (clocked) logic  |
| Updates           | In code order               | At end of time step         |
| Typical Hardware  | Gates, muxes                | Flip-flops                  |

---

## 4. Labs

### 🔹 Lab 1: Ternary Operator MUX
```verilog
module ternary_operator_mux (input i0, input i1, input sel, output y);
  assign y = sel ? i1 : i0;
endmodule
```
- **Function**: Outputs `y = i1` if `sel=1`, else `y = i0`.

### 🔹 Lab 2: Synthesizing the MUX
Run with Yosys:
```bash
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### 🔹 Lab 3: GLS of MUX
Run GLS with primitives and testbench:
```bash
iverilog /path/to/primitives.v /path/to/sky130_fd_sc_hd.v ternary_operator_mux.v tb_mux.v
./a.out
```

### 🔹 Lab 4: Bad MUX Example
```verilog
module bad_mux (input i0, input i1, input sel, output reg y);
  always @(sel) begin
    if (sel)
      y <= i1;
    else
      y <= i0;
  end
endmodule
```
⚠️ **Problems**:
- Missing `i0` and `i1` in sensitivity list.  
- Uses non-blocking (`<=`) for combinational logic.  

✅ **Corrected**:
```verilog
module good_mux (input i0, input i1, input sel, output reg y);
  always @(*) begin
    if (sel)
      y = i1;
    else
      y = i0;
  end
endmodule
```

### 🔹 Lab 5: GLS of Bad MUX
- Simulate `bad_mux`.  
- Expect mismatches due to incorrect sensitivity list and assignment style.

### 🔹 Lab 6: Blocking Assignment Caveat
```verilog
module blocking_caveat (input a, input b, input c, output reg d);
  reg x;
  always @(*) begin
    d = x & c;  // Uses old value of x
    x = a | b;  // x updated after
  end
endmodule
```
⚠️ **Issue**: `d` uses old value of `x` due to blocking assignment order.  

✅ **Fix**:
```verilog
module blocking_caveat_fixed (input a, input b, input c, output reg d);
  reg x;
  always @(*) begin
    x = a | b;
    d = x & c;
  end
endmodule
```

### 🔹 Lab 7: Synthesizing Blocking Caveat
- Synthesize the corrected design and verify netlist simplifications.

---

## 5. Summary
- **GLS** validates post-synthesis netlist behavior (functional and timing).  
- **Synthesis-Simulation Mismatch** arises from non-synthesizable or ambiguous RTL.  
- **Blocking (`=`)**: Use for combinational logic.  
- **Non-Blocking (`<=`)**: Use for sequential logic.  
- **Labs**: Demonstrated correct MUX coding, pitfalls in bad MUXes, and blocking assignment issues.  

✅ **Best Practice**: Always simulate both RTL and netlist, and resolve synthesis warnings early to avoid issues in GLS or silicon.