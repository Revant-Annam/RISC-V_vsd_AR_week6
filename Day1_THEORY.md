# Sky130 Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK

This documents a high-level introduction to the entire chip design ecosystem, from the physical hardware components to the abstract software that controls them and OpenLane introduction.

---

## The Main Parts of a Chip

A chip isn't just one monolithic block; it's a hierarchical system of components.

1.  **Package and Pads:** The chip is housed in a package, such as the **QFN-48** (Quad Flat No-leads, 48 pins). The external pins of this package connect to the **PADS** on the perimeter of the silicon **Die**.
2.  **Die:** This is the actual square of silicon containing all the circuitry.
3.  **Core:** This is the functional brain of the chip, which sits inside the ring of I/O pads.
<p align ="center">
   <img width="632" height="556" alt="image" src="https://github.com/user-attachments/assets/7d7152da-8644-4030-b1d4-d653a6d47097" />
</p>

The **Core** itself is further broken down:

* **Macros:** These are large, complex, pre-designed digital blocks. In this design, they include the main **RISC-V SoC**, and the **gpio bank**.
* **Foundry IPs (Intellectual Property):** These are specialized, often analog or mixed-signal, blocks provided by the foundry (e.g., Skywater). The key IPs here are the **PLL** (clock), **ADC** (Analog-to-Digital Converter), the **SRAM** (memory) and **DAC** (Digital-to-Analog Converter).

<p align ="center">
<img width="632" height="556" alt="image" src="https://github.com/user-attachments/assets/b7f44aff-acb6-4ef1-82ad-87f0da6c1d30" />
</p>

## High-Level Computer Workflow

At a high level, a computer's operation is a translation from human-readable software to machine-executable hardware operations.

1.  **Application Software:** The programs which are interacted by the user.
2.  **System Software:** This is the layer that translates application requests into machine code. It includes:
    * **OS (Operating System):** Manages the hardware, allocates memory, and handles I/O.
    * **Compiler:** Converts high-level code (like C or C++) into assembly language instructions.
    * **Assembler:** Converts the human-readable assembly instructions into binary machine code (1s and 0s).
3.  **Hardware:** The physical chip layout that receives and executes this binary code.

<p align ="center">
<img width="632" height="556" alt="image" src="https://github.com/user-attachments/assets/14abc3c1-469b-4f0d-aa20-bfeaba476805" />
</p>

## The ISA: Bridging Software and Hardware

The **Instruction Set Architecture (ISA)** is the critical interface between software and hardware. It defines the set of all commands (instructions) that the hardware is guaranteed to understand. For this project, we use **RISC-V**.

A compiler's job is to take a high-level C program and translate it into this RISC-V instruction list. The image provided shows a C-code stopwatch on the left and the corresponding RISC-V assembly output on the right (e.g., `lui`, `addi`, `sd`, `jalr`).

<p align ="center">
<img width="1024" height="576" alt="image" src="https://github.com/user-attachments/assets/f798daf1-6d72-4f64-88d5-3e3f8a69aada" />
</p>

## Implementation: From ISA to Silicon

The final step is understanding how the hardware *implements* the ISA. This is where **synthesizable Verilog** becomes essential.

1.  An **assembly instruction** (like `add x6, x10, x6`) is just a command.
2.  We write an **RTL snippet** (e.g., a Verilog module for a RISC-V core) that *describes the logic* to execute that command. This Verilog code is the "implementation" of the ISA.
3.  A **synthesis tool** (like Yosys) reads this Verilog code and converts it into a **Synthesized Netlist**—a structural map of all the basic logic gates (standard cells) and flip-flops needed.
4.  This netlist is then fed into the physical design tools (like OpenROAD) to create the final **Hardware Layout**, which is the physical arrangement of transistors and wires on the silicon die that brings the Verilog code to life.

<p align ="center">
<img width="632" height="556" alt="image" src="https://github.com/user-attachments/assets/52b08833-a73c-413c-a94b-6054acc381ba" />
</p>

-----

## Open source ASIC implementation

In an ASIC flow, we need three key components, all of which are now available as open-source:

<img width="1773" height="964" alt="image" src="https://github.com/user-attachments/assets/8e2516c6-42ff-4fe5-9aa6-8f60f4dce224" />

1.  **RTL Design:** The digital design, written in a Hardware Description Language (HDL) like Verilog modelling the actual circuit. Open-source designs can be found on sites like `opencores.org` and `github.com`.
2.  **EDA Tools:** Electronic Design Automation tools are the software used to design and verify the chip. We are using an open-source flow called **OpenLANE**, which integrates tools like **OpenROAD** and **Yosys**.
3.  **PDK Data:** The **Process Design Kit (PDK)** is the essential data bridge between the foundry and the designer. It contains all the technology-specific files (design rules, standard cell libraries, I/O libraries) needed by the EDA tools to model the foundry's specific fabrication process.

In 2020, Google and SkyWater released the first open-source PDK, **Sky130** (130nm node). While not the most advanced node, 130nm is cost-effective and powerful—a pipelined RISC-V CPU on this node can still exceed a 1 GHz clock speed.

## Simplified RTL2GDS Flow

<img width="1183" height="632" alt="image" src="https://github.com/user-attachments/assets/2fa4c3db-bfa7-4eda-8e5c-561a3924ef80" />

OpenLANE automates the following 6-step flow to turn an RTL design into a GDSII layout file:

1.  **Synthesis (Yosys + abc):** Translates the abstract digital design from RTL (Verilog) into a gate-level netlist, which is a mapping of all the specific logic gates (AND, OR, flip-flops) from the PDK's standard cell library.
   
   <img width="1772" height="959" alt="image" src="https://github.com/user-attachments/assets/845fb231-7fa6-404e-a33e-41858ae690fb" />

2.  **Floor Planning and Power Planning (OpenROAD):** Defines the chip's total area (the floorplan), places the I/O pads, and creates the **Power Distribution Network (PDN)**. The PDN is a grid of horizontal and vertical metal straps for VDD (power) and GND (ground).
   
   <img width="1782" height="941" alt="image" src="https://github.com/user-attachments/assets/b6434939-e1e3-4e88-80e4-b152d3e4d25c" />

3.  **Placement:** Takes the netlist from synthesis and places each standard cell onto the rows defined in the floorplan. This is done in two stages:
      * **Global Placement:** Finds a rough, optimized location for all cells, focusing on timing and congestion. The connected cells are placed together to reduce the timing constraint.
      * **Detailed Placement:** Snaps each cell to its exact, legal position on the placement grid. Removes any overlap and places the cell in the rows.

      <img width="1773" height="956" alt="image" src="https://github.com/user-attachments/assets/5e5a1a01-5dc7-4506-909a-dade7de06892" />

4.  **Clock Tree Synthesis (OpenROAD):** Builds the network of buffers and inverters needed to distribute the clock signal to all sequential elements (like flip-flops). The goal is to ensure the clock arrives at every flip-flop at the same time, minimizing **clock skew**.

   <img width="1774" height="940" alt="image" src="https://github.com/user-attachments/assets/51539e98-edba-4941-affa-fe1578fbb238" />

5.  **Routing (Triton Route):** Physically connects all the pins of the placed cells using the metal layers defined in the PDK.
      * **Global Routing:** Generates "routing guides," which are a high-level plan for all the wires, avoiding congestion.
      * **Detailed Routing:** Uses the guides to lay down the actual metal traces and vias to complete all connections.

   <img width="1785" height="978" alt="image" src="https://github.com/user-attachments/assets/8c714538-5b5b-4c65-9d4f-be676b4e847d" />

   <img width="1306" height="1074" alt="image" src="https://github.com/user-attachments/assets/fa2ab8d8-70c6-40b4-9d5e-4ad32b58fc83" />

7.  **Sign Off (OpenSTA, magic and netgen):** The final verification stage to ensure the layout is correct and manufacturable. This involves:
      * **Physical Verification:** Running **DRC** (Design Rule Checking) and **LVS** (Layout vs. Schematic).
      * **Timing Verification:** Running **STA** (Static Timing Analysis).

   <img width="1783" height="938" alt="image" src="https://github.com/user-attachments/assets/6f6620c8-170a-45b4-92cb-7a74d7c11a56" />

## Introduction to OpenLANE

**OpenLANE** is an automated RTL-to-GDSII flow that integrates all the necessary open-source tools (Yosys, OpenROAD, Magic, etc.) to achieve a "no-human-in-the-loop" design. The goal is to produce a **clean GDSII** file, one with:

  * No DRC Violations
  * No LVS Violations
  * No Timing Violations

It is tuned for the SkyWater 130nm PDK and can be run in two modes:

  * **Autonomous:** A mode that runs the entire flow from start to finish.
  * **Interactive:** A step-by-step mode that allows us to run commands one by one, which is what we will be using for these labs.

## OpenLANE Detailed Flow and Concepts

<img width="815" height="467" alt="image" src="https://github.com/user-attachments/assets/07cced8c-6532-4148-b93a-7c13fbf465b0" />

The OpenLANE flow includes several critical sub-steps other than the ones mentioned before:

  * **Static Timing Analysis (OpenSTA):** After routing, RC (resistance/capacitance) parasitic data is extracted from the layout and used by OpenSTA (in OpenROAD) to verify the design meets its timing (e.g., the clock period).
  * **Design for Test (Fault):** It perform scan inserption, automatic test pattern generation, Test patterns compaction, Fault coverage, Fault simulation.
  * **Antenna Rules Violation:** During fabrication, long, unconnected metal wires can act as antennas and collect static charge, which can destroy transistor gates. To prevent this, routers must limit wire length. OpenLANE has a different solution:
    1.  It preventively adds "fake" antenna diode cells next to cell inputs where the antenna effect might occur after placement.
    2.  After routing, it runs an antenna checker.
    3.  If a violation is reported, it replaces only the necessary *fake* diode with a *real* antenna diode cell, which safely leaks away any collected charge.
  * **LCE (Yosys):** This is used to formally confirm that the function did not change after modifying the netlist.
  * **RC Extraction:** Generates a .spef file which is modelling of parasitic resistance and capacitances in the wire.
  * **Physical Verification (DRC/LVS):** **Magic** is used for DRC and to extract a SPICE netlist from the final layout. **Netgen** is then used to perform LVS, comparing the layout's SPICE netlist against the original synthesized netlist to ensure they match.

