# Week 6

This repository documents my progress, key learnings, and detailed labs of Week 6. It focuses on the complete **RTL-to-GDSII flow** using open-source tools like **OpenLANE**, **Magic**, **OpenSTA**, and **ngspice**.

This project covers the entire journey: starting from a Verilog RTL description of a `picorv32a` CPU core, synthesizing it, running placement and routing, and finally sign-off.

-----

## 📂 Repository Structure

Here is a quick guide to the files in this repository:

```
.
├── 📄 Day1_THEORY.md
├── 📄 Day2_THEORY.md
├── 📄 Day3_THEORY.md
├── 📄 Day4_THEORY.md
├── 📄 Day5_THEORY.md
├── 🔬 Labs.md
└── 📖 README.md
```

  * **DayX\_THEORY.md**: Contains the theoretical concepts and notes for each day.
  * **Labs.md**: A detailed log of all practical labs, commands, and results.

-----

## 💡 Daily Lab Summaries

### 📅 Day 1: Synthesis & Flop Ratio

Learned to run the synthesis flow (`run_synthesis`) in OpenLANE, converting RTL (Verilog) into a gate-level netlist. We analyzed the output statistics to calculate the design's **Flop Ratio** (sequential vs. combinational cells).

### 🏠 Day 2: Floorplan & Placement

Performed **Floorplanning** (`run_floorplan`) to define the core/die area and place the I/O pins. After that, we ran **Placement** (`run_placement`) to legally place all the standard cells. We visualized the resulting `.def` (Design Exchange Format) files in the Magic layout tool.

### 🎨 Day 3: Custom Inverter Design & DRC

Dived into full-custom layout by designing a standard cell inverter from scratch in Magic. We performed **layout-to-SPICE extraction** (`ext2spice`) to get a netlist with parasitics and ran post-layout simulations in `ngspice` to characterize it. We also learned to debug and fix DRC rules in the PDK's `.tech` file.

### 🔧 Day 4: Standard Cell Integration & Timing ECO

Integrated our custom-designed inverter into the main `picorv32a` design by generating a `.lef` (Library Exchange Format) abstract and using its `.lib` timing file. We also performed a manual **Timing ECO (Engineering Change Order)** using OpenSTA, fixing setup violations by strategically replacing cells (`replace_cell`) with higher-drive-strength versions.

### 🔌 Day 5: PDN, Routing & Post-Route STA

Completed the PnR flow by first generating the **Power Distribution Network (PDN)** with `gen_pdn`. We then ran **Detailed Routing** (`run_routing`) to connect all the cells. Finally, we performed a "signoff-level" **Post-Route Static Timing Analysis (STA)**, using the extracted parasitic **SPEF** file to get the most accurate, real-world timing results.

-----

## 🧬 The PnR Flow & Key Files

The entire process is a chain of steps where the output of one stage becomes the input for the next. Here are the most critical files and how they are generated:

  * **1. RTL (Verilog)**

      * **What it is:** This is initial design code.
      * **File:** `picorv32a.v` (This is our primary input).

  * **2. Synthesis** (using `run_synthesis`)

      * **What it is:** A gate-level **Netlist**. This is the RTL design mapped to standard cells (AND, OR, DFF, etc.).
      * **File to find:** `.../results/synthesis/picorv32a.v`

  * **3. Floorplan & Placement** (using `run_floorplan`, `run_placement`)

      * **What it is:** A **Design Exchange Format (DEF)** file. This text file describes the *physical placement* of all cells and I/O pins.
      * **File to find:** `.../results/placement/picorv32a.placement.def`
      * **How to view:** Open this file in **Magic** to see the placed cells.

  * **4. Cell Abstraction** (using Magic's `lef write` command)

      * **What it is:** A **Library Exchange Format (LEF)** file. This is an abstract, "black box" model of a cell, defining its physical footprint and pin locations for the router.
      * **File to find:** `sky130_vsdinv.lef` (This was the one we made).

  * **5. CTS & Routing** (using `run_cts`, `run_routing`)

      * **What they are:**
        1.  The **final routed DEF** (with clock trees and all signal wires connected).
        2.  The **Parasitic (SPEF)** file.
      * **DEF File:** `.../results/routing/picorv32a.def` (This is the final layout).
      * **SPEF File:** `.../results/routing/picorv32a.spef`. This file contains all the parasitic Resistor (R) and Capacitor (C) values from the physical wires, which is crucial for accurate signoff timing analysis.
