# Day 5 - Final steps for RTL2GDS using tritonRoute and openSTA

## Routing and design rule check (DRC)

The final stage in the physical design flow is **Routing** and **DRC**. Routing is the process of finding the best and shortest possible connection between two endpoints (a source and a target). The goal is to 
create these connections with the fewest possible twists and turns.

### Introduction to Maze Routing – Lee’s algorithm

A fundamental method for routing is **Maze Routing**, which uses **Lee's Algorithm**. This algorithm seeks to avoid complex zig-zag paths, preferring simple "L" or "Z" shaped connections.

The algorithm works by first creating a **routing grid** over the layout. To find the optimal path between a 'Source' point and a 'Target' point on this grid, the algorithm begins by labeling all adjacent grid
cells. This expansion wave only moves horizontally and vertically (not diagonally).

<img width="772" height="626" alt="image" src="https://github.com/user-attachments/assets/070c1f12-4cc2-41ce-98a5-8e34cbb83b4a" />

### Lee’s Algorithm conclusion

The algorithm continues labeling the grid with the next integer in the sequence (2, 3, 4...) until the expansion wave reaches the 'Target'. For instance, if the target is reached with the integer 9, it signifies
that the shortest path is 9 grid units long.

<img width="778" height="631" alt="image" src="https://github.com/user-attachments/assets/67231c93-b5cc-4c50-85de-27844db1b499" />

While many paths to the target might exist, the tool traces back from the target to the source (by following the decreasing integers) to identify the shortest possible path. This process inherently avoids 
zig-zags and favors "L" shaped routes (less bends).

<img width="751" height="618" alt="image" src="https://github.com/user-attachments/assets/aef1c11d-b00f-4081-8dd1-4d04a660e648" />

<img width="872" height="627" alt="image" src="https://github.com/user-attachments/assets/c35570c0-8883-4d8a-894b-46a753643cf0" />

### Design Rule Check

After routing is complete, the design must pass a **Design Rule Check (DRC)**, a process often called "DRC cleaning." This step verifies that the physical layout meets all the foundry's rules for 
manufacturability.

<img width="906" height="637" alt="image" src="https://github.com/user-attachments/assets/12060991-0988-41ec-a0b5-d0edb10e2eb3" />

For example, if two wires run parallel, the rules specify the minimum required distance between them. Key rules include:

* **Rule 1) Wire Width:** The minimum width of a single wire, which is often derived from the wavelength of light used in the photolithography process.

  <img width="382" height="180" alt="image" src="https://github.com/user-attachments/assets/7ee95c99-c879-4109-b156-ceb204316e8d" />

* **Rule 2) Wire Pitch:** The minimum distance from the center of one wire to the center of an adjacent wire.

  <img width="381" height="192" alt="image" src="https://github.com/user-attachments/assets/8a8e5880-836e-41a2-a8b3-44d3cface9fb" />

* **Rule 3) Wire Spacing:** The minimum amount of empty space required between the edges of two wires.

  <img width="382" height="186" alt="image" src="https://github.com/user-attachments/assets/b0754628-a0b1-42df-8975-8be0ccc3492f" />

<img width="1272" height="663" alt="image" src="https://github.com/user-attachments/assets/7ba15376-68dd-465d-bdb0-3b1379a64837" />

If a signal short or spacing violation occurs, a common solution is to re-route one of the wires onto a different metal layer (often an upper layer, which may be wider). 

<img width="337" height="286" alt="image" src="https://github.com/user-attachments/assets/0ef33908-a438-4e63-a2f3-64984a7ecbb5" />

This introduces **vias** (vertical connections between layers), which come with their own set of DRC rules:

* **Rule 1) Via Width:** The minimum required width or diameter of the via.

  <img width="345" height="255" alt="image" src="https://github.com/user-attachments/assets/b84db042-077f-4584-abe2-b072057c1023" />

* **Rule 2) Via Spacing:** The minimum spacing required between two adjacent vias.

  <img width="342" height="260" alt="image" src="https://github.com/user-attachments/assets/c30653d2-feeb-46c1-9206-3c1bdd79fca6" />

Once routing is complete and the layout is "DRC clean," the next step is **Parasitic Extraction**. This step analyzes the final, complex layout and extracts the parasitic **resistance (R)** and 
**capacitance (C)** associated with every wire. This parasitic data is essential for the final, accurate timing and power analysis.

<img width="883" height="627" alt="image" src="https://github.com/user-attachments/assets/88c04bb6-89b9-4b90-980b-8a68aaf82a71" />

## Routing

The complete routing process is divided into two main parts:

1.  **FastRoute (Global Route):** This is the first stage. The entire routing region is divided into a grid of rectangular cells, often visualized as a 3D routing graph (including the different metal layers).
The `FastRoute` engine determines the general path or guides for each net (e.g., a net connecting pins A, B, C, and D) through this grid, focusing on avoiding congestion.
2.  **Detailed Route:** This is the second stage, handled by the `TritonRoute` engine, which follows the guides from the global router to create the exact, DRC-clean wire connections.

<img width="1170" height="515" alt="image" src="https://github.com/user-attachments/assets/c6fa4f44-20e0-4f00-ad57-470323a92367" />

### TritonRoute Features

**TritonRoute feature 1 - Honors pre-processed route guides**

TritonRoute performs the initial detailed route by strictly honoring the **route guides** provided by the global router (`FastRoute`).

For this to work, the route guides must meet certain requirements:
* They should have a "unit width."
* They must follow the **preferred routing direction** of that metal layer (e.g., horizontal for Metal 2, vertical for Metal 3).

<img width="1022" height="343" alt="image" src="https://github.com/user-attachments/assets/2720b825-f9f1-4fb0-8f2e-22656b11b056" />

`TritonRoute` assumes these guides already satisfy inter-guide connectivity. Two guides are considered connected if:
* They are on the **same metal layer** and their edges are touching.
* They are on **neighboring metal layers** and have a non-zero, vertically overlapped area (allowing for a via).

It uses a MILP (Mixed Integer Linear Programming) based scheme to route within defined "panels."

**TritonRoute Feature 2 & 3 - Inter-guide connectivity and intra- & inter-layer routing**

A key requirement is that every pin (or "terminal") of a standard cell must be **overlapped by a route guide**. This ensures that the detailed router has a clear "entry point" to connect to every pin. 
Placing pins at the intersection of horizontal and vertical tracks is a good way to guarantee they are covered by guides.

<img width="477" height="242" alt="image" src="https://github.com/user-attachments/assets/0bc36464-9a22-41fe-84f7-e06578ca674f" />

`TritonRoute` employs a method called **intra-layer parallel and inter-layer sequential panel routing**.
* **Intra-layer (within a layer):** The tool divides a metal layer (e.g., Metal 2, with a vertical preferred direction) into vertical "panels." It then routes *in parallel* within these panels. For example,
it might first route all the "even-indexed" panels simultaneously, and then route all the "odd-indexed" panels simultaneously.
* **Inter-layer (between layers):** It performs this intra-layer routing *sequentially*, moving from one metal layer to the next (e.g., finishing M2, then M3, then M4).

<img width="945" height="315" alt="image" src="https://github.com/user-attachments/assets/a154afdf-3440-4449-809e-a50125507d0f" />

### TritonRoute method to handle connectivity

`TritonRoute` takes the **LEF** file (for cell and layer info) and the **route guides** as its main inputs. Its goal is to produce a detailed routing solution (the **output DEF**) that honors these guides, 
maintains all connections, is DRC-clean, and optimizes for minimum wire length and via count.

To manage the complex connectivity, it uses a few key abstractions:
* **Access Point (AP):** A specific, on-grid point on a metal layer that is used as a connection point. It can connect to segments on lower layers, segments on upper layers, or pins.
* **Access Point Cluster (APC):** A group of all Access Points that are derived from the same common source (e.g., all the APs that connect to a single pin, or all the APs that connect to a single guide on a
 lower layer).

<img width="859" height="311" alt="image" src="https://github.com/user-attachments/assets/a3ab1aac-ea08-42a7-9045-f406a075921e" />

### Routing topology algorithm and final files list post-route

The routing algorithm works by determining the "cost" associated with each **Access Point Cluster (APC)**. It then calculates the **Minimum Spanning Tree (MST)** between all the APCs belonging to a single net.
This finds the lowest-cost (i.e., shortest) wiring path that successfully connects all the pins in that net.

<img width="636" height="356" alt="image" src="https://github.com/user-attachments/assets/5b462d86-9e10-41a2-b5fc-fe4fb34a897f" />
