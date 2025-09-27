# RTL Workshop: Day 2 - Timing Libraries, Synthesis Styles, and Flip-Flop Coding

Welcome to **Day 2 of the RTL Workshop**.  
In this session, we dive deeper into three key aspects of RTL design:

1. Understanding timing libraries (`.lib` files) from the SKY130 PDK.  
2. Comparing **hierarchical vs. flat synthesis** approaches.  
3. Exploring **efficient flip-flop coding practices** in Verilog.  

---

## 📂 Timing Libraries

### SKY130 PDK at a Glance
The **SKY130 Process Design Kit (PDK)** is an open-source 130nm CMOS technology stack from **SkyWater Technology**.  
It provides everything required for chip design, including timing, power, and process variation data, enabling **accurate synthesis and analysis**.

### Decoding `tt_025C_1v80`
The file `sky130_fd_sc_hd__tt_025C_1v80.lib` encodes process, temperature, and voltage information:

- **tt**: "Typical-Typical" process corner  
- **025C**: Characterization at 25 °C  
- **1v80**: Core supply voltage = 1.8 V  

This library models a standard scenario for timing analysis.

### Viewing the `.lib` File
You can open and inspect it with a text editor:

```bash
sudo apt install gedit   # install editor (optional)
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

Inside, you’ll find definitions of standard cells (gates, flops, latches), their delays, setup/hold times, and power data.

---

## 🏗️ Synthesis Approaches

### Hierarchical Synthesis
- **What it is**: Keeps the RTL module boundaries intact during synthesis.  
- **How it works**: Tools like Yosys analyze and synthesize each module separately.  
- **Pros**:  
  - Faster compile time on large designs.  
  - Easier debugging (can trace signals to RTL hierarchy).  
  - Good for modular projects.  
- **Cons**:  
  - Limited cross-module optimization.  
  - Reporting can be less straightforward.  

### Flattened Synthesis
- **What it is**: Removes hierarchy, generating a single flat netlist.  
- **How it works**: Using `flatten` in Yosys, all modules are merged into one.  
- **Pros**:  
  - Better whole-design optimization.  
  - Single unified netlist (sometimes easier for place & route).  
- **Cons**:  
  - Slower runtime and higher memory use on big projects.  
  - Debugging is harder without hierarchy.  

### Quick Comparison
| Aspect              | Hierarchical                     | Flattened                       |
|---------------------|----------------------------------|---------------------------------|
| Module Structure    | Preserved                       | Removed                         |
| Optimization Scope  | Local (per module)              | Global (whole design)           |
| Runtime (Large Design) | Faster                       | Slower                          |
| Debugging           | Easier                          | Harder                          |
| Output Netlist      | Modular                         | One big block                   |

👉 **Rule of thumb**: Use hierarchical for modular design/debugging, and flattened when maximum optimization is required.

---

## 🔄 Flip-Flop Coding Styles

Flip-flops are the fundamental sequential storage elements in RTL. Below are standard Verilog templates for common types:

### Asynchronous Reset D-FF
```verilog
module dff_async_reset(input clk, input rst, input d, output reg q);
  always @(posedge clk or posedge rst)
    if (rst)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
- Reset takes priority over the clock.  
- Output goes to 0 immediately when reset is active.

### Asynchronous Set D-FF
```verilog
module dff_async_set(input clk, input set, input d, output reg q);
  always @(posedge clk or posedge set)
    if (set)
      q <= 1'b1;
    else
      q <= d;
endmodule
```
- Forces output to 1 when set is active, regardless of clock.

### Synchronous Reset D-FF
```verilog
module dff_sync_reset(input clk, input rst, input d, output reg q);
  always @(posedge clk)
    if (rst)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
- Reset only works on the rising edge of the clock.

---

## ⚙️ Simulation & Synthesis Flow

### Simulating with Icarus Verilog
1. **Compile**:
   ```bash
   iverilog dff_async_reset.v tb_dff_async_reset.v
   ```
2. **Run**:
   ```bash
   ./a.out
   ```
3. **View waveform**:
   ```bash
   gtkwave tb_dff_async_reset.vcd
   ```

### Synthesizing with Yosys
1. **Start Yosys**:
   ```bash
   yosys
   ```
2. **Load the standard cell library**:
   ```bash
   read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
3. **Read your Verilog**:
   ```bash
   read_verilog dff_async_reset.v
   ```
4. **Run synthesis**:
   ```bash
   synth -top dff_async_reset
   ```
5. **Map to library flip-flops**:
   ```bash
   dfflibmap -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
6. **Technology map**:
   ```bash
   abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
7. **View netlist**:
   ```bash
   show
   ```

---

## ✅ Summary
In this session, we explored:
- The role of timing libraries in synthesis and analysis.  
- Trade-offs between hierarchical vs. flat synthesis.  
- Standard flip-flop coding styles for reset/set functionality.  
- A basic simulation & synthesis workflow with Icarus Verilog and Yosys.  

This forms the foundation for timing analysis, technology mapping, and building reliable RTL modules.
