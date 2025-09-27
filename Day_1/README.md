# Verilog RTL Design & Synthesis Workshop - Day 1

Welcome to **Day 1** of the RTL Workshop! This repository contains resources and code for learning **Verilog HDL**, simulating designs with **Icarus Verilog (iverilog)**, visualizing results with **GTKWave**, and performing logic synthesis using **Yosys** with the **Sky130 open-source PDK**.

## 📖 Table of Contents
1. [Simulator, Design, and Testbench – What are they?](#simulator-design-and-testbench--what-are-they)
2. [Getting Started with Icarus Verilog](#getting-started-with-icarus-verilog)
3. [Lab: Simulating a 2-to-1 Multiplexer](#lab-simulating-a-2-to-1-multiplexer)
4. [Understanding the Verilog Code](#understanding-the-verilog-code)
5. [Introduction to Yosys and Gate Libraries](#introduction-to-yosys-and-gate-libraries)
6. [Lab: Synthesizing with Yosys](#lab-synthesizing-with-yosys)
7. [Summary](#summary)
8. [Repository Structure](#repository-structure)
9. [Prerequisites](#prerequisites)
10. [Contributing](#contributing)
11. [License](#license)

## Simulator, Design, and Testbench – What are they?

- **Simulator** → A software tool that executes your HDL design in a “virtual” environment. It helps you verify correctness before burning it into silicon.
- **Design** → The Verilog code that describes your logic (e.g., multiplexer, adder, flip-flop).
- **Testbench** → A special piece of code that applies inputs to your design and observes the outputs. Think of it as a “virtual lab setup” for your circuit.

📌 The design + testbench together make simulation possible.

## Getting Started with Icarus Verilog

**Icarus Verilog (iverilog)** is a widely used open-source Verilog simulator. The general flow is:

1. Write your **design** (`design.v`)
2. Write a **testbench** (`tb_design.v`)
3. Compile both with `iverilog`
4. Run the simulation → produces `dump.vcd`
5. Open `dump.vcd` in **GTKWave** to see waveforms

## Lab: Simulating a 2-to-1 Multiplexer

Let’s build and test a simple **2:1 MUX**.

### Step 1: Install Tools (Ubuntu/WSL)
```bash
sudo apt update
sudo apt install iverilog gtkwave -y
```

### Step 2: Verilog Code
Below is the Verilog code for the 2-to-1 multiplexer and its testbench.

```verilog
module mux (
    input wire a, b, sel,
    output reg y
);
    always @(*) begin
        if (sel)
            y = b;
        else
            y = a;
    end
endmodule

module tb_mux;
    reg a, b, sel;
    wire y;

    mux uut (.a(a), .b(b), .sel(sel), .y(y));

    initial begin
        $dumpfile("mux.vcd");
        $dumpvars(0, tb_mux);

        a=0; b=1; sel=0; #10;
        a=0; b=1; sel=1; #10;
        a=1; b=0; sel=0; #10;
        a=1; b=0; sel=1; #10;

        $finish;
    end
endmodule
```

### Step 3: Compile and Simulate
```bash
iverilog -o mux_sim mux.v tb_mux.v
vvp mux_sim
```

### Step 4: Visualize Waveforms
```bash
gtkwave mux.vcd
```

## Understanding the Verilog Code

- **Inputs**: `a`, `b` → data inputs; `sel` → control input
- **Output**: `y` → selected output
- **Logic**: If `sel=0`, output is `a`; if `sel=1`, output is `b`.

This is the simplest form of a multiplexer, and a great example to start RTL design.

## Introduction to Yosys and Gate Libraries

**Yosys** is an open-source tool that converts your Verilog RTL into a gate-level netlist using a technology library (like Sky130).

- **Synthesis**: Turns high-level HDL into actual logic gates
- **Optimization**: Improves design for area, speed, or power
- **Technology Mapping**: Matches your logic to cells from a specific library (`.lib` file)

### Why Multiple Gate Variants Exist?
In `.lib` files, you’ll find many versions of each gate (e.g., `AND2_X1`, `AND2_X2`):
- **Speed**: Some are optimized for timing
- **Power**: Some consume less power
- **Drive Strength**: Stronger cells can drive heavier loads
- **Area**: Compact cells save chip space

## Lab: Synthesizing with Yosys

Let’s synthesize the same multiplexer.

### Step 1: Install Yosys
```bash
sudo apt install yosys -y
```

### Step 2: Launch Yosys
```bash
yosys
```

### Step 3: Run Commands in Yosys
```tcl
# Load Sky130 standard cell library
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib

# Read design
read_verilog mux.v

# Run synthesis
synth -top mux

# Map flip-flops and logic to library cells
dfflibmap -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib

# Write synthesized netlist
write_verilog mux_netlist.v

# Report synthesis stats
stat

# Visualize the netlist
show
```

✅ This opens a schematic view of your gate-level design.

### Step 4: Obtain Sky130 PDK
Download the Sky130 standard cell library (`sky130_fd_sc_hd__tt_025C_1v80.lib`) from the [SkyWater PDK GitHub](https://github.com/google/skywater-pdk) and place it in your working directory.

## Summary

Today, you:
- Understood what simulators, designs, and testbenches are
- Learned the `iverilog` + `GTKWave` flow for RTL simulation
- Built and simulated a 2:1 multiplexer
- Got introduced to Yosys and gate libraries
- Performed synthesis and saw how RTL maps to actual hardware gates

🚀 You’ve successfully completed Day 1! Next, we’ll dive deeper into timing libraries and flip-flop design.

## Repository Structure

```plaintext
├── mux.v                # 2-to-1 MUX Verilog design
├── tb_mux.v             # Testbench for MUX simulation
├── mux_netlist.v        # Synthesized netlist (generated)
├── mux.vcd              # Waveform file (generated)
└── README.md            # This file
```

## Prerequisites

- **Operating System**: Ubuntu or WSL (Windows Subsystem for Linux).
- Basic familiarity with command-line interfaces.
- A GitHub account to clone and contribute to this repository.
- Sky130 PDK library for synthesis (see [SkyWater PDK GitHub](https://github.com/google/skywater-pdk)).

## Contributing

Feel free to fork this repository, experiment with the code, and submit pull requests with improvements or additional test cases.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
