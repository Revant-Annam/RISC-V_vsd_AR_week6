# Library Binding and Placement

## Netlist Binding and Initial Place Design

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

Here is the paraphrased summary of your notes on placement.

### Labs: Congestion-aware placement using RePlAce

For this run, the placement tool's primary constraint is **congestion**, not timing. The main goal is to place the cells in a way that reduces routing hotspots, ensuring the design can be successfully routed later.

Placement is executed in two distinct stages:

1.  **Global Placement:** This is the first step, often performed by an algorithm like RePlAce. Its main objective is to determine an optimal, rough location for every cell to **minimize total wire length** which
is found by the HPWL (Half Perimeter Width Length). At this stage, the placement is *not* legal; cells might overlap, and they are not snapped to the physical placement grid.
3.  **Detailed Placement:** Following global placement, this stage takes the rough placement and makes it *legal*. It moves the cells to their final, exact locations on the standard cell rows,
aligning them with the site grid and ensuring no cells overlap.

After running the placement, we can open the resulting DEF file in Magic to see the actual layout. The core area, which was empty after floorplanning, is now populated with thousands of standard cells, filling 
the rows.

If we zoom in, we can see the individual standard cells—such as buffers, logic gates, and flip-flops—all neatly arranged in their rows, ready for clock tree synthesis and routing.
