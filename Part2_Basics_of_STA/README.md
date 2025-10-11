# 📘 Table of Contents

1. **Introduction**
2. **Timing Paths** – start/end points, types (in→reg, reg→reg, etc.)
3. **Setup & Hold Checks** – setup, hold, slack
4. **Types of Analysis** – reg2reg, in2reg, reg2out, in2out, clock gating, recovery/removal, latch
5. **Additional Analysis** – slew/transition, load, clock (skew, pulse width)
6. **Reg2Reg Setup Timing** – single clock, skew, GBA/PBA, pin-node, setup condition
7. **Flip-Flop Timing** – launch/capture, clk→Q, setup/hold, jitter, uncertainty, eye diagram
8. **Hold Timing Concepts** – hold condition, slack, OCV, clock push/pull, CPPR
9. **Summary** – setup vs hold, OCV, jitter, real-chip timing


## Introduction

**Static Timing Analysis** is one of the many techniques available to verify the timing of a digital design. 

An alternate approach used to verify the timing is the timing simulation which can verify the functionality as well as the timing of the design. 

The term timing analysis is used to refer to either of these two methods - static timing analysis, or the timing simulation. 

Thus, timing analysis simply refers to the analysis of the design for timing issues.

The STA is static since the analysis of the design is carried out statically and does not depend upon the data values being applied at the input pins. 

This is in contrast to simulation based timing analysis where a stimulus is applied on input signals, resulting behavior is observed and verified, then time is advanced with new input stimulus applied, and the new behavior is observed and verified and so on.



In a CMOS digital design flow, the static timing analysis can be performed at many different stages of the implementation. Figure below shows a typical flow.

![Alt Text](Images/flow.png)

**OpenSTA** is an open source static timing analyzer (STA) tool used in digital design. It is utilized to analyze and verify the timing performance of digital circuits at the gate level.

OpenSTA uses a TCL command interpreter to read the design, specify timing constraints and print timing reports.

![Alt Text](Images/opensta.png)

#### Input Files

- `*.v`  : Gate-level Verilog Netlist  
- `*.lib` : Liberty Timing Libraries  
- `*.sdc` : Synopsys Design Constraints (clocks, delays, false paths)  
- `*.sdf` : Annotated Delay File (optional)  
- `*.spef`: Parasitics (RC extraction)  
- `*.vcd` / `*.saif` : Switching Activity for Power Analysis 

#### Clock Modeling Features

- `Generated Clocks`: Derived from existing clocks  
- `Latency`: Clock propagation delay  
- `Source Latency`: Insertion delay from clock source to input  
- `Uncertainty`: Jitter or skew margins  
- `Propagated vs. Ideal`: Real vs. ideal clock network modeling  
- `Gated Clock Checks`: Verifies clocks that are enabled conditionally  
- `Multi-Frequency Clocks`: Analyzes multiple domains  

#### Timing Analysis and Reporting

OpenSTA provides a rich set of commands for analyzing timing paths, delays, and setup/hold checks:

- `report_checks`  
  Reports timing violations across specified paths using options like `-from`, `-through`, and `-to`. Supports multi-path analysis to any endpoint.

  ```tcl
  report_checks -from [get_pins U1/Q] -to [get_pins U2/D]
  ```
#### Timing Paths 

`What do you mean by Timing Paths?`
* It Refer to the logical paths a signal takes through a digital circuit from its source to its destination, including sequential and combinational elements. STA analyzes timing paths to determine their delay, setup and hold times, and other timing parameters specified in the constraints. Timing paths are categorized into combinatorial and sequential, and the critical path is the longest path in the design with the maximum operating frequency.

#### Timing Path Elements
   
Timing path elements in STA are the start point, where a signal originates, the end point, where it terminates, and the combinational logic elements, such as gates, that the signal passes through. Timing paths are traced to determine the overall delay and timing performance of the digital circuit.
<img width="1920" height="1080" alt="Screenshot (133)" src="https://github.com/user-attachments/assets/9409fe0c-59b5-49fc-b4d7-4dcc78b7838e" />




**Start Point**: Is the point where the signal originates or enters the digital circuit. This point is typically an input port of the design, where the signal is first introduced to the circuit.

The start point of a timing path can be either:

- An input port, where data enters the design, or

- The clock pin of a register, where data is launched on a clock edge.

**End Point:** Is the point where the signal terminates or leaves the digital circuit. This point is typically an output port of the design, where the signal is outputted from the circuit.

The end point of a timing path can be either:

- A register's data input pin (D pin), where data is captured by the clock edge, or

- An output port, where data must be available at a specific time.

**Combinational Logic:** Combinational logic elements are the building blocks of a digital circuit and are used to perform logic operations on the signals passing through the circuit. These elements do not store any information, and the output of a combinational logic element is solely determined by the input values at that moment.

The diagram illustrates four distinct timing paths:

Path 1: Input to Register (in2reg)

Path 2: Register to Register (reg2reg)

Path 3: Register to Output (reg2out)

Path 4: Input to Output (in2out)

![Alt Text](Images/paths.png)

#### Setup and Hold Checks

-> **What is Setup Check?**
* Is the minimum time that the data must be stable before the clock edge, and if this time is not met, it can lead to setup violations, resulting in incorrect data being stored in the sequential element. The setup check is essential to ensure correct timing behavior of a digital circuit and prevent data loss or other timing-related issues.
* The setup time of a flip-flop depends on the technology node, operating conditions, and other factors. The value of the setup time is usually provided in the logic libraries.

-> **What is Hold Check?**
* Is the minimum amount of time that the data must remain stable after the clock edge, and if this time is not met, it can lead to hold violations, resulting in incorrect data being stored in the sequential element. The hold check is necessary to prevent issues such as data corruption, metastability, and other timing-related problems in digital circuits.

#### Slack Calculation 

Setup and hold slack is defined as the difference between data required time and data arrivals time. 

>Setup slack = Data required time - Data arrival time

>Hold slack = Data arrival time - Data required time

-> **What is Data Arrival Time?**
* The time taken by the signal to travel from the start point to the end point of the digital circuit. 

-> **What is Data Required Time?** 
* The time for the clock to traverse through the clock path of the digital circuit. 

-> **What is Slack?** 
* It is difference between the desired arrival times and the actual arrival time for a signal. 
* Positive Slack indicates that the design is meeting the timing and still it can be improved. 
* Zero slack means that the design is critically working at the desired frequency. 
* Negative slack means, design has not achieved the specified timings at the specified frequency.
* Slack has to be positive always and negative slack indicates a violation in timing.

# 🧮 Types of Setup / Hold Timing Analysis

This section covers various types of **timing analyses** performed in Static Timing Analysis (STA).

---

## 🔹 Types of Setup / Hold Paths
1. **Reg → Reg**  
   (Register to Register path)

2. **In → Reg**  
   (Input port to Register path)

3. **Reg → Out**  
   (Register to Output port path)

4. **In → Out**  
   (Input port to Output port path)

5. **Clock Gating Path**  
   (Used to verify enable paths in gated clock circuits)

6. **Recovery / Removal Path**  
   (Checks for asynchronous signals like reset or clear)

7. **Data → Data Path**  
   (Used for combinational logic timing checks)

8. **Latch Path**  
   (Includes **time borrowing** in latch-based designs)

---

## ⚙️ Additional Timing Analyses

### 1. **Slew / Transition Analysis**
Analyzes **signal transition time (rise/fall time)** for both data and clock paths.

- **For Data (max / min):**  
  - Based on data path transition rate.  
  - Ensures data changes within valid limits.

- **For Clock (max / min):**  
  - Clock should switch at regular intervals.  
  - Ensures reliable triggering without distortion.

---

### 2. **Load Analysis**
Examines loading effects on each node in the design.

- **Fanout (max / min):**  
  - Number of loads driven by a single output.  

- **Capacitance (max / min):**  
  - Total capacitive load seen by a driver.  

---

### 3. **Clock Analysis**
Verifies clock network parameters critical to timing.

- **Skew:**  
  - Difference in clock arrival times between flops.  

- **Pulse Width:**  
  - Ensures clock high/low times are within limits.  

---

📘 **In summary:**  
These analyses (setup, hold, slew, load, and clock) collectively ensure that all signal transitions, delays, and clock timings in the design meet the required performance and reliability constraints.

# 🧩 Reg2Reg Setup Timing Analysis (Single Clock Domain)

## 🔹 Overview
In a **register-to-register (reg2reg)** timing path, data is launched from one flip-flop (**launch flop**) and captured by another (**capture flop**) through **combinational logic**.  
Both flip-flops are triggered by the **same clock**.
Clock → Launch FF → Combinational Logic → Capture FF → Clock


---

## 🔹 Key Timing Points
1. **Launch Edge** – Clock edge that triggers data output from the launch flop.  
2. **Capture Edge** – Next active clock edge that captures the data at the capture flop.

---

## 🔹 Setup Time Condition
For correct data capture, the data must arrive **before** the next active clock edge minus the setup time of the capture flop.
Tclk ≥ Tcq + Tcomb + Tsetup + Tskew





**Where:**
- `Tclk` → Clock period  
- `Tcq` → Clock-to-Q delay of launch flop  
- `Tcomb` → Combinational delay  
- `Tsetup` → Setup time of capture flop  
- `Tskew` → Clock skew between capture and launch clocks  

If this condition is violated → **Setup violation** occurs.

---

## 🔹 Clock Skew
- **Positive skew** → Capture clock arrives later → Helps setup (more time for data).  
- **Negative skew** → Capture clock arrives earlier → Hurts setup (less time for data).

---

## 🔹 Timing Analysis Types
### 1. **GBA (Graph-Based Analysis)**
- Uses **worst-case delay** for each path.  
- Fast but **pessimistic**.

### 2. **PBA (Path-Based Analysis)**
- Calculates timing **per specific path** end-to-end.  
- More **accurate** but slower.

---

## 🔹 Pin / Node Convention
- A **pin** (or **node**) represents a connection point in the timing graph (e.g., Q pin of launch flop or D pin of capture flop).  
- Each node has:
  - **Arrival Time (AT)**
  - **Required Arrival Time (RAT)**

---

## 🔹 Setup Analysis Equation
Slack = RAT - AT
If `Slack ≥ 0` → No setup violation.  
If `Slack < 0` → Setup violation.

---

### 📘 Summary Formula:
Tclk ≥ Tcq + Tcomb + Tsetup + Tskew


Ensures that data launched by one flip-flop reaches the next flip-flop **within one clock period**, considering **delays, setup time, and skew**, using either **GBA or PBA** timing models.
# 🧩 Flip-Flop Internal Timing & Advanced STA Concepts

## 🔹 Opening Up the Flip-Flop
A flip-flop can act as either:
- **Launch Flip-Flop:** Sends data on the active clock edge.  
- **Capture Flip-Flop:** Receives and stores data on the next active edge.

Inside a flip-flop, there are two **latches**:
- **Positive Latch:** Active during the high phase of the clock.  
- **Negative Latch:** Active during the low phase of the clock.

---

## 🔹 Key Timing Parameters
- **Setup Time (Tsetup):**  
  Minimum time before the active clock edge during which data must remain stable.

- **Hold Time (Thold):**  
  Minimum time after the clock edge during which data must stay stable.

- **Clock-to-Q Delay (Tcq):**  
  Time from the clock edge to the output data transition at Q.

---

## 🔹 Setup Time Condition (Effect of Setup)
The setup condition ensures that data arrives before the capture clock edge:

Tclk ≥ Tcq + Tcomb + Tsetup + Tskew + Tuncertainty


If setup time increases → available **timing margin** decreases, reducing maximum operating frequency.

---

## 🔹 Jitter & Uncertainty
- **Jitter:** Variation in clock edge timing from its ideal position.  
- **Uncertainty:** Accounts for combined effects of jitter, skew, and variations.  
- **Impact:** Increases setup time requirement and reduces effective clock period.

---

## 🔹 Steps for Jitter Analysis
1. Define clock source and reference period.  
2. Measure edge-to-edge variation (cycle-to-cycle jitter).  
3. Compute RMS or peak jitter.  
4. Include jitter value as **clock uncertainty** in STA.

---

## 🔹 Eye Diagram & Window Creation
- **Eye Diagram:**  
  Overlay of multiple bit periods showing signal integrity and timing margins.  
  - Eye opening → indicates good timing and low noise.  
  - Eye closure → indicates jitter, noise, or skew issues.

- **Timing Window:**  
  Defines valid region for data sampling based on setup/hold and jitter limits.

---

## 🔹 Graphical to Textual Conversion
- Graphical waveforms (clock, data) are represented in text form using:  
  - **Arrival Time (AT)**  
  - **Required Arrival Time (RAT)**  
  - **Slack = RAT – AT**

This allows STA tools to perform timing analysis numerically rather than visually.

---

📘 **In short:**  
Flip-flop internal timing (setup, hold, Tcq), along with jitter, uncertainty, and eye diagram analysis, ensures reliable data capture by verifying that timing windows are not violated in the presence of real-world variations.


# ⚙️ Setup & Hold Timing Concepts — STA Overview

## 🔹 Hold Analysis
Hold analysis checks **minimum delay paths** to ensure data does **not change too early** after the capture clock edge.

- **Hold Time (Thold):**  
  Minimum time after the clock edge during which input data must remain stable.

- **Hold Condition:**
Tcq + Tcomb ≥ Thold + Tskew + Tuncertainty

If violated → **Hold violation** (data captured incorrectly).

---

## 🔹 Slack (Timing Margin)
Slack indicates whether a path meets timing requirements.

- **For Setup:**
Slack = (Tclk - (Tcq + Tcomb + Tsetup + Tskew + Tuncertainty))
- **For Hold:**
Slack = (Tcq + Tcomb - (Thold + Tskew + Tuncertainty))

If `Slack ≥ 0` → timing met.  
If `Slack < 0` → violation.

---

## 🔹 Graphical to Textual Conversion
Timing waveforms are represented numerically using:
- **AT (Arrival Time)** – actual time signal reaches capture point.  
- **RAT (Required Arrival Time)** – latest (for setup) or earliest (for hold) allowable arrival.  
- **Slack = RAT – AT**

This enables **automated STA** instead of manual waveform inspection.

---

## 🔹 On-Chip Variation (OCV)
OCV models **timing variation** due to manufacturing process differences, voltage, or temperature across the chip.

- Each path (data or clock) may vary independently.  
- Delays are adjusted using derating factors:

Delay_actual = Nominal_delay × (1 ± derate)


---

## 🔹 OCV-Based Setup & Hold Analysis
- **Setup OCV:**  
- Launch path → faster (– derate)  
- Capture path → slower (+ derate)  
→ Ensures worst-case **late data** arrival.

- **Hold OCV:**  
- Launch path → slower (+ derate)  
- Capture path → faster (– derate)  
→ Ensures worst-case **early data** arrival.

---

## 🔹 Clock Path Effects

### 1. **Clock Push-Out**
- Launch clock arrives **later** (positive skew).  
- Increases setup margin, reduces hold margin.

### 2. **Clock Pull-In**
- Launch clock arrives **earlier** (negative skew).  
- Reduces setup margin, increases hold margin.

### 3. **Clock Path Pessimism (CPPR)**
- Common clock segments may be double-counted as uncertainty.  
- **Clock Path Pessimism Removal (CPPR):**  
Removes shared path delay between launch and capture clocks to reduce pessimism and improve timing accuracy.

---

📘 **Summary**
Setup analysis ensures data arrives **before** capture edge;  
Hold analysis ensures data **stays stable after** capture edge.  
OCV, jitter, and clock path adjustments refine these checks for **real silicon behavior**.














