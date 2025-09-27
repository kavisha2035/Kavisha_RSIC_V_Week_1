# RTL Workshop: Day 3 - Combinational and Sequential Optimization

Welcome to **Day 3** of the RTL Workshop!  
This session focuses on **optimization techniques for combinational and sequential circuits** to improve performance, reduce resource usage, and simplify logic.

---

## 📑 Table of Contents
1. [Constant Propagation](#1-constant-propagation)  
2. [State Optimization](#2-state-optimization)  
3. [Cloning](#3-cloning)  
4. [Retiming](#4-retiming)  
5. [Labs on Optimization](#5-labs-on-optimization)  
   - [Lab 1](#lab-1)  
   - [Lab 2](#lab-2)  
   - [Lab 3](#lab-3)  
   - [Lab 4](#lab-4)  
   - [Lab 5](#lab-5)  
   - [Lab 6](#lab-6)  
6. [Summary](#6-summary)  

---

## 1. Constant Propagation
Constant propagation substitutes **constant values** directly into the logic, enabling the removal of redundant logic.

🔹 **How it works:**  
- Identify signals with constant assignments.  
- Replace them during synthesis.  
- Simplify the resulting logic equations.  

🔹 **Benefits:**  
- Smaller circuits (fewer gates).  
- Reduced propagation delay.  
- Lower power consumption.  

**Example:**  
```verilog
module opt_check (input a, input b, output y);
  assign y = a ? b : 0;
endmodule
```
- If `a=1`, then `y=b`.  
- If `a=0`, then `y=0`.  
- Simplifies to: `y = a & b`.

---

## 2. State Optimization
Finite State Machines (FSMs) often have **redundant or inefficient state encodings**. State optimization improves FSM efficiency by:  

- **State Reduction:** Merging equivalent states.  
- **State Encoding:** Assigning binary codes to minimize next-state and output logic.  
- **Logic Minimization:** Applying Boolean algebra or automated tools.  
- **Power Optimization:** Using techniques like clock gating to save dynamic power.  

---

## 3. Cloning
Cloning duplicates logic gates or submodules to **reduce fanout, balance loads, or shorten wire delays**.  

🔹 **Benefits:**  
- Shortens critical paths.  
- Improves signal integrity.  
- Enhances timing closure through load balancing.  

---

## 4. Retiming
Retiming repositions registers (flip-flops) **without altering design functionality** to optimize performance.  

🔹 **Steps:**  
1. Represent circuit as a **directed graph**.  
2. Move registers across combinational logic.  
3. Ensure **functional equivalence** is preserved.  
4. Optimize for **minimum clock period** or **reduced power**.  

---

## 5. Labs on Optimization

### Lab 1
```verilog
module opt_check (input a, input b, output y);
  assign y = a ? b : 0;
endmodule
```
- **Description**: If `a=1`, then `y=b`; if `a=0`, then `y=0`.  
- **Simplified**: `y = a & b`.  
- **Synthesis Command**: Run `opt_clean -purge` to observe optimization.

### Lab 2
```verilog
module opt_check2 (input a, input b, output y);
  assign y = a ? 1 : b;
endmodule
```
- **Description**: Acts as a 2:1 multiplexer.  
  - If `a=1`, then `y=1`.  
  - If `a=0`, then `y=b`.  

### Lab 3
```verilog
module opt_check3 (input a, input b, output y);
  assign y = a ? 1 : b;
endmodule
```
- **Description**: Equivalent to Lab 2.  
- **Simplified**: Optimizes to `y = a | b`.

### Lab 4
```verilog
module opt_check4 (input a, input b, input c, output y);
  assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```
- **Description**: Simplified logic: `y = a ? c : ~c`.  
- **Note**: Synthesis tool removes unnecessary conditions.

### Lab 5
```verilog
module dff_const1(input clk, input reset, output reg q);
  always @(posedge clk, posedge reset) begin
    if (reset)
      q <= 1'b0;
    else
      q <= 1'b1;
  end
endmodule
```
- **Description**: DFF with async reset; loads `1` after reset.  
- **Note**: Output will always be `1` after the first clock cycle.

### Lab 6
```verilog
module dff_const2(input clk, input reset, output reg q);
  always @(posedge clk, posedge reset) begin
    if (reset)
      q <= 1'b1;
    else
      q <= 1'b1;
  end
endmodule
```
- **Description**: Output `q` is always `1`, regardless of input.  
- **Simplified**: Synthesizer optimizes this into constant logic `1`.

---

## 6. Summary
- **Constant Propagation**: Replaces constants to simplify logic.  
- **State Optimization**: Reduces redundant FSM states and optimizes encoding.  
- **Cloning**: Duplicates logic to reduce fanout and improve performance.  
- **Retiming**: Repositions registers to balance path delays.  
- **Labs**: Demonstrate synthesis optimizations using Verilog examples.

🔗 **Tip**: Continue experimenting with these labs to observe how synthesis tools simplify logic and improve timing automatically!