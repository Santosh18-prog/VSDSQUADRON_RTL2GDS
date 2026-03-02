# 🚀 VSD-Squadron RTL2GDSII SOC Implementation 

## WEEK-1


<details>
<summary><strong>Phase 1 — OpenLANE Flow Familiarity </strong></summary>

--- 

<details>
  <summary>1.1. Theory</summary>

## SoC Design Using OpenLane

This repository documents my hands-on implementation of the complete **RTL to GDSII ASIC design flow** using OpenLANE.

---

## 1. Digital ASIC Design

Digital ASIC design is the process of converting a hardware description (RTL) into a fabricated silicon chip (ASIC).

![ASIC](phase1/asic.jpeg)

Traditionally, ASIC design flow required:

- EDA Tools
- PDK Data
- Standard Cell Libraries

These together enable the transformation of RTL → ASIC.

---

## 2. Open-Source Digital ASIC Design

In open-source ASIC design:

- EDA tools are open-source
- RTL design is written in HDL (Verilog/VHDL)
- Standard cell libraries are publicly available
- Open PDKs are used

OpenLane integrates:

- Open-source EDA tools
- Open PDK
- Automated flow

And produces:

RTL → GDSII (layout ready for fabrication)

---

### What is a PDK?

PDK = Process Design Kit

A PDK is a collection of files used to model the fabrication process for EDA tools while designing an IC.

It allows designers to create layouts that match actual manufacturing rules.

---

### Contents of a PDK

A PDK typically contains:

- Process Design Rules (DRC rules)
- LVS rules
- PEX rules
- Device Models
- Digital Standard Cell Libraries
- I/O Libraries

These ensure that the design matches the real manufacturing process.

---

### Evolution of IC Design and PDK Usage

In earlier days:

- Chip design was tightly integrated with fabrication.
- Each company (e.g., Intel) controlled manufacturing.
- Designers and fabrication teams worked under the same organization.

These companies controlled:

- Process technology
- Design methodology
- Manufacturing flow

Later:

- Fabless design companies emerged.
- Foundries handled fabrication.
- Designers needed a standardized interface to fabrication.

To separate design from fabrication:

Process Design Kits (PDKs) were introduced.

PDKs standardized the interaction between:

Design team ↔ Manufacturing process

---

## 5. Open PDK

Open PDK used:

- License: Apache 2.0
- Collaboration between Google and SkyWater Technology

FOSS 130nm Production PDK:

Available at:
github.com/google/skywater-pdk

---

### Is 130nm Fast?

Yes.

Example reference:

Intel Pentium 4 Extreme Edition
3.46 GHz (Quad-core era reference)

Even 130nm technology was capable of high-performance designs.


---

## Openlane ASIC Flow


Goal:
- ✅ Clean GDSII
- ✅ Zero DRC
- ✅ Zero LVS mismatch
- ✅ Timing Clean Design

![flow](phase1/flow.jpeg)

---

## 1. RTL Synthesis

### Description
- Converts RTL (Verilog) into a gate-level netlist  
- Maps logic to Standard Cell Library (Sky130 SCL)  
- Performs logic optimization  

### Tool Used
- Yosys (via OpenLANE)

### Output
- Gate-level netlist



---

## 2) Floorplanning

### Description
- Defines die & core area  
- IO placement  
- Power planning  
- Macro placement (if any)

### Objective
- Efficient silicon usage  
- Proper power distribution  
- Low congestion  



---

## 3. Placement

### Steps
- Global Placement  
- Detailed Placement  
- Post-placement Optimization  

### Goals
- Minimize wirelength  
- Improve timing slack  
- Reduce congestion  

---

## 4. Clock Tree Synthesis (CTS)

### Description
- Builds clock distribution network  
- Inserts buffers  
- Minimizes clock skew  

### Objective
Stable clock delivery to all sequential elements.


---

## 5. Routing

### Steps
- Global Routing  
- Detailed Routing  

### Handles
- Congestion  
- Antenna violations  
- Wire optimization  

---


## 6. Signoff Verification

### Physical Verification
- DRC – Magic  
- LVS – Magic + Netgen  

### Timing Verification
- STA – OpenSTA  

### Parasitic Extraction
- RC Extraction – Magic  



---
###  OpenLane Regression Testing

The design exploration utility is used for regression testing.

Regression testing means:

- Running automation on multiple designs
- Comparing results with previously known correct results
- Ensuring no functionality breaks

This ensures flow stability.

---

###  Design For Test (DFT)

DFT techniques improve testability of a chip.

Includes:

- Scan Insertion
- Automatic Test Pattern Generation (ATPG)
- Test Pattern Compression
- Fault Coverage
- Fault Simulation

These techniques ensure high manufacturing test quality.

---

###  Physical Implementation (Automated Place & Route)

Also called:

Automated PnR (Place and Route)

Steps include:

1. Floorplanning (includes power planning)
2. End Decoupling Capacitors & Tap Cell Insertion
3. Placement (Global & Detailed)
4. Post-Placement Optimization
5. Clock Tree Synthesis (CTS)
6. Routing (Global & Detailed)

This converts logical netlist into physical layout.

---

###  Logic Equivalence Check (LEC)

Every time the netlist is modified, verification must be performed.

Why?

Because:

- CTS modifies the netlist.
- Post-placement optimizations modify the netlist.
- Routing optimizations may affect structure.

LEC is used to formally confirm that:

The functionality has NOT changed after netlist modifications.

It ensures:

Original RTL ≡ Final netlist

---

###  Dealing with Antenna Rule Violations

When a metal wire segment is fabricated, it can act as an antenna.

During fabrication:

- Reactive ion etching causes charge accumulation on the wire.
- Accumulated charge can damage transistor gates.
- Gate oxide may break.

This is called an **Antenna Effect**.

---

### Two Solutions for Antenna Violations

1) Bridge the wire to a higher metal layer (by adding jump/bridge)

- Requires router awareness.

2) Add Antenna Diodes

- These leak away accumulated charge.
- Antenna diodes are provided by the Standard Cell Library (SCL).

---

###  Preventive Antenna Approach

Preventive method:

- Add a fake antenna diode near cell input after placement.
- Run antenna checks (Magic tool) on routed layout.
- If violation is detected on a cell input pin:
    Replace fake diode with real antenna diode.

This ensures safe fabrication.

---

###  Static Timing Analysis (STA)

After routing:


Timing analysis is performed using:

- OpenSTA (via OpenROAD)

This ensures setup and hold timing constraints are satisfied.

---

###  Physical Verification (DRC & LVS)

Physical verification ensures manufacturability.

Includes:

1) DRC – Design Rule Check
2) LVS – Layout Versus Schematic


---
</details>

--- 

<details> 

<summary>1.2. LAB</summary>
  
### Determine Flip-flop Ratio

The task is to find the flip-flop ratio ratio for the design picorv32a. This is the ratio of the number of flip flops to the total number of cells.

1. Run OpenLANE:
   $ make mount  -  Opens the Docker container for the OpenLane environment

   % ./flow.tcl -interactive  -  Starts the OpenLane flow in interactive mode to execute the RTL to GDSII flow step-by-step

   % package require openlane 1.0.2  -  Loads the OpenLane v1.0.2 package and its required dependencies

2. prep -design picorv32a

The command:

    prep -design picorv32a

is used to initialize the OpenLane flow for the `picorv32a` design.

It performs the following tasks:
- Loads the RTL (Verilog) files
- Reads the `config.tcl` file
- Links the Sky130 standard cell library
- Applies clock and design constraints
- Creates a new `runs/` directory to store logs, reports, and results

This step must be executed before running synthesis or any physical design stage.

In simple words, `prep -design` prepares the environment for the ASIC design flow.

![prep_design](phase1/1_prep_design.png)

### 3. Run Synthesis
   When we run:

    run_synthesis

in the VSD Cloud OpenLane environment:

- The RTL (Verilog) code is converted into a gate-level netlist
- The design is mapped to Sky130 standard cells
- Logic optimization is performed to reduce area and improve timing
- Static Timing Analysis (STA) is executed
- Reports for cell count, area, and timing are generated

All results are stored inside:

    designs/picorv32a/runs/RUN_2026.02.22_09.53.55

![path](phase1/3_path.png)
In simple words, run_synthesis converts RTL code into real standard cell logic and checks whether the design meets timing requirements.

### Synthesis Result : 


![run_synth](phase1/5-1_stat.png)
![run_synth](phase1/5-2_stat.png)
![run_synth](phase1/5-3_stat.png)

The flipflop ratio is (number of flip flops)/(total number of cells) is  (1613/15762) = 0.102334 = 10.2334%

![Worst_slack](phase1/worst_slack.png)

### Timing Report Explanation

The above timing report shows the synthesis stage timing analysis results.

- TNS (Total Negative Slack) = 0.00  
  This means there are no accumulated timing violations in the design.

- WNS (Worst Negative Slack) = 0.00  
  This indicates that the worst timing path does not violate the setup constraint.

- Worst Setup Slack = 0.52 ns  
  The data arrives 0.52 ns earlier than the required time.  
  This means setup timing is satisfied and the design has a positive timing margin.

- Worst Hold Slack = 0.16 ns  
  The data is stable for 0.16 ns after the clock edge.  
  This confirms that there are no hold violations.
  
</details>

---

</details>


<details>
<summary><strong>Phase 2 — Floorplan Fundamentals</strong></summary>

---

<details>
  <summary>2.1. Theory</summary>
  
## Floor Planning, Pre-Placed Cells & Power Planning

---

##  Chip Floor Planning Considerations

### 1. Define Width and Height of Core & Die

In Physical Design (PD) flow, the first step is to determine:

- Core dimensions  
- Die dimensions  

The chip dimensions depend on the total area of logic gates inside the netlist.

---

### Example Netlist

Consider a simple netlist containing:

- Flip-flops (FF)
- AND gates (A1Y)
- OR gates (O1Y)

These are standard cells.

The chip area depends on the number and dimensions of these logic gates.

---

### Assume Standard Cell Dimensions

Let:

- Each standard cell = 1 unit width × 1 unit height  
- Area of each cell = 1 square unit  

If we place 4 cells together:

Total Area = 4 units  
Possible layout → 2 units × 2 units core  

---

### What is Core & Die?

- Core: Area where logic blocks are placed  
- Die: Entire silicon area including core and IO region  

A chip contains multiple dies on a silicon wafer.

---

### 1.1 Utilization Factor

Even if core area is 2 × 2 = 4 units,

Not all area is usable for cell placement due to:

- Routing space
- Buffers
- Spare cells
- Congestion
- Power planning

### Formula:

Utilization Factor = Area occupied by Netlist / Total Core Area

Example:

UF = 4 / (2 × 2) = 1

But 100% utilization is not practical.

### Practical Utilization:
- Typically 50% – 60%
- Rarely exceeds 70%

Because routing and power network also require space.

---

### 1.2 Aspect Ratio

### Definition:

Aspect Ratio = Height / Width

### Example 1:
Core = 2 units × 2 units

Aspect Ratio = 2 / 2 = 1

![utilization_factor](phase2/utilization_factor.png)

### Example 2:
Core Area = 8 units  
Width = 4 units  
Height = 2 units  

Aspect Ratio = 2 / 4 = 0.5

### Notes:
- Ideal aspect ratio is usually close to 1
- Most chips are rectangular
- Practical aspect ratio range: 0.5 to 2

---

## 2. Define Locations of Pre-Placed Cells

---

### 2.1 What Are Pre-Placed Cells?

Pre-placed cells are:

- Large IP blocks
- Fixed-location modules
- Placed before standard cell placement
- Cannot be moved during placement & routing

---

### Examples of Pre-Placed Cells (IPs)

- Memory
- Clock Gating Cells
- Comparator
- Multiplexer (MUX)

These IPs:

- Are designed separately
- May appear multiple times
- Have fixed locations in the chip
- Are treated as black boxes

---

### 2.2 Black Boxing

When logic blocks are not expanded internally:

- Internal logic is hidden
- Treated as a single module
- Called black boxing

![Black_box](phase2/blk_box.png)

Purpose:

If a block repeats multiple times:
- Instead of showing all internal logic
- Represent as a single block

This simplifies:

- Timing analysis
- Placement
- Floorplanning

---

### 2.3 Floor Planning

Floorplanning involves:

- Arranging IP blocks
- Fixing pre-placed cells
- Assigning physical regions
- Leaving routing channels
- Defining core boundary

Automated placement & routing tools:
- Place standard cells around pre-placed macros

---

## 3. Surrounding Pre-Placed Cells with Decoupling Capacitors

### Pre-Placed Cells

Pre-placed cells (IP blocks/macros) are:

- Large blocks like Memory, Comparator, Clock gating cells, MUX etc.
- Placed before standard cell placement.
- Fixed in location.
- Not moved during placement and routing.
- Treated as black boxes.

  ![pre_placed](phase2/pre_placed.png)

These blocks:

- May consume large switching current.
- May switch simultaneously.
- Can cause supply instability.

---


### IR Drop and Noise Margin Impact

![circuit](phase2/ckt.png)

Example:

If VDD = 1V  
Due to IR drop, internal node may receive only 0.7V.

This may:

- Reduce noise margin.
- Push logic level toward undefined region.
- Cause functional failure.

---

![noise_margin](phase2/noise_margin.png)

### Noise Margin and Undefined Region (Unsafe Zone)

For digital circuits, valid voltage levels are defined as:

- VOL  → Maximum voltage recognized as Logic 0
- VOH  → Minimum voltage recognized as Logic 1
- VIL  → Input voltage threshold for Logic 0
- VIH  → Input voltage threshold for Logic 1

Noise margins are defined as:

NMH = VOH − VIH  
NML = VIL − VOL  

For proper logic operation, both NMH and NML must be positive and sufficiently large.

### Undefined Region

The voltage region between VIL and VIH is called the **Undefined Region**.

This region is an **unsafe zone** because:

- The logic level is not guaranteed to be 0 or 1.
- Circuit behavior becomes unpredictable.
- Output may oscillate or produce incorrect logic.
- Functional failures may occur.

If due to IR drop or ground bounce:

- Logic High drops below VIH
- Logic Low rises above VIL

The signal may enter the undefined region.

This unsafe voltage zone must always be avoided in reliable chip design.

---

## Solution: Decoupling Capacitors (DECAP)

![Decoupling_capacitor](phase2/decoupling_cap.png)

To solve supply instability:

We place decoupling capacitors near macros.

### What is a Decoupling Capacitor?

- A local capacitor placed between VDD and VSS.
- Stores charge.
- Supplies current locally during switching.
- Reduces sudden current demand from global supply.

---

## Surrounding the Pre-Placed Cells

Decoupling capacitors are placed around large IP blocks:

![Decoupling_capacitor_pre_placed](phase2/decoupling_cap_ip.png)

These DECAP cells are inserted:

- Around the boundary of the macro.
- Between large IP blocks.
- Near power sensitive regions.

---

## Why Surround the Pre-Placed Cells?

Because:

- Macros draw high instantaneous current.
- Long power routing introduces resistance.
- Large switching activity causes voltage fluctuations.

Decoupling capacitors:

- Provide a local charge reservoir.
- Reduce IR drop.
- Reduce ground bounce.
- Improve power integrity.
- Maintain stable VDD and VSS.
- Keep signals away from the undefined (unsafe) region.

---
## 4. Power Planning 

![power_supply](phase2/power_supply.png)

### Scenario:

If a macro is repeated many times:

- All blocks may switch simultaneously
- Large current drawn at same time
- Causes IR drop
- Causes Ground bounce

---

### Two Major Issues:

### 1) When Logic switches 1 → 0

- All capacitors discharge
- Sudden current to ground
- Causes ground bounce

![ground_bounce](phase2/ground_bounce.png)

### 2) When Logic switches 0 → 1

- All capacitors charge
- Heavy current from VDD
- Causes voltage drop at VDD

 ![Voltage_drop](phase2/voltage_drop.png)

---

### Solution: Multiple Power Tap Points

Instead of:

Single VDD/VSS source

We provide:

- Multiple VDD rails
- Multiple VSS rails
- Power grid network

![power_planning](phase2/power_planning.png)

![power_planning](phase2/power_plan.png)

This ensures:

- Current distributed evenly
- Reduced IR drop
- Reduced ground bounce
- Stable supply


---


## 5) Pin Placement

### Complete Design

The complete design connects:

![complete_design](phase2/complete_design.png)

The connectivity information (how gates are connected) is coded using a hardware description language.

This connectivity description is written using **VHDL language**  
and is called the **Netlist**.

---

After floorplanning, pins are placed on the boundary of the die.

Inputs (DIN1–DIN4) and clocks (CLK1, CLK2) are placed on one side.

Outputs (DOUT1–DOUT4, CLKOUT) are placed on another side.

Pin placement is done carefully to:

- Reduce routing congestion
- Reduce wirelength
- Improve timing

![pin_placement](phase2/pin_placement.png)

### Important Note on Clock Pins

As observed in the pin placement:

- Clock pins (CLK1, CLK2) are bigger in size.

Reason:

Clock signals drive multiple flip-flops continuously.

They require:

- Lower resistance path
- Better current handling
- Stronger metal routing

Similarly:

- CLKOUT should also have lower resistance.

Conclusion:

Bigger the pin size → Lower the resistance.

This improves clock distribution and reduces delay/skew.

---

### Placement Tool Behavior

Now once size and pin placement are defined,

Automatic routing and placement tools try to place standard cells in the available core area.

However,

Certain regions must not have standard cells placed.

---

## Logical Cell Placement Blockage

To prevent tools from placing cells in restricted areas,

We create a **Logical Cell Placement Blockage**.

![cell_placement_blockage](phase2/cell_placement_blockage.png)

Blockage means:

- A defined region in the core
- Where placement tool is restricted
- No cell can be placed in that area

This blockage ensures:

- Space is reserved for routing
- Space is reserved for future modifications
- Macros and critical routing areas are protected
- Congestion is reduced

The placement and routing tool will not place any cell inside the blockage area.

---
</details>

---

<details>
  <summary>2.2. LAB</summary>

  ## Floor Planning:

   Command:

    run_floorplan

![floor_planning](phase2/floor_plan.png)

During this step:

- The chip area (die size) is calculated.

- The width and height of the design are decided.

- IO pins are placed on the boundary of the chip.

- Tap cells are inserted to prevent latch-up issues.

- Decoupling (decap) cells are inserted to reduce power noise and IR drop.

- Power planning is performed for VPWR (power) and VGND (ground).

- The Power Distribution Network (PDN) is generated.

After this step, the chip has:

- Fixed dimensions

- IO pin locations

- Power grid

- Prepared rows for standard cell placement

### Config.tcl

`config.tcl` controls how the chip is built.

It decides:
- How big the chip will be
- How much area is used by logic cells
- The shape of the chip
- The target operating frequency
- How the floorplanning stage should behave

![config.tcl](phase2/config_tcl_before.png)

With this config.tcl we will get floorplanning output as:

![output](phase2/util_before.png)

- `FP_ASPECT_RATIO = 1`  
  Sets the chip shape to square (width ≈ height).

- `FP_CORE_UTIL = 10`  
  Sets core utilization to 10%.  
  Only 10% of the core area is used for logic cells, making the chip area larger.

  This is the def file for this config.tcl

  ![DEF](phase2/def_before.png)
### Creating sky130A_sky130_fd_sc_hd_config.tcl file inside picorv32a directory:
This file overrides the default core utilization value specifically for the sky130A PDK and sky130_fd_sc_hd standard cell library.

![sky130A_sky130_fd_sc_hd_config.tcl](phase2/sky.tcl.png)

### Modified config.tcl
![Config.tcl](phase2/modified_tcl.png)


After creating the new configuration file and updating config.tcl, the floorplanning stage is executed again.

### Floorplan Output

![OUTPUT](phase2/util_after.png)

  picorv32a.def after adding the sky130A_sky130_fd_sc_hd_config.tcl file

  The def file is in:

  
   Command:

    /home/vscode/Desktop/OpenLane/designs/picorv32a/runs/RUN_2026.02.25_01.46.35/results/floorplan/picorv32a.def


  ![def](phase2/def_after.png)

## Effect of the Modification

### Previous Configuration

- Core Utilization = 10%
- Chip area was larger
- More unused white space inside the core
- Lower routing congestion

### After Modification

- Core Utilization = 30%
- More standard cells are packed inside the core
- Chip area becomes smaller compared to 10% utilization
- Routing density increases moderately
- Area utilization becomes more efficient

## What Happens After Adding sky130A_sky130_fd_sc_hd_config.tcl

After adding the file `sky130A_sky130_fd_sc_hd_config.tcl`, OpenLane reads this file automatically during the flow.

This file overrides the default configuration for the sky130A PDK and sky130_fd_sc_hd standard cell library.

As a result:

- The new core utilization value (30%) is applied.
- Floorplanning is recalculated using the updated value.
- The chip area is reduced compared to 10% utilization.
- Standard cells are placed more compactly.
- Overall area usage becomes more efficient.

## Viewing Floorplan Layout in Magic using this commands

```bash
cd /home/vscode/Desktop/OpenLane/designs/picorv32a/runs/RUN_2026.02.25_01.46.35

magic -T ~/.ciel/sky130A/libs.tech/magic/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32a.def &


  ```
![layout](phase2/layout_new.png)
![layout1](phase2/ln-2.png)
![layout2](phase2/ln-3.png)

## Basic Magic Layout Viewing Commands

- Press `v` to fit the entire layout to the screen.

- To zoom into a specific area:
  - Left click 
  - Right click to form a bounding box.
  - Press `Ctrl + Z` to zoom into the selected area.

- To select a specific object:
  - Place the mouse pointer over the object.
  - Press `s` to select it.
  - In the `tkcon` window, type:
    ```
    what
    ```
    to view the details of the selected object.

    ![label](phase2/label.png)

 ### why macros like RAM behave differently than standard cells?

- Because they are large pre-designed memory blocks treated as black boxes, while standard cells are small logic gates.

- RAM delay comes from internal memory access (decoding, wordline, bitline, sensing), whereas standard cell delay is just simple gate delay.

- Also, macros cannot be resized or internally optimized like standard cells during physical design.
  
</details>



  ---
  
</details>

<details>
<summary><strong>Phase 3 — Timing Literacy with Ideal Clocks</strong></summary>
  
---

 <details><summary>3.1. Theory</summary>


## 1. Introduction to Setup Timing

Timing analysis is performed using an **ideal clock** (before clock tree is built).

We consider:

- Launch Flip-Flop
- Combinational Logic
- Capture Flip-Flop

Clock is assumed ideal (no clock tree delay).

---

![ideal](phase3/ideal.png)

## 2. Basic Setup Timing Condition

At time t = 0  
Clock edge reaches the **launch flip-flop**.

At time t = T  
Next clock edge reaches the **capture flip-flop**.

Let:

Θ = Total combinational delay between launch and capture FF  
T = Clock period  

For correct operation:

Θ < T

If Θ exceeds T:

- Clock period must be increased  
- Frequency decreases  

---

## 3. Practical Scenario – Flip-Flop Structure

A flip-flop is not a simple black box.  
It contains:

- MUX1
- MUX2
- Several internal logic gates

![mux](phase3/mux.png)

When:

CLK = 0  
- Data propagates inside first stage  
- Output remains stable  

CLK transitions 0 → 1  
- Data is captured  
- Output changes  

Because flip-flop is built using logic gates and multiplexers:

Internal delays exist.

This creates a **setup time requirement**.

---

## 4. Setup Time Consideration

The capture flip-flop needs finite time to settle.

Let:

S = Setup time  

Effective timing condition becomes:

Θ < (T - S)

![setup](phase3/delay.png)

So available time reduces.

---

## 5. Introduction to Clock Jitter

Clock edges are ideally expected at:

0, T, 2T, 3T, ...

However, due to circuit variations:

- Clock edge may arrive slightly earlier or later
- This variation is called **Clock Jitter**

![jitter](phase3/uncertinity.png)

Clock jitter = temporary variation of clock period.

Cause:
- Clock source circuitry variation
- Environmental and internal variations

---

## 6. Introducing Clock Uncertainty

All clock variations are represented as a single parameter:

Clock Uncertainty (δU)

New timing equation becomes:

Θ < (T - S - δU)

![delay](phase3/uc-delay.png)

Where:

- Θ = Combinational delay
- S = Setup time
- δU = Clock uncertainty

---

## 7. Example Calculation

Assume:

T = 1 ns  
S = 0.01 ns  
Uncertainty = 0.09 ns  

Then:

Θ < (1 - 0.01 - 0.09)

Θ < 0.9 ns

So maximum allowed combinational delay = 0.9 ns.

---

## 8. Complete Timing Path Expression

Total delay (Θ) includes:

- Clock to Q delay of launch flip-flop
- Delay of combinational logic block 1
- Delay of combinational logic block 2
- Additional gate delays

General form:

![ff](phase3/ff.png)
Θ = FF1 (Clock-to-Q delay)
    + wire delay estimate 1
    + Delay of gate 1
    + wire delay estimate 2
    + Delay of gate 2
    + wire delay estimate 3

![ff](phase3/ff_delay.png)

Θ = FF1 (Clock-to-Q delay)
    + Delay of gate 1
    + Delay of gate 2

Final condition:

Θ < (T - S - δU)

If this condition satisfies → setup timing is met.

If not → timing violation occurs.

---

## 9. Summary

Initial condition:
Θ < T

With setup time:
Θ < (T - S)

With setup time + uncertainty:
Θ < (T - S - δU)

This is the complete setup timing condition used in OpenSTA analysis.
   
 </details>
 
 ---

 <details><summary>3.2. LAB</summary>

##  Objective

To perform Static Timing Analysis (STA) on the synthesized netlist of the `picorv32a` design using OpenSTA and to analyze setup and hold timing behavior under worst-case PVT corners.

---


The RTL (`picorv32a.v`) was synthesized using OpenLane.
The synthesized gate-level netlist was generated inside:

```
results/synthesis/picorv32a.v
```

A custom `pre_sta.conf` file was created to perform static timing analysis.

![pre_sta](phase3/pre_sta.png)

The following PVT corners were selected:

- Setup (MAX delay) → `ss_100C_1v60`
- Hold (MIN delay) → `ff_n40C_1v95`

This is the path to get the lib files

![lib](phase3/libs.png)

This corresponds to:

- Slow process, high temperature, low voltage → worst-case setup
- Fast process, low temperature, high voltage → worst-case hold

---

The STA was executed using:

```bash
sta pre_sta.conf
```
## RESULT

![sta](phase3/sta-1.png)
![sta2](phase3/sta-2.png)
![sta3](phase3/sta-3.png)
![sta4](phase3/sta-4.png)
![sta5](phase3/sta-5.png)

##  Results Observed

###  Setup Timing (MAX Path)

- Data Arrival Time = 22.45 ns
- Data Required Time = 11.71 ns
- Slack = -10.75 ns (VIOLATED)

WNS = -10.75 ns  
TNS = -553.07 ns  

This indicates significant setup timing violation.

---

###  Hold Timing (MIN Path)

- Slack = +0.25 ns (MET)

Hold timing is satisfied.

---


## 7️ High Fanout Observation

During analysis, several high fanout nets were observed (fanout 10, 6, 5).

High fanout causes:

- Increased capacitive loading
- Higher RC delay
- Slower signal transition
- Increased data arrival time

This contributes to the setup timing violation.

---
### 🛠 Optimization Commands Used

Upsized selected buffers using:

```tcl
replace_cell _16820_ sky130_fd_sc_hd__buf_4
replace_cell _17121_ sky130_fd_sc_hd__buf_8
```

Re-ran STA:

```tcl
report_checks -fields {net cap slew input_pins} -digits 4
```

---

![red](phase3/red.png)
![red](phase3/red-1.png)
![re](phase3/re.png)
![red](phase3/re-1.png)

Again optimizing using these commands:


```tcl
replace_cell _17506_ sky130_fd_sc_hd__or4bb_4   
replace_cell _22014_ sky130_fd_sc_hd__or4_4
```

Re-ran STA:

```tcl
report_checks -fields {net cap slew input_pins} -digits 4
```

---

![and](phase3/and.png)
![and1](phase3/and2.png)
![or](phase3/or.png)
![or2](phase3/or2.png)

### 📊 Results After Optimization

- Data Arrival Time:  -21.4136 ns
- Data Required Time: 11.7053 ns
- Slack: **-9.7083 ns (VIOLATED)**

### 📈 Improvement

Slack improved from:

```
-10.75 ns → -9.7083 ns
```

Improvement ≈ **1.0417 ns**

### 🧠 Technical Reason for Improvement

Upsizing buffer:

- Increases drive strength
- Reduces output resistance (R)
- Improves slew
- Reduces RC delay on high-fanout nets

##  Conclusion

- Setup timing is violated at synthesis level.
- Hold timing is satisfied.
- The violation is due to large combinational delay relative to the 12 ns clock period.
- High fanout nets significantly impact delay.

   
 </details>

 ---
 
</details>

<details>
  <summary><strong>Phase 4 — CTS and Timing with Real Clocks</strong></summary>

  ---
  <details>
   <summary>4.1. Theory</summary>

   ---

## 1. Clock Tree Synthesis (CTS)

### Problem Before CTS

In the design, a single clock source (CLK1) must drive multiple flip-flops.

![problem](phase4/prob.png)
![problem](phase4/prob1.png)

If clock is directly connected:

- Time to reach FF1 = T1  
- Time to reach FF2 = T2  

If:

T2 ≠ T1  

Then:

Skew = T2 − T1  

This creates clock skew.

If skew is large → design becomes unreliable.

---

### Why Bad Clock Tree is Problematic?

A badly constructed clock tree:

- Causes different clock arrival times  
- Creates setup/hold violations  
- Increases timing uncertainty  

---

## 2. H-Tree Concept

![H-tree](phase4/Htree.png)

H-Tree structure connects the clock to the midpoint of flip-flops.

### Key Idea

- Clock is routed symmetrically  
- Equal distance from clock source to all flip-flops  
- Ensures nearly equal arrival time  

Therefore:

Skew ≈ 0  

---

## 3. Clock Tree Buffering

Clock nets are physical wires.

Wires have:

- Resistance (R)  
- Capacitance (C)  

Due to RC:

- Signal integrity reduces  
- Edge becomes slow  
- Slew increases  

RC effect distorts waveform before reaching flip-flop.

---

### RC Network Effect

Clock signal passes through:

R → C → Load  

Waveform becomes:

- Slow rising edge  
- Reduced slope  
- Increased delay  

This affects:

- Setup timing  
- Hold timing  

---

## 4. Solution: Repeaters (Buffers)

Best solution: Add buffers.

![Buffers](phase4/Buff.png)

Repeaters:

- Restore signal strength  
- Improve rise/fall time  
- Reduce RC delay impact  

Each buffer re-generates clean clock signal.

Thus:

- Signal reaches flip-flops properly  
- Slew improves  
- Delay optimized  

---

## 5. Clock Net Shielding

Clock nets are critical nets.

If not shielded:

- Crosstalk occurs  
- Coupling capacitance increases  
- Noise/glitches may appear  

---

### Problems Due to Crosstalk

- Glitch  
- Delta Delay  

![glitch](phase4/Glitch.png)
![Delta](phase4/Delta.png)
If aggressor net switches:

- Victim net sees bump  
- May create timing violation  
- Can flip memory content  

---

### Shielding Technique

Shield wire is placed between aggressive nets.

![Shield](phase4/shield.png)

Shield connected to:

- VDD  
- GND  

This reduces coupling capacitance and prevents glitch.

⚠ Important:

We do NOT shield all nets.

We only shield:

- Critical nets  
- Long nets  
- Clock nets  

Because shielding increases:

- Routing congestion  
- Area  
- Cost  

---

## 6. Timing Analysis (With Real Clocks)

### Setup Analysis – Single Clock

Launch flop sends data.  
Capture flop captures data.

### Definitions

- Data Arrival Time  
- Data Required Time  
- Clock Network Delay  

---

### Setup Condition

![setup](phase4/setup.png)

Data Arrival Time < Data Required Time  

If:

Data Arrival Time > Data Required Time  

→ Setup violation

---

### Clock Skew Impact on Setup

Let:

Δ1 = clock delay to launch flop  
Δ2 = clock delay to capture flop  

Skew = Δ1 − Δ2  

Setup slack:

Slack = Data Required − Data Arrival  

---

## 7. Hold Timing Analysis

In Hold analysis:

- Same clock edge used  
- Check if new data corrupts old data  

---

### Hold Condition

![hold](phase4/hold.png)

Data Arrival Time > Hold Time  

If violated:

→ Hold violation  

---

## 8. Real Clock Model

Clock delay consists of:

- Wire RC delay  
- Buffer delay  
- Multiple stages  

  ![delta1](phase4/delta1.png)
  ![delta2](phase4/delta2.png)

Skew:

Skew = Δ1 − Δ2  

---

## Final Timing Equations

### Setup Timing

(θ + Δ1) < (T + Δ2) − SU  

Where:

- T = clock period  
- SU = setup uncertainty  

---

### Hold Timing

(θ + Δ1) > (Δ2 + HU)  

Where:

- HU = hold uncertainty  
  </details>

---

<details>
  <summary>4.2. LAB</summary>

  ---

## CTS, STA and Timing Optimization -- picorv32a

------------------------------------------------------------------------

## Full OpenLane Flow Used

    run_synthesis
    run_floorplan
    run_placement
    run_cts
    run_routing

------------------------------------------------------------------------

## Reading LEF and DEF in OpenROAD

Launch OpenROAD:

    openroad

Read merged LEF file:

    read_lef /openlane/designs/picorv32a/runs/RUN_2026.02.25_01.29.26/tmp/merged.nom.lef

Read CTS DEF file:

    read_def /openlane/designs/picorv32a/runs/RUN_2026.02.25_01.29.26/results/cts/picorv32a.def

### Observations After Loading DEF

-   14 Technology Layers created
-   25 Technology Vias created
-   441 Library Cells loaded
-   32014 Components
-   21614 Nets
-   411 Pins

Save database:

    write_db pico_cts1.db

------------------------------------------------------------------------

## Loading Design for Timing Analysis

Reload database:

    read_db pico_cts.db

Read Verilog netlist:

    read_verilog /openlane/designs/picorv32a/runs/RUN_2026.02.25_01.29.26/results/synthesis/picorv32a.v

Read Liberty file:

    read_liberty $::env(LIB_SYNTH_COMPLETE)

Link design:

    link_design picorv32a

Read SDC constraints:

    read_sdc /openlane/designs/picorv32a/src/picorv32a.sdc

Set propagated clock:

    set_propagated_clock [all_clocks]

Generate timing report:

    report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

------------------------------------------------------------------------

![run](phase4/run.png)
![place](phase4/place.png)
![openroad](phase4/openroad.png)
![openroad](phase4/p-1.png)
![openroad](phase4/p-2.png)
![openroad](phase4/p-3.png)
![openroad](phase4/p-4.png)
![openroad](phase4/p-5.png)

## Hold Timing Analysis

Path Type: MIN
Startpoint: irq[4]
Endpoint: Flip-flop D input

Data Required Time = 3.3461 ns
Data Arrival Time = 2.6119 ns

Slack Calculation:

Slack = Required Time - Arrival Time
Slack = 3.3461 - 2.6119
Slack = -0.7342 ns

Result: Hold Violation

Conclusion: Data is arriving too early at the capture flip-flop.

------------------------------------------------------------------------

## Setup Timing Analysis

Path Type: MAX
Startpoint: Flip-flop
Endpoint: mem_la_wdata[16]

Data Required Time = 9.6000 ns
Data Arrival Time = 5.6667 ns

Slack = 9.6000 - 5.6667
Slack = +3.9333 ns

Result: Setup Timing MET

------------------------------------------------------------------------

## Clock Skew Analysis

Commands used:

    report_clock_skew -hold
    report_clock_skew -setup

Observed:

-   Clock Latency ≈ 2.99 ns
-   Skew = 0.00 ns

Clock tree is balanced.

------------------------------------------------------------------------

## Optimization Strategy Applied

    set ::env(SYNTH_STRATEGY) "DELAY 3"
    set ::env(SYNTH_SIZING) 1

Re-run flow:

    run_synthesis
    run_floorplan
    run_placement
    run_cts

------------------------------------------------------------------------

## Floorplan Comparison

Before Optimization:

Width = 837.2
Height = 835.04

After Optimization:

Width = 731.86
Height = 731.68

Observation:

-   Reduced core area
-   Improved placement compactness
-   Better optimization impact

------------------------------------------------------------------------

# Final Result Summary

Setup : MET
Hold : VIOLATED (Initial CTS Run)
Clock Skew : Balanced

------------------------------------------------------------------------

# Conclusion

This lab provided hands-on understanding of:

-   Post-CTS timing analysis
-   Slack calculation
-   Setup and Hold violations
-   Clock skew behavior
-   Synthesis optimization strategies
-   Impact of optimization on area and timing

------------------------------------------------------------------------
</details>

---

</details>

<details>
  <summary><strong>Phase 5 — PDN Awareness  </strong></summary>

  ---
<details>
  <summary>5.1. Theory</summary>

  ---

 ## Introduction

Routing is one of the most important stages in Physical Design. Its
objective is to connect all pins belonging to the same net while
satisfying Design Rule Constraints (DRC).

![route](phase5/route.png)

In industry, routing is typically divided into two main stages:

1.  Global Routing (Fast Route)
2.  Detailed Routing (Detail Route)

------------------------------------------------------------------------

## Routing Flow Overview

Routing ├── Global Route (FastRoute) └── Detailed Route (TritonRoute)

Global Route is done using FastRoute. Detailed Route is done using
TritonRoute.

------------------------------------------------------------------------

## Global Routing (Fast Route)

Global routing is a high-level routing stage.

It does not create exact metal geometries. Instead, it generates routing
guides.

### Key Features

-   Divides routing region into rectangular grid cells.
-   Abstracted as a coarse 3D routing grid.
-   Determines approximate routing paths.
-   Generates route guides for each net.
-   Estimates congestion between global cells.

The colored rectangular boxes observed after fast route represent
routing guides generated for each net.

These boxes define where the detailed router is allowed to route.

------------------------------------------------------------------------

## Understanding Routing Guides

After FastRoute:

-   Each net gets a routing guide.
-   Guides indicate which layers and regions can be used.
-   No actual metal wires are drawn yet.
-   Only approximate connectivity is determined.

Global routing ensures inter-guide connectivity but does not ensure
final pin-to-pin geometry connection.

------------------------------------------------------------------------

## Detailed Routing (TritonRoute)

Detailed routing is the stage where exact metal geometries are created.

TritonRoute:

-   Takes routing guides as input.
-   Performs initial detailed routing.
-   Connects all pins geometrically.
-   Ensures DRC compliance.
-   Uses panel-based routing scheme.
-   Supports multi-layer routing.
-   Performs intra-layer parallel routing.
-   Performs inter-layer sequential routing.

------------------------------------------------------------------------

## What Detailed Route Does

In detailed route:

-   Actual wires are drawn.
-   Vias are inserted between layers.
-   Spacing, width and design rules are satisfied.
-   Final physical connectivity is created.

Before detailed routing, pins are not physically connected. After
detailed routing, exact geometrical connections are established.

------------------------------------------------------------------------

## Example Concept

Consider 4 pins: A, B, C, D All belong to the same net.

Global Route:

-   Creates routing guides covering these pins.

Detailed Route:

-   Connects A, B, C, D with actual metal segments.
-   Finds optimal path.
-   Ensures DRC clean routing.

------------------------------------------------------------------------

## Routing Grid Representation

Global routing uses:

-   Rectangular global cells
-   Global edges between cells
-   Abstract 3D routing grid model

It estimates how many tracks are available between adjacent global
cells.

------------------------------------------------------------------------

## TritonRoute Characteristics

-   Grid-based detailed router
-   Uses MILP-based optimization strategies
-   Handles DRC violations iteratively
-   Performs rip-up and re-route if required
-   Focuses on clean final layout generation

------------------------------------------------------------------------

## Commands Used in OpenLane

To run routing:

    run_routing

This automatically performs:

1.  Global Routing (FastRoute)
2.  Detailed Routing (TritonRoute)

------------------------------------------------------------------------

## Summary

Global Routing (FastRoute): - Coarse routing - Generates routing
guides - Abstract connectivity

Detailed Routing (TritonRoute): - Final metal routing - DRC-compliant
connections - Pin-to-pin geometry generation

------------------------------------------------------------------------

## Key Learning

-   Understood difference between Global and Detailed routing
-   Learned how routing guides are generated
-   Studied TritonRoute working mechanism
-   Understood grid-based routing model
-   Learned how nets are geometrically connected

</details>

---

<details><summary>5.2. LAB</summary>
  
  ---
  

## Design Preparation

Command used:

    prep -design picorv32a -tag FRESH_RUN

Observations:

-   Configuration loaded from designs/picorv32a/config.tcl
-   PDK: sky130A
-   Standard Cell Library: sky130_fd_sc_hd
-   Run directory created at: /openlane/designs/picorv32a/runs/FRESH_RUN

------------------------------------------------------------------------

# Synthesis

Command:

    run_synthesis

Stages:

-   Logic synthesis
-   Optimization
-   Single-corner STA

------------------------------------------------------------------------

# Floorplanning

Command:

    run_floorplan

Operations performed:

-   Core area calculation
-   IO placement
-   Tap & Decap insertion
-   Power planning
-   PDN generation

Floorplan dimensions:

Width = 731.86
Height = 731.68

------------------------------------------------------------------------

# Placement

Command:

    run_placement

Steps observed:

1.  Global Placement
2.  STA after placement
3.  Resizer Design Optimization
4.  Detailed Placement
5.  Post-placement STA

Global placement arranges cells optimally. Detailed placement legalizes
the cell positions.

------------------------------------------------------------------------

# Clock Tree Synthesis (CTS)

Command:

    run_cts

Operations:

-   Clock buffer insertion
-   Clock balancing
-   Clock tree construction
-   Post-CTS STA

CTS ensures reduced clock skew and balanced clock latency across
flip-flops.

------------------------------------------------------------------------

# Power Distribution Network (PDN)

Command:

    gen_pdn

PDN generation includes:

-   Power stripes (VDD/VSS)
-   Power grid formation
-   Macro/block connection
-   Core ring formation

Ensures stable power delivery and reduces IR drop risk.

------------------------------------------------------------------------

# Routing

Command:

    run_routing

Routing stages:

Step 15: Global Routing Resizer Optimization
Step 16: STA
Step 17: Routing Timing Optimization
Step 18: STA
Step 19: Global Routing
Step 20: Netlist write
Step 21: STA
Step 22: Fill Insertion
Step 23: Detailed Routing
Step 24: Wire Length Check

------------------------------------------------------------------------

![final](phase5/final.png)
![final](phase5/f2.png)
![final](phase5/f3.png)
![result](rount_result.png)


## Routing Observations

-   Global routing completed successfully.
-   Antenna repair attempted (1 violation remained).
-   No DRC violations after detailed routing.
-   Final layout is DRC clean.


## Final Routing Result

-   Detailed Routing Completed
-   No DRC Violations
-   Wire length report generated
-   Design ready for final verification

------------------------------------------------------------------------

## Key Learnings

-   Understood full OpenLane backend flow
-   Learned placement stages and optimization
-   Understood CTS impact on timing
-   Studied PDN generation
-   Differentiated global vs detailed routing
-   Analyzed antenna effects
-   Verified DRC clean layout
-   Interpreted wire length report

------------------------------------------------------------------------

## Conclusion

This lab provided hands-on experience with:

-   Backend Physical Design flow
-   Placement and legalization
-   Clock Tree Synthesis
-   Power Grid generation
-   Routing optimization
-   DRC clean layout generation

## Layout

![layout](phase5/layout.png)
![layout](phase5/l2.png)
![layout](phase5/l3.png)
![layout](phase5/l4.png)
  
</details>

---

</details>
