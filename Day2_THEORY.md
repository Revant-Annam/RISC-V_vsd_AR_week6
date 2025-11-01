# Sky130 Day 2 - Good floorplan vs bad floorplan and introduction to library cells

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

## Library binding and Placement

### Bind Netlist with Physical Cells

Consider the netlist of gates where each gate's shape represents its function. For instance, a NOT gate appears triangular, but in reality, it's a box with defined width and height. 
Similarly, standard cell are square or rectangular boxes. We assign physical dimensions to all gates and flip-flops, giving each netlist component a specific shape and size, as real-world components don't 
exist as symbolic shapes—they're all rectangular blocks with proper dimensions.

<img width="1027" height="671" alt="image" src="https://github.com/user-attachments/assets/bfda9a60-8a1d-4596-8dc6-bb3df68b343f" />

After removing wires, all gates, flip-flops, and blocks reside in a collection called the Library.

A library contains all available cells, similar to books in a collection. It includes timing data like gate delays. Libraries can be divided into sub-categories:
one containing physical shape and size information, another with delay information. Libraries offer multiple variants of each cell—larger cells have lower resistance paths, 
enabling faster operation with reduced delay. Selection depends on timing requirements and available floorplan space.

<img width="1297" height="517" alt="image" src="https://github.com/user-attachments/assets/bf52281e-2204-4a1b-b316-879b8aa27a6e" />

### Placement

After assigning proper dimensions to gates, we position these components on the floorplan. With the floorplan's input/output ports defined, the netlist specified, and each component sized, 
we have the physical representation of logic gates. The next phase involves placing the netlist onto the floorplan, using connectivity information to position physical gates appropriately.

The floorplan contains pre-placed cells from earlier stages. Placement ensures these pre-placed cell locations remain unchanged and prevents any cell placement over them. The physical netlist must be positioned
to maintain logical connectivity while enabling circuit interaction with input/output ports for optimal timing and minimal delay.

Initially, we arrange remaining netlist components on the floorplan, positioning elements close to their respective input/output pins. However, some connections like FF1 of Stage 4 to Din4 remain distant. 
Placement optimization addresses this issue.

<img width="1305" height="633" alt="image" src="https://github.com/user-attachments/assets/82fe5d46-0f81-4603-9277-46906ce47604" />

### Optimize Placement Using Estimated Wire-Length and Capacitance

**Optimize Placement**: Optimization resolves distance-related issues. Consider the FF1 to Din2 connection. Before routing or wiring, we estimate capacitances. The capacitance from Din2 to FF1 is substantial
due to long wire length, causing high resistance. Transmitting signals over such distances makes reception at FF1 difficult. We insert intermediate elements to preserve signal integrity,
successfully driving the input from Din2 to FF1. These intermediate elements are Repeaters—buffers that restore the original signal, create a replicated signal, and forward it. 
This process continues until reaching the destination cell, maintaining signal integrity. While repeaters solve signal integrity issues, they consume additional floorplan area.

Stage 1 requires no repeaters for signal transmission. 

<img width="1297" height="637" alt="image" src="https://github.com/user-attachments/assets/6c347e43-aa6b-408c-8b17-00dd15345661" />

Stage 2, however, needs repeaters due to long wire length preventing signal transmission within acceptable range. Similar to Stage 2, Stage 3 requires buffer between gate2 and FF2.

<img width="1295" height="622" alt="image" src="https://github.com/user-attachments/assets/bd091953-47d9-4e49-8ca6-2f6a01f5e017" />

Stage 4 presents more complexity with multiple buffers and the wire estimates running over the standard cells. We must verify our work through timing analysis using ideal clocks.
This analysis data confirms whether placement is correct.

<img width="1306" height="631" alt="image" src="https://github.com/user-attachments/assets/50610928-8509-4c6c-b114-862d03fc6cd4" />

### Need for Libraries and Characterization

Every IC design flow progresses through multiple stages. 
* First is Logic Synthesis—converting RTL-coded functionality into valid hardware. Logic synthesis output is a gate arrangement representing the original RTL-described functionality.
* Following logic synthesis is Floorplanning, where we import synthesis output and determine Core and Die sizes.
* Next is Placement, positioning logic cells on the chip for optimal initial timing.
* Then comes CTS (Clock Tree Synthesis), ensuring clock signals reach all points simultaneously with equal rise and fall times.
* Next is Routing, which depends on flip-flop characterization.
* Finally, STA (Static Timing Analysis) examines setup time, hold time, and maximum achievable circuit frequency.
The common element across all stages: Gates or Cells. The collection of the cells or gates is called the library and if the proper characterization of the library is not done then all the stages gets affected.

# Cell Design and Characterization Flows

## Inputs for Cell Design Flow

In Cell Design Flow, components like gates, flip-flops, and buffers are termed 'Standard Cells'. These standard cells reside in a collection called the 'Library', which contains multiple cells with different 
functionality and also multiple cells with identical functionality but varying sizes. The standard cells also vary with the threshold voltage. 

<img width="1122" height="718" alt="image" src="https://github.com/user-attachments/assets/9fb81de1-7119-4a13-88d9-47fc66145dee" />

Examining an inverter from the library, the cell design flow proceeds as follows:

The inverter must be represented through its shape, drive strength, power characteristics, and more. Cell design flow comprises three components:

Inputs

Design steps

Outputs

**1) Inputs**: Cell design requires PDKs, DRC and LVS rules, SPICE models, library specifications, and user-defined specifications. DRC & LVS rules provide tech files containing design rules and actual values 
that can be coded. SPICE models describe threshold voltage equations. User defined specifications like the cell height and width, supply voltage, metal layers. Power rail and ground rail separation determines cell height.
Cell width depends on timing requirements and drive strength.

<img width="1111" height="650" alt="image" src="https://github.com/user-attachments/assets/e7dc5884-6318-413b-949b-abad73727b26" />

**2) Design Steps**: Design encompasses three phases: circuit design, layout design, and characterization.

Circuit Design involves two phases: First, implementing the function itself; second, modeling PMOS and NMOS transistors to meet library requirements. The main condition is that the sum of the drain currents is 0.
With that we can model various parameters of the cell with the required switching theshold. These are done using SPICE simulations.

**3) Outputs**

Typical circuit design outputs include CDL (circuit description language) file, GDSII, LEF, and extracted spice netlist (.cir).

<img width="1122" height="668" alt="image" src="https://github.com/user-attachments/assets/3116f911-e688-49b0-9e10-f32a32312027" />

### Layout Design Step

In Layout Design, the first phase implements the function using MOS transistors through PMOS and NMOS transistor sets. The second phase extracts PMOS and NMOS network graphs from the implemented design.

<img width="1127" height="673" alt="image" src="https://github.com/user-attachments/assets/79668f18-c7c1-4193-936d-34ba69717d12" />

After obtaining network graphs, the next phase determines Euler's path—a path traced only once.

<img width="1142" height="673" alt="image" src="https://github.com/user-attachments/assets/e149675e-e408-443f-ba05-d5dfc8004909" />

Following this, create a stick diagram based on Euler's path, derived from the circuit diagram.

<img width="735" height="395" alt="image" src="https://github.com/user-attachments/assets/3b7d45e7-c1bb-4881-8ead-56616b714967" />

Next, convert the stick diagram into a proper layout following previously discussed rules. Once the layout is complete, specifications like cell width, cell length, drain current, and pin locations are established.

<img width="423" height="482" alt="image" src="https://github.com/user-attachments/assets/886de9d1-0129-4a75-907b-bcfdad431bc4" />

The final phase extracts parasitics from the layout and characterizes it for timing. The layout design output is GDSII. After obtaining the extracted spice netlist, characterization provides timing, noise,
and power information.

### Typical Characterization Flow

Building the characterization flow from available inputs:

1. Read the model
2. Read the extracted spice netlist
3. Define or recognize buffer behavior
4. Read inverter subcircuits
5. Attach necessary power supplies
6. Apply stimulus
7. Provide necessary output capacitance
8. Provide simulation commands (e.g., .tran for transient simulation, .dc for DC simulation).

<img width="1147" height="633" alt="image" src="https://github.com/user-attachments/assets/e8d9256a-9456-43a6-808b-f440256e8261" />

Next, compile inputs 1 through 8 into a configuration file for the characterization software "GUNA".

<img width="1173" height="435" alt="image" src="https://github.com/user-attachments/assets/ee867712-badc-4bd8-8974-19c268fd15c7" />

This software generates power, noise, and timing models.

### Timing Threshold Definitions

With back-to-back connected inverters, power sources, and applied stimulus, understanding different waveform threshold points becomes crucial—termed Timing threshold definitions.

The term 'Slew_low_rise_thr' represents values near 0, typically around 20% or possibly 30%.

<img width="1191" height="647" alt="image" src="https://github.com/user-attachments/assets/8a624557-4cb5-4680-bc94-81db0f774d2b" />

Slew_high_rise_thr

<img width="986" height="595" alt="image" src="https://github.com/user-attachments/assets/14ff63ec-b22b-421c-8cb9-8f06c439a355" />

Slew_low_fall_thr

<img width="983" height="602" alt="image" src="https://github.com/user-attachments/assets/cd6801b6-e36c-415c-825a-61e60686151a" />

Slew_high_fall_thr

<img width="972" height="605" alt="image" src="https://github.com/user-attachments/assets/eaaf9dc1-8375-45b8-b79b-d57e89c5661d" />

Examining the input stimulus waveform (first buffer input) alongside the first buffer output, delay thresholds are also available, similar to slew. Extract rise and fall points from waveforms, 
with thresholds typically around 50%.

in_rise_thr

<img width="1006" height="595" alt="image" src="https://github.com/user-attachments/assets/4a02f87a-5cf0-4c2e-9ff3-e6e812bb95b0" />

in_fall_thr, typically valued at 50%

<img width="987" height="598" alt="image" src="https://github.com/user-attachments/assets/f6fbb7a8-6a1d-4076-894e-3ea8ffe4d6cf" />

out_rise_thr

<img width="992" height="603" alt="image" src="https://github.com/user-attachments/assets/da082832-178a-4c20-9b82-c320dd70b108" />

out_fall_thr

<img width="1022" height="602" alt="image" src="https://github.com/user-attachments/assets/cc1c7a8e-78d5-497d-bf06-469926a8519a" />

### Propagation Delay and Transition Time

These values enable calculation of further parameters like propagation delay, current, and slews.

Delay calculation requires subtracting in_rise_thr from out_rise_thr. Using a typical 50% value, examining the waveform: 

<p align = "center"> Time delay = Time(out_thr) - time(in_thr). </p>

<img width="1156" height="615" alt="image" src="https://github.com/user-attachments/assets/8a4987d0-a630-4674-80da-1651831a9eeb" />

In the example with in_rise_thr and out_fall_thr at 50%, if the threshold point shifts upward, output precedes input, resulting in negative delay—which is unacceptable. Negative delay stems from poor threshold
point selection, emphasizing the importance of proper threshold point choice.

<img width="1180" height="616" alt="image" src="https://github.com/user-attachments/assets/1a2cdc00-edb1-46dc-aea2-a573f8ff2c7f" />

Another example shows correctly chosen threshold points still yielding negative delay because output precedes input—also unacceptable.

<img width="1177" height="612" alt="image" src="https://github.com/user-attachments/assets/81a178e8-d502-44a4-b968-4444fe5a161a" />

<p align = "center"> Transition time = time(slew_high_rise_thr) - time(slew_low_rise_thr) </p>

or

<p align = "center"> Transition time = time(slew_high_fall_thr) - time(slew_low_fall_thr) </p>

Examining a waveform demonstrates slew calculation.

<img width="1201" height="576" alt="image" src="https://github.com/user-attachments/assets/535cc8a8-9fb0-49d5-86f0-4a581f86f88e" />
