# RTL Workshop: Day 5 - Optimization in Synthesis

Welcome to **Day 5** of the RTL Workshop!  
This session focuses on **optimization in Verilog synthesis**, exploring how coding style impacts hardware implementation.  
We cover `if-else` statements, `case` statements, `for` loops, and `generate` blocks, along with **inferred latches**, their issues, and how to avoid them.  
Hands-on labs are included to reinforce these concepts.

---

## 📑 Contents
1. [If-Else Statements in Verilog](#1-if-else-statements-in-verilog)  
2. [Inferred Latches in Verilog](#2-inferred-latches-in-verilog)  
3. [Labs for If-Else and Case Statements](#3-labs-for-if-else-and-case-statements)  
   - [Lab 1: Incomplete If Statement](#lab-1-incomplete-if-statement)  
   - [Lab 2: Synthesis Result of Lab 1](#lab-2-synthesis-result-of-lab-1)  
   - [Lab 3: Nested If-Else](#lab-3-nested-if-else)  
   - [Lab 4: Synthesis Result of Lab 3](#lab-4-synthesis-result-of-lab-3)  
   - [Lab 5: Complete Case Statement](#lab-5-complete-case-statement)  
   - [Lab 6: Synthesis Result of Lab 5](#lab-6-synthesis-result-of-lab-5)  
   - [Lab 7: Incomplete Case Handling](#lab-7-incomplete-case-handling)  
   - [Lab 8: Partial Assignments in Case](#lab-8-partial-assignments-in-case)  
4. [For Loops in Verilog](#4-for-loops-in-verilog)  
5. [Generate Blocks in Verilog](#5-generate-blocks-in-verilog)  
6. [What is an RCA (Ripple Carry Adder)?](#6-what-is-an-rca-ripple-carry-adder)  
7. [Labs on Loops and Generate Blocks](#7-labs-on-loops-and-generate-blocks)  
   - [Lab 9: 4-to-1 MUX Using For Loop](#lab-9-4-to-1-mux-using-for-loop)  
   - [Lab 10: 8-to-1 Demux Using Case](#lab-10-8-to-1-demux-using-case)  
   - [Lab 11: 8-to-1 Demux Using For Loop](#lab-11-8-to-1-demux-using-for-loop)  
   - [Lab 12: 8-bit Ripple Carry Adder with Generate Block](#lab-12-8-bit-ripple-carry-adder-with-generate-block)  
8. [Summary](#8-summary)  

---

## 1. If-Else Statements in Verilog
- Used for **behavioral modeling** to implement conditional execution.  
- Typically written inside `always` or `initial` blocks.  

### Syntax
```verilog
if (condition) begin
   // statements
end else begin
   // statements
end
```

---

## 2. Inferred Latches in Verilog
- A latch is **unintentionally created** when a signal is not assigned in all branches of combinational logic.  
- This can lead to unexpected hardware behavior and should be avoided.

### Example (Latch Inferred)
```verilog
always @(a, b, sel) begin
    if (sel)
        y = a;   // y not assigned when sel=0 → Latch inferred
end
```

### Solution (Add Default/Else)
```verilog
always @(a, b, sel) begin
    case (sel)
        1'b1: y = a;
        default: y = 1'b0;
    endcase
end
```

**Rule**: Always assign outputs in all possible paths to avoid latches.

---

## 3. Labs for If-Else and Case Statements

### Lab 1: Incomplete If Statement
```verilog
module incomp_if (input i0, input i1, input i2, output reg y);
always @(*) begin
    if (i0)
        y <= i1;
end
endmodule
```
⚠️ Leads to latch inference due to incomplete assignment.

### Lab 2: Synthesis Result of Lab 1
- Synthesize with `synth_tool` → `incomp_synth` (shows latch insertion).

### Lab 3: Nested If-Else
```verilog
module incomp_if2 (input i0, input i1, input i2, input i3, output reg y);
always @(*) begin
    if (i0)
        y <= i1;
    else if (i2)
        y <= i3;
end
endmodule
```
⚠️ Still incomplete, as `y` is not assigned in all cases.

### Lab 4: Synthesis Result of Lab 3
- Synthesize with `synth_tool` → `incomp2synth`.

### Lab 5: Complete Case Statement
```verilog
module comp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        default: y = i2;
    endcase
end
endmodule
```
✅ No latch inferred due to complete assignment.

### Lab 6: Synthesis Result of Lab 5
- Synthesize with `synth_tool` → `compcase_synth`.

### Lab 7: Incomplete Case Handling
```verilog
module bad_case (input i0, i1, i2, i3, input [1:0] sel, output reg y);
always @(*) begin
    case(sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3;  // Wildcard → risk of missing conditions
    endcase
end
endmodule
```
⚠️ Incomplete handling risks latch inference.

### Lab 8: Partial Assignments in Case
```verilog
module partial_case_assign (
    input i0, input i1, input i2,
    input [1:0] sel,
    output reg y, output reg x
);
always @(*) begin
    case(sel)
        2'b00: begin
            y = i0;
            x = i2;
        end
        2'b01: y = i1;      // x not assigned here → latch
        default: begin
            x = i1;
            y = i2;
        end
    endcase
end
endmodule
```

---

## 4. For Loops in Verilog
- Used for **repetition** in procedural blocks.  
- Must have a **fixed iteration count** for synthesis.  

### Example: 4-to-1 MUX
```verilog
always @(data, sel) begin
    y = 1'b0;
    for (i = 0; i < 4; i = i + 1) begin
        if (i == sel)
            y = data[i];
    end
end
```

---

## 5. Generate Blocks in Verilog
- Used for **hardware replication** at compile time.  
- Often combined with `genvar` for structural modeling.  

### Example
```verilog
genvar i;
generate
    for (i = 0; i < 4; i = i + 1) begin : gen_loop
        and_gate and_inst (.a(in[i]), .b(in[i+1]), .y(out[i]));
    end
endgenerate
```

---

## 6. What is an RCA (Ripple Carry Adder)?
- Adds two n-bit numbers using **n full adders**.  
- Carry propagates through each stage (ripple effect).  
- Structure: `FA0 → FA1 → FA2 → ... → FAn-1`.

---

## 7. Labs on Loops and Generate Blocks

### Lab 9: 4-to-1 MUX Using For Loop
- File: `mux_generate.v`

### Lab 10: 8-to-1 Demux Using Case
- File: `demux_case.v`

### Lab 11: 8-to-1 Demux Using For Loop
- File: `demux_generate.v`

### Lab 12: 8-bit Ripple Carry Adder with Generate Block
- Files: `rca.v`, `fa.v`

---

## 8. Summary
- Use **complete if-else/case statements** to avoid inferred latches.  
- Include **default assignments** in combinational logic.  
- **For loops** enable concise behavioral models.  
- **Generate blocks** are powerful for scalable hardware design.  
- **Ripple Carry Adder (RCA)** is a fundamental design used as a baseline in synthesis.

🔑 **Tip**: Always verify synthesis results and ensure all signals are assigned in combinational logic to prevent latches.
