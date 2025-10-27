# Sky130 Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK

This first day serves as a high-level introduction to the entire chip design ecosystem, from the physical hardware components to the abstract software that controls them.

---

### The Main Parts of a Chip

A chip isn't just one monolithic block; it's a hierarchical system of components. We start from the "outside-in" view.

1.  **Package and Pads:** The chip is housed in a package, such as the **QFN-48** (Quad Flat No-leads, 48 pins). The external pins of this package connect to the **PADS** on the perimeter of the silicon **Die**.
2.  **Die:** This is the actual square of silicon containing all the circuitry.
3.  **Core:** This is the functional "brain" of the chip, which sits inside the ring of I/O pads.

The **Core** itself is further broken down:

* **Macros:** These are large, complex, pre-designed digital blocks. In this design, they include the main **RISC-V SoC**, the **SRAM** (memory), and the **gpio bank**.
* **Foundry IPs (Intellectual Property):** These are specialized, often analog or mixed-signal, blocks provided by the foundry (e.g., Skywater). The key IPs here are the **ADC** (Analog-to-Digital Converter) and **DAC** (Digital-to-Analog Converter).


---

### High-Level Computer Workflow

At a high level, a computer's operation is a translation from human-readable software to machine-executable hardware operations.

1.  **Application Software:** The programs a user interacts with (e.g., a web browser or word processor).
2.  **System Software:** This is the layer that translates application requests into machine code. It includes:
    * **OS (Operating System):** Manages the hardware, allocates memory, and handles I/O.
    * **Compiler:** Converts high-level code (like C or C++) into assembly language instructions.
    * **Assembler:** Converts the human-readable assembly instructions into binary machine code (1s and 0s).
3.  **Hardware:** The physical chip layout that receives and executes this binary code.


---

### The ISA: Bridging Software and Hardware

The **Instruction Set Architecture (ISA)** is the critical interface between software and hardware. It defines the set of all commands (instructions) that the hardware is guaranteed to understand. For this project, we use **RISC-V**.

A compiler's job is to take a high-level C program and translate it into this RISC-V instruction list. The image provided shows a C-code stopwatch on the left and the corresponding RISC-V assembly output on the right (e.g., `lui`, `addi`, `sd`, `jalr`).


---

### Implementation: From ISA to Silicon

The final step is understanding how the hardware *implements* the ISA. This is where **synthesizable Verilog** becomes essential.

1.  An **assembly instruction** (like `add x6, x10, x6`) is just a command.
2.  We write an **RTL snippet** (e.g., a Verilog module for a RISC-V core) that *describes the logic* to execute that command. This Verilog code is the "implementation" of the ISA.
3.  A **synthesis tool** (like Yosys) reads this Verilog code and converts it into a **Synthesized Netlist**—a structural map of all the basic logic gates (standard cells) and flip-flops needed.
4.  This netlist is then fed into the physical design tools (like OpenROAD) to create the final **Hardware Layout**, which is the physical arrangement of transistors and wires on the silicon die that brings the Verilog code to life.


