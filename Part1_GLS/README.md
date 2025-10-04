

# VSD Hardware Design Program: VSDBabySoC Post-Synthesis Simulation

  

Post-synthesis simulation is critical for validating both **functionality and timing** after RTL code has been synthesized into standard cells.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Pre-Synthesis vs Post-Synthesis Simulation](#pre-synthesis-vs-post-synthesis-simulation)
3. [Synthesis Flow](#synthesis-flow)
   - [Step 1: Load RTL Modules](#step-1-load-rtl-modules)
   - [Step 2: Load Liberty Files](#step-2-load-liberty-files)
   - [Step 3: Synthesize the Design](#step-3-synthesize-the-design)
   - [Step 4: Map Flip-Flops to Standard Cells](#step-4-map-flip-flops-to-standard-cells)
   - [Step 5: Optimization & Technology Mapping](#step-5-optimization--technology-mapping)
   - [Step 6: Clean-Up & Renaming](#step-6-clean-up--renaming)
   - [Step 7: Check Statistics](#step-7-check-statistics)
   - [Step 8: Write Synthesized Netlist](#step-8-write-synthesized-netlist)
4. [Post-Synthesis Simulation](#post-synthesis-simulation)
   - [Step 1: Compile Testbench](#step-1-compile-testbench)
   - [Step 2: Navigate to Simulation Directory](#step-2-navigate-to-simulation-directory)
   - [Step 3: Run Simulation](#step-3-run-simulation)
   - [Step 4: View Waveforms in GTKWave](#step-4-view-waveforms-in-gtkwave)
5. [Comparing Pre- and Post-Synthesis Outputs](#comparing-pre--and-post-synthesis-outputs)

---

## Introduction

Post-synthesis simulation validates that the **synthesized gate-level netlist** behaves identically to the original RTL, both logically and in terms of timing.  

It identifies issues such as:

- Unintended latches or glitches
- Race conditions due to gate delays
- Timing violations

This ensures a robust design before hardware implementation.

---

## Pre-Synthesis vs Post-Synthesis Simulation

- **Pre-Synthesis Simulation**  
  Focuses on **logical correctness** of RTL. Fast and ideal for detecting functional errors early.

- **Post-Synthesis Simulation**  
  Validates **logic and timing** of the synthesized netlist, including gate delays and technology-specific constraints.

Performing both ensures the final design is functionally correct and meets timing requirements.

---

## Synthesis Flow

All commands assume the working directory:  

```bash
devichinni20@devi-VirtualBox:~Desktop/vsdflow/VLSI/VSDBabySoC/
````

### Step 1: Load RTL Modules

1. Launch Yosys:

```bash
yosys
```

2. Read top-level module:

```yosys
read_verilog src/module/vsdbabysoc.v
```
<img width="1477" height="761" alt="Screenshot from 2025-10-03 19-57-42" src="https://github.com/user-attachments/assets/a10f246a-8bac-4428-a8a3-74b4751ccda4" />



3. Copy include files:

```bash
cp -r src/include/sp_verilog.vh .
cp -r src/include/sandpiper.vh .
cp -r src/include/sandpiper_gen.vh .
```
<img width="1464" height="708" alt="Screenshot from 2025-10-03 19-56-08 (Copy)" src="https://github.com/user-attachments/assets/1163a13e-9fa9-497d-b50c-f85a0338c021" />



4. Read dependent modules:

```yosys
read_verilog -I src/include/ src/module/rvmyth.v
read_verilog -I src/include/ src/module/clk_gate.v
```

> ⚠️ Ensure include files are in the working directory to avoid parsing errors.

---

### Step 2: Load Liberty Files

```yosys
read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

---
<img width="1433" height="519" alt="Screenshot from 2025-10-03 19-58-09" src="https://github.com/user-attachments/assets/22f7818c-910d-4dd7-afd0-34e74e454f1a" />

### Step 3: Synthesize the Design

```yosys
synth -top vsdbabysoc
```

---

### Step 4: Map Flip-Flops to Standard Cells

```yosys
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

---
<img width="665" height="809" alt="Screenshot from 2025-10-03 20-00-10" src="https://github.com/user-attachments/assets/415405ed-8d33-4ce3-a6e0-73d8f2cabffe" />

### Step 5: Optimization & Technology Mapping

```yosys
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```
<img width="690" height="846" alt="Screenshot from 2025-10-03 20-06-12" src="https://github.com/user-attachments/assets/8d922d72-1635-4e94-b046-65c973cf3107" />
<img width="1718" height="830" alt="Screenshot from 2025-10-03 20-06-39" src="https://github.com/user-attachments/assets/e9b7e3fc-211a-4f94-87e5-204224873425" />



**Optimization Steps:**

| Command      | Purpose                                                    |
| ------------ | ---------------------------------------------------------- |
| strash       | Reduce logic redundancy                                    |
| scorr        | Sequential sweeping to remove redundant logic              |
| ifraig       | Equivalence checking & incremental optimization            |
| retime;{D}   | Move registers to optimize timing                          |
| dch,-f       | Delay-aware combinational optimization                     |
| map,-M,1,{D} | Map logic to standard cells minimizing area & timing-aware |

---
<img width="1920" height="923" alt="Screenshot from 2025-10-03 20-06-55" src="https://github.com/user-attachments/assets/b567463f-c5d8-4ebe-9012-e444e2122c89" />



### Step 6: Clean-Up & Renaming

```yosys
flatten
setundef -zero
clean -purge
rename -enumerate
```

| Command           | Purpose                                       |
| ----------------- | --------------------------------------------- |
| flatten           | Flatten hierarchy into a single-level netlist |
| setundef -zero    | Replace undefined values (`x`) with `0`       |
| clean -purge      | Remove unused wires, cells, and modules       |
| rename -enumerate | Unique numbering for internal wires and cells |

---
<img width="742" height="382" alt="Screenshot from 2025-10-03 20-08-45" src="https://github.com/user-attachments/assets/8720d7d3-6916-44ae-951f-8e0194150a51" />



### Step 7: Check Statistics

```yosys
stat
```
<img width="1920" height="923" alt="Screenshot from 2025-10-03 20-13-19" src="https://github.com/user-attachments/assets/23caa02a-168f-418c-867d-014e27a28d3d" />
<img width="1920" height="923" alt="Screenshot from 2025-10-03 20-13-29" src="https://github.com/user-attachments/assets/fd1f0230-0c1c-460a-94d9-5be16eda1691" />



> Review the number of cells, wires, memories, and other statistics.

---

### Step 8: Write Synthesized Netlist

```yosys
write_verilog -noattr output/post_synth_sim/vsdbabysoc.synth.v
```

---
<img width="1675" height="249" alt="Screenshot from 2025-10-03 20-30-45" src="https://github.com/user-attachments/assets/ebfb98e2-bbbb-40e0-ae96-ca5aa86c389f" />


## Post-Synthesis Simulation

### Step 1: Compile Testbench

1. Copy standard cell and primitive models:

```bash
cp -r ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v src/module/
cp -r ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v src/module/
```

2. Copy synthesized netlist to module directory:

```bash
cp -r output/post_synth_sim/vsdbabysoc.synth.v src/module/
```

3. Compile testbench:

```bash
iverilog -o output/post_synth_sim/post_synth_sim.out \
-DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
-I src/include -I src/module src/module/testbench.v
```

> ⚠️ Fix any syntax errors in `sky130_fd_sc_hd.v` if encountered, e.g.:

```verilog
`endif // SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V
```

---
<img width="1920" height="923" alt="Screenshot from 2025-10-04 01-21-42" src="https://github.com/user-attachments/assets/06f09d79-dfc6-4578-80be-7de522c93a40" />


### Step 2: Navigate to Simulation Directory

```bash
cd output/post_synth_sim/
```

---

### Step 3: Run Simulation

```bash
./post_synth_sim.out
```

---

### Step 4: View Waveforms in GTKWave

```bash
gtkwave post_synth_sim.vcd
```

---
<img width="1920" height="923" alt="Screenshot from 2025-10-04 01-31-27" src="https://github.com/user-attachments/assets/122b2eba-cdb6-4b1c-b698-c2256827aa99" />



## Comparing Pre- and Post-Synthesis Outputs

To verify **functional equivalence**, compare the GTKWave outputs from pre-synthesis and post-synthesis simulations.

<img width="1920" height="923" alt="VirtualBox_opensource_tool_ubuntu_02_10_2025_16_06_27" src="https://github.com/user-attachments/assets/3545e3b9-490c-442f-aecd-89c0a7a291ed" />
<img width="1920" height="923" alt="Screenshot from 2025-10-04 01-41-19" src="https://github.com/user-attachments/assets/14ce6e7a-c6ae-496a-81a8-4ead21496f8e" />





✅ Matching waveforms confirm that the synthesis process **preserves functionality**.

---

---

## 🏁 **Conclusion: **

### ✅ **Key Outcome**

- **The SoC is functionally correct**
  - The CPU core (**RVMYTH**), **PLL**, and **DAC** interact seamlessly.  
  - Clock generation, signal flow, and synchronization operate as intended.

- **Synthesis preserved functionality**
  - Logic was successfully mapped to **Sky130 standard cells**.  
  - Optimization maintained behavior without altering logical correctness.

- **The SoC is now gate-level ready**
  - The synthesized netlist is ready for:
    - **Static Timing Analysis (STA)**
    - **Place & Route**
    - **Final GDSII fabrication**

- **The design flow is fully verified end-to-end**
  - Demonstrated a **complete open-source ASIC design flow**:  
    **RTL → Synthesis → Post-Synthesis Simulation**

---

### 💡 **In Simpler Words**

We have effectively **built and verified a miniature open-source RISC-V SoC**, integrating:

- A **RISC-V CPU core (RVMYTH)**  
- A **PLL** for stable clock generation  
- A **DAC** for digital-to-analog signal conversion  

All using **open-source EDA tools** like **Yosys**, **Icarus Verilog**, and **GTKWave** with **Sky130 technology**.

---

### 🧩 **Final Verification Insight**

By observing **matching pre- and post-synthesis waveforms**, you have proven that:

> My design is **functionally stable**, **logically sound**, and **synthesis-ready** for real-world hardware realization.

---


```
