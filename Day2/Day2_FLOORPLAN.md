# Sky130 Day 2 - Floorplan

## Chip Floor Planning Strategy

The main steps of the Floorplan include:
1. **Define width and height of core and die**
2. **Define locations of pre placed cells**
3. **Decoupling capacitors**
4. **Power planning**
5. **Pin placement**
6. **Logical cell placement blockage**
   
### 1. **Define width and height of core and die**

This section explores determining the core and die dimensions, which represents the initial phase of the physical design workflow. We'll start with a netlist—a configuration featuring two flip-flops connected
through basic combinational logic. A netlist defines how components within an electronic design interconnect. The dimensions of logic gates (AND & OR) and flip-flops are crucial for our calculations. 
First transform these symbolic representations into actual physical measurements.

<img width="868" height="640" alt="image" src="https://github.com/user-attachments/assets/5e97ac71-42ec-42de-b152-f073d3d54c40" />

Consider standard cells with area = 1 square unit. Assume the same area for flip-flops = 1 square unit. Using these measurements and the netlist information, we'll determine the silicon wafer area required
by the netlist.

Consider a silicon wafer where all logic elements are implemented. One section is designated as the 'Die', which contains the Core within it. A Die, containing the core, is a compact semiconductor material
sample on which essential circuits are fabricated. The 'Core' represents the chip section where the design's primary logic resides. 

Now, assume the core area is 4 square unit, which is enough for the logic to fit within the core, if the netlist occupies the entire core area, it achieves 100% utilization. This allows us to compute the utilization factor:

<p align = "center"> 
Utilization Factor = Netlist occupied area / Core's total area
</p>

Applying our dimensions:

<p align = "center"> Utilization factor = 4×1 sq. unit / 2 unit × 2 unit = 4 sq unit / 4 sq unit </p>

Therefore, utilization factor = 1 (indicating complete core area utilization with no remaining space)

<p align = "center"> Aspect Ratio = Height / Width = 2 unit / 2 unit = 1 </p>

An Aspect Ratio of 1 indicates a square-shaped chip. Any other value indicates a rectangular configuration.

### 2. **Define locations of pre placed cells**

Consider combinational logic performing specific functions—imagine a large circuit with N logic gates that we'll partition into smaller sections. We'll divide the circuit into two segments, implementing each block independently.

For both blocks, we'll extend input/output pins, then black box and isolate them. After black boxing, the internal structure becomes invisible to anyone viewing the main netlist. These become separate intellectual properties (IPs) or modules.

The advantage is reusability—implement once, use multiple times. Similar IPs exist, including memory blocks, clock-gating cells, comparators, and multiplexers—all part of the top-level netlist. They receive signals, execute functions, and generate outputs, with functionality implemented only once.

Positioning these IPs within a chip is called floorplanning.

These IPs occupy user-specified locations and are positioned before placement and routing, which is the reason why they are called pre-placed cells or macros.

These cells are positioned so that placement and routing tools don't modify their locations.

<img width="1127" height="603" alt="image" src="https://github.com/user-attachments/assets/3f313bbc-4b0f-41e4-b8b9-388a02a277c7" />

<img width="972" height="452" alt="image" src="https://github.com/user-attachments/assets/d241f286-3092-41a7-978d-d909ca2416a1" />

### De-coupling Capacitors

Consider a circuit segment from previously described blocks. When gates (like an AND gate) transition between states (0 to 1 or 1 to 0), switching current is required due to load capacitance.
This capacitor must fully charge for logic 1 and completely discharge for logic 0. With defined Rdd, Ldd, and Rss, Lss values, during switching, the circuit demands peak current. 
Due to Rdd and Ldd presence, voltage drops occur, causing Node 'A' voltage to be Vdd' rather than Vdd. Vdd' is lesser than Vdd

For signals to qualify as Logic '0' and '1', they must fall within noise margin low and noise margin high ranges. 
As Vdd' is less than Vdd we place a de-coupling capacitor parallel to the circuit which is charged to Vdd. During each switching event, current draws from this capacitor, 
while the RL network replenishes its charge. The de-coupling capacitor supplies the circuit's current requirements.

On the chip, decoupling capacitors are positioned between blocks a, b, and c. This ensures these blocks receive power from the de-coupling capacitors, addressing local communication needs.

<img width="1037" height="627" alt="image" src="https://github.com/user-attachments/assets/20f1a2a2-1844-43e4-ab18-b4f24bf87005" />

<img width="970" height="605" alt="image" src="https://github.com/user-attachments/assets/429c7493-d41a-4f31-9aea-3bdaed9409a8" />

<img width="860" height="672" alt="image" src="https://github.com/user-attachments/assets/5ed36159-47f7-4567-8dfc-4f26e9ec7c99" />

### Power Planning

Consider the local circuitry as a black box that repeats multiple times, with logic at the boundaries. While de-coupling capacitors solved local current demands, signals must travel from driver to load
(logic 0 to logic 1 transitions). The driver-to-load line must maintain signal integrity for proper load reception. With power supply applied, a 16-bit bus must maintain consistent signals from driver to load,
requiring adequate power. However, placing capacitors everywhere isn't feasible. With power supply distant from the bus, voltage drops occur.

When a 16-bit bus line reads logic 1, its capacitor charges to Vdd; for logic 0, it discharges to ground. Consider this bus connected to an inverter, causing all initially charged capacitors to discharge and 
vice versa.

Problems arise when all capacitors connect to a single ground point. This creates a voltage bump at the ground tap. If this bump exceeds noise margin limits, it enters an undefined state,
potentially registering as either logic 1 or 0, creating unpredictability.

Similarly, capacitors at 0 volts charging to V volts through a single Vdd tap point causes voltage lowering at that point. This remains acceptable within noise margin levels but becomes problematic in 
undefined regions.

This supply voltage lowering occurs because power applies at only one point. The solution employs multiple power supplies. Each block draws charge from its nearest power supply and discharges to the 
nearest ground—creating a mesh power supply structure.

The power planning configuration is illustrated below.

<img width="957" height="607" alt="image" src="https://github.com/user-attachments/assets/9d818974-ef55-4120-bab0-561626d1668e" />

<img width="988" height="617" alt="image" src="https://github.com/user-attachments/assets/04d85931-a0bd-4b11-af96-8ed687df643d" />

<img width="895" height="697" alt="image" src="https://github.com/user-attachments/assets/e84ed06a-3110-4be3-b9dc-a193491a0c02" />

<img width="1117" height="661" alt="image" src="https://github.com/user-attachments/assets/df77164c-d42b-402c-91c6-d742526ec99d" />

### Pin Placement and Logical Cell Placement Blockage

**Pin Placement Strategy**

Consider different designs requiring implementation. The first circuit operates with clk1, the second with clk2, each having distinct inputs (Din1 and Din2) and outputs (Dout1 and Dout2).
Pre-placed cells include Block A (receiving Din1 and Din2 inputs) and Block B (receiving clk1 and clk2, outputting a clock signal). Currently, we have 4 input ports (Din1, Din2, Clk1, Clk2) 
and 3 output ports (Dout1, ClkOut, Dout2).

<img width="902" height="577" alt="image" src="https://github.com/user-attachments/assets/cf7c774c-07bb-40a1-b284-6d62ed863db8" />

<img width="791" height="731" alt="image" src="https://github.com/user-attachments/assets/58aa9e77-5c80-4d60-bebe-9ebe2091ab86" />

Consider an additional design for implementation—such circuits help understand inter-clock timing analysis. The complete design now features 6 input ports and 5 output ports.
Gate connectivity information is coded in VHDL/Verilog, called the 'Netlist'.

Placing this netlist in our previously designed core, we'll populate the empty area between core and die with pin information. The frontend team determines netlist connectivity and input/output specifications,
while the backend team handles pin placement. Based on pin placement, we position preplaced blocks near their corresponding inputs.

Notice that clock input and output pins are larger than standard input/output pins. This is because input clocks continuously supply signals to all chip elements, and output clocks must transmit signals rapidly.
Clocks require minimal resistance paths—larger size means lower resistance.

Additionally, the pin placement area must be blocked for routing and cell placement purposes. This logical cell placement blockage appears in the image between pins.

With this complete, the floor plan is prepared for the Placement and Routing phase.

<img width="1100" height="687" alt="image" src="https://github.com/user-attachments/assets/cbe6625e-2ed1-47f3-bf60-4d4950a1b0d1" />

-----

## Floorplanning Configuration in OpenLANE

Before running the floorplan, OpenLANE relies on several configuration variables. By default, these are set to a **`CORE_UTILIZATION`** of 50% and an **`ASPECT_RATIO`** of 1 (creating a square core).

These default values are not fixed and can be overridden in the configuration file to meet specific design requirements. Other critical settings defined by default include:

  * **`FP_PDN_...`** variables: These automatically set up the power distribution network (PDN).
  * **`FP_IO_MODE`**: This controls how I/O pins are placed. The default (mode 1, 0) signifies that pins will be placed at equal distances from each other but in a random order.

OpenLANE uses a priority system for loading these settings. The PDK variant's configuration file (`sky130A_sky130_fd_sc_hd_config.tcl`) has the **highest priority**,
followed by the design-specific `config.tcl`. The flow's system default (`floorplanning.tcl`) has the **lowest priority**.

### Review floorplan files and steps to view floorplan

After `run_floorplan` completes, we can review the output files. Inside the specific run directory (`design/picorv32a/runs/date_time/`), the `config.tcl` file is updated to show all the final parameters 
that were actually used for the flow.

The main physical output is located in the `runs/date_time/results/floorplan/` directory as a **.def (Design Exchange Format)** file (e.g., `picorv32a.floorplan.def`). 
This is a text file that contains the physical layout data.

If we open the `.def` file, we can find key information:

  * **`DIEAREA`**: This defines the chip's boundaries, for example, `(0 0) (660685 671405)`.
  * **`UNITS DISTANCE MICRONS 1000`**: This is a critical line. It states that 1000 database units are equal to 1 micron ($\mu m$).

Using this, we can calculate the chip's physical dimensions:

  * **Chip Width:** $660,685 \text{ units} / 1000 = 660.685 \; \mu m$
  * **Chip Height:** $671,405 \text{ units} / 1000 = 671.405 \; \mu m$

To visually inspect the layout, we use **Magic**. The following command loads the sky130 technology file, the merged LEF (which defines the cells), and our newly created DEF file (which places the cells and pins):

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def
```

### Review floorplan layout in Magic

Once Magic opens, we can see the floorplan layout. This includes the core area with the standard cell rows, the I/O pins, and the power rings. We can observe that the I/O pins are placed at equal distances around the core's perimeter.

We can navigate the layout in Magic with these key commands:

  * **Select:** Click on an object and press `s`.
  * **Fit to screen:** Fits the layout to screen.
  * **Zoom In:** Click on a location and press `z`.
  * **Zoom Out:** Press `Shift+Z`.

To get information about a selected object, we can open the **tkcon window** (Magic's console) and type the `what` command. This reveals details about the object:

  * Selecting a **horizontal pin** and using `what` shows that it is on the **metal 3** layer.
  * Selecting a **vertical pin** and using `what` shows it is on the **metal 2** layer.

We can also see the **Decap cells** that have been placed along the border of the core, adjacent to the standard cell rows, to ensure power stability.
The standard cell rows themselves (e.g., for mux, AND gates, etc.) are now defined and ready to be populated during the placement stage.
