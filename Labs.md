# Week6 - Lab Documentation

This repository documents week 6 of the journey through the **RISC-V SoC Tapeout Program**, documenting all labs and key learnings.

## 🗂️ Table of Contents

* [Day 1 Labs: 'picorv32a' Synthesis & Flop Ratio](#day-1-labs-picorv32a-synthesis--flop-ratio)
* [Day 2 Labs: Floorplan & Placement](#day-2-labs-floorplan--placement)
* [Day 3 Labs: Custom Inverter Design & DRC Fix](#day-3-labs-custom-inverter-design--drc-fix)
* [Day 4 Labs: Standard Cell Integration & Timing ECO](#day-4-labs-standard-cell-integration--timing-eco)
* [Day 5 Labs: PDN, Routing & Post-Route STA](#day-5-labs-pdn-routing--post-route-sta)
* [Notes on Digital and Analog Block Interaction](#notes-on-digital-and-analog-block-interaction)
* [Observations on DRC, LVS, and STA Inter-dependencies](#observations-on-drc-lvs-and-sta-inter-dependencies)

## Day 1 Labs: 'picorv32a' Synthesis & Flop Ratio
This lab focuses on running the 'picorv32a' design through the OpenLANE synthesis flow and analyzing the resulting cell distribution.

### 1. Invoking openlane
**Commands:**
```bash
cd Desktop/work/tools/openlane_working_dir/openlane
docker
./flow.tcl -interactive
```
The prompt will be changed to a **'%'** symbol.

<img width="1280" height="768" alt="image" src="https://github.com/user-attachments/assets/b3d5e5be-d308-4f2a-a99c-08695816f17f" />

### 2. Run 'picorv32a' Design Synthesis

In this lab we are working on the `picorv32a` folder present in the `Desktop/work/tools/openlane_working_dir/openlane/designs/` directory.

<img width="1280" height="768" alt="image" src="https://github.com/user-attachments/assets/e4e41c36-d4c1-4c6c-81c8-446287d37777" />

In the `picorv32a` we can find the design files and the various TCL files.

<img width="1280" height="768" alt="image" src="https://github.com/user-attachments/assets/91829527-b4f8-47ed-8a22-543f99fba14a" />

Any changes made in the 'config.tcl' file will alter the openlane in-built settings.

<img width="1280" height="768" alt="image" src="https://github.com/user-attachments/assets/ae557bab-ebe4-4c6e-ac56-0b0d78fa3556" />

In the `conig.tcl` file we can see the command which is the creating the pdk specific TCL file and then sourcing the .tcl file:
`set filename $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1} {
        source $filename
}`

Changes made in `sky130A_sky130_fd_sc_hd_config.tcl` will affect the settings in the `config.tcl`.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/95fdcbaf-dfa5-483e-935a-66402fac8d4c" />

**Commands to be run in the '%' prompt for the synthesis of picorv32a:**
```bash
package require openlane 0.9
prep -design picorv32a
run_synthesis
exit
exit
````

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ac4e6266-6905-4202-8406-dfe506ed075d" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7d9fc647-c4b0-4332-b162-336d0b5cdfc0" />

These will create a `runs` folder in the `picorv32a` folder which contains the reports, results, logs, etc specific to the time when the `prep` command was ran.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7c42104e-af08-471b-aab0-4d5a2a6a9ee3" />

This is the netlist which has been created after the synthesis which is located in the `../picorv32a/runs/29-10_19-51/results/synthesis` folder:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e6a918cd-4d39-44db-a286-09767b641a3b" />

### 3\. Calculate the Flop Ratio

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d2e5fa4e-f795-4806-9d00-943b8067b069" />

**Calculation:**
$$ \text{Flop Ratio} = \frac{\text{Number of D Flip Flops}}{\text{Total Number of Cells}} = \frac{1613}{14876} \approx 0.1084 $$
$$ \text{Percentage of DFFs} = \text{Flop Ratio} \times 100 \approx 10.84 \text{ \%} $$

### Key Learnings (Day 1)

  * [Your key learning point 1...]

### Command Explanations (Day 1)

  * `docker`: Starts the OpenLANE Docker container.
  * `./flow.tcl -interactive`: Starts the OpenLANE flow in interactive mode.
  * `package require openlane 0.9`: Loads the necessary OpenLANE Tcl package.
  * `prep -design picorv32a`: Prepares the design directory and links configuration/source files.
  * `run_synthesis`: Runs the synthesis flow, converting RTL to a gate-level netlist.

-----

## Day 2 Labs: Floorplan & Placement

This lab covers the floorplanning and placement stages for the 'picorv32a' design.

### 1\. Run 'picorv32a' Design Floorplan & Placement

**Commands (in OpenLANE interactive shell):**

```tcl
# Assuming you have already run prep -design picorv32a and run_synthesis
run_floorplan
run_placement
```

### 2\. Calculate Die Area

From the `.def` file `DIEAREA ( 0 0 ) ( 660685 671405 )`:
$$ \text{Die Width} = \frac{660685}{1000} = 660.685 \text{ µm} $$
$$ \text{Die Height} = \frac{671405}{1000} = 671.405 \text{ µm} $$
$$ \text{Area} = 660.685 \times 671.405 \approx 443587.21 \text{ µm}^2 $$

### 3\. Explore Floorplan & Placement in Magic

**Commands (in new terminals):**

```bash
# To view Floorplan
cd .../results/floorplan/
magic -T ...sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &

# To view Placement
cd .../results/placement/
magic -T ...sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

### Key Learnings (Day 2)

  * [Your key learning point 1...]

### Command Explanations (Day 2)

  * `run_floorplan`: Defines the chip's core dimensions, places I/O pins, and inserts well taps/decap cells.
  * `magic -T ... lef read ... def read ... &`: Opens the Magic layout tool, reading the LEF (abstract library) and DEF (design placement) files.
  * `run_placement`: Legally places all standard cells within the core boundaries.

-----

## Day 3 Labs: Custom Inverter Design & DRC Fix

This lab involves analyzing a custom inverter standard cell, performing post-layout simulation, and correcting DRC rules in the Magic tech file.

### 1\. Clone & View Custom Inverter

**Commands:**

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
git clone [https://github.com/nickson-jose/vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign)
cd vsdstdcelldesign
cp .../sky130A.tech .
magic -T sky130A.tech sky130_inv.mag &
```

### 2\. SPICE Extraction of Inverter

**Commands (in tkcon window):**

```tcl
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```

### 3\. Post-Layout ngspice Simulations

**Commands:**

```bash
ngspice sky130_inv.spice
plot y vs time a
```

**Timing Calculations:**

  * **Rise Transition Time:** $T_{rise} = \mathbf{63.96 \text{ ps}}$
  * **Fall Transition Time:** $T_{fall} = \mathbf{41.9 \text{ ps}}$
  * **Rise Cell Delay:** $D_{rise} = \mathbf{61.36 \text{ ps}}$
  * **Fall Cell Delay:** $D_{fall} = \mathbf{20 \text{ ps}}$

### 4\. Fix DRC Rules in Magic Tech File

**Setup Commands:**

```bash
cd
wget [http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz](http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz)
tar xfz drc_tests.tgz
cd drc_tests
magic -d XR &
```

**Corrections Made:**

  * **`poly.9` (Simple Rule):** Fixed incorrect poly spacing rule.
  * **`difftap.2` (Simple Rule):** Fixed incorrect tap-to-diff spacing rule.
  * **`nwell.4` (Complex Rule):** Fixed rule to correctly check for n-well tap presence.

### Key Learnings (Day 3)

  * [Your key learning point 1...]

### Command Explanations (Day 3)

  * `git clone ...`: Copies a Git repository from a remote URL.
  * `magic ... .mag &`: Opens Magic and loads a `.mag` (Magic layout) file.
  * `extract all`: (Magic tkcon) Extracts connectivity from the drawn layout layers.
  * `ext2spice cthresh 0 rthresh 0`: (Magic tkcon) Enables parasitic capacitance extraction for SPICE.
  * `ext2spice`: (Magic tkcon) Converts the extracted layout into a SPICE netlist.
  * `ngspice ...`: Runs the ngspice circuit simulator.
  * `plot y vs time a`: (ngspice) Generates a plot of nodes 'y' and 'a' versus time.
  * `wget ...`: Downloads a file from a web URL.
  * `tar xfz ...`: Extracts a compressed `.tgz` archive.
  * `tech load ...`: (Magic tkcon) Loads or re-loads a technology file.
  * `drc check`: (Magic tkcon) Runs the Design Rule Checker.
  * `drc why`: (Magic tkcon) Reports the specific DRC violation under the cursor.

-----

## Day 4 Labs: Standard Cell Integration & Timing ECO

This lab covers integrating the custom inverter into the `picorv32a` flow, optimizing synthesis, and performing manual/automated timing analysis and fixes.

### 1\. Verify Standard Cell Layout Conditions

Verified the custom inverter layout against standard cell requirements (port alignment, width, height).

  * $$ \text{Width} = 1.38 \text{ µm} = 0.46 \text{ µm} \times 3 $$ (Verified)
  * $$ \text{Height} = 2.72 \text{ µm} = 0.34 \text{ µm} \times 8 $$ (Verified)

### 2\. Generate LEF from Layout

**Commands (in tkcon window):**

```tcl
save sky130_vsdinv.mag
lef write
```

### 3\. Copy LEF and LIB files to 'picorv32a' Source

**Commands (in bash terminal):**

```bash
cp sky130_vsdinv.lef .../picorv32a/src/
cp libs/sky130_fd_sc_hd__* .../picorv32a/src/
```

### 4\. Edit 'config.tcl' to Include Custom Cell

Added `set ::env(LIB_SYNTH)`, `set ::env(LIB_FASTEST)`, `set ::env(LIB_SLOWEST)`, `set ::env(LIB_TYPICAL)`, and `set ::env(EXTRA_LEFS)` to `config.tcl`.

### 5\. Run Synthesis & Optimize

**Commands (in OpenLANE interactive shell):**

```tcl
prep -design picorv32a -tag 29-10_19-51 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_STRATEGY) "DELAY 3"
set ::env(SYNTH_SIZING) 1
run_synthesis
```

**Result:** Area increased, but Worst Negative Slack (WNS) improved to 0.

### 6\. Run Floorplan & Placement with Custom Cell

**Commands (in OpenLANE interactive shell):**

```tcl
init_floorplan
place_io
tap_decap_or
run_placement
```

**Result:** The custom inverter was successfully placed and abutted.

### 7\. Post-Synthesis STA & Manual Timing ECO

Used a previous synthesis run with violations to practice manual ECO.
**Commands (in bash terminal):**

```bash
sta pre_sta.conf
```

**Commands (in STA shell):**

```tcl
report_net -connections _11672_
replace_cell _14510_ sky130_fd_sc_hd__or3_4
report_checks -fields {net cap slew input_pins} -digits 4
write_verilog .../picorv32a.synthesis.v
exit
```

**Result:** WNS improved from -23.9000 ns to -22.6173 ns.

### 8\. Run PnR & Post-CTS Timing Analysis

Ran the full PnR flow on the *clean* (0-violation) design and performed Post-CTS STA.
**Commands (in OpenLANE interactive shell):**

```tcl
# ... (prep, synthesis, floorplan, placement) ...
run_cts
openroad
read_lef .../merged.lef
read_def .../picorv32a.cts.def
read_verilog .../picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc .../my_base.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max ...
exit
```

### 9\. Post-CTS Analysis (Modified Buffer List)

Explored the impact of the clock buffer list by removing `sky130_fd_sc_hd__clkbuf_1`, re-running CTS, and re-running STA.

### Key Learnings (Day 4)

  * [Your key learning point 1...]
  * [Your key learning point 2...]
  * [Your key learning point 3...]

### Command Explanations (Day 4)

  * `grid ...`: (Magic tkcon) Sets the layout grid to match the track pitches.
  * `lef write`: (Magic tkcon) Generates a LEF file from the layout.
  * `set ::env(EXTRA_LEFS) [glob ...]`: (Tcl) Tells OpenLANE to find all `.lef` files in the `src` directory.
  * `add_lefs -src $lefs`: (OpenLANE Tcl) Adds the found LEF files to the run.
  * `set ::env(SYNTH_STRATEGY) "DELAY 3"`: (Tcl) Sets the synthesis strategy to prioritize delay.
  * `init_floorplan`, `place_io`, `tap_decap_or`: (OpenLANE Tcl) The individual sub-steps of `run_floorplan`.
  * `sta pre_sta.conf`: (Bash) Starts OpenSTA and executes the script.
  * `replace_cell <old> <new>`: (STA Tcl) Swaps a cell instance with a new cell type.
  * `write_verilog <path>`: (STA Tcl) Saves the current (ECO-fixed) netlist.
  * `run_cts`: (OpenLANE Tcl) Runs Clock Tree Synthesis.
  * `openroad`: (Bash) Starts the OpenROAD tool.
  * `read_lef`, `read_def`, `read_verilog`, `read_liberty`: (OpenROAD Tcl) Commands to load the design database.
  * `link_design <top>`: (OpenROAD Tcl) Connects the netlist to the physical cell models.
  * `read_sdc <path>`: (OpenROAD Tcl) Loads the timing constraints.
  * `set_propagated_clock [all_clocks]`: (OpenROAD Tcl) Instructs the timer to use realistic, calculated clock latencies.

-----

## Day 5 Labs: PDN, Routing & Post-Route STA

This lab covers the final steps of the PnR flow: generating the power grid, running detailed routing, and performing a final timing analysis with extracted parasitic values.

### 1\. Perform Generation of Power Distribution Network (PDN)

Ran all steps from synthesis through CTS, then generated the PDN.

**Commands (in OpenLANE interactive shell):**

```tcl
# ... (prep, add_lefs, synthesis, floorplan, placement) ...
# unset ::env(LIB_CTS)
run_cts

# Now that CTS is done we can do power distribution network
gen_pdn
```

**Screenshots of PDN Run:**
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c4e5609e-3d85-49a1-a7cf-9819bb9be4eb" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b5dff2ff-ab0e-4a7e-b555-eb799cccebee" />

**Commands to Load PDN in Magic (new terminal):**

```bash
# Change directory to path containing generated PDN def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/29-10_19-51/tmp/floorplan/

# Command to load the PDN def in magic tool
magic -T .../sky130A.tech lef read ../../tmp/merged.lef def read 14-pdn.def &
```

**Screenshots of PDN Layout:**

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/42bfbe4f-add6-49d3-b4aa-1429eb06d2c9" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2988ad2e-9fb8-4f0a-b3d2-aa6e10efcd64" />

### 2\. Perform Detailed Routing (TritonRoute)

**Commands (in OpenLANE interactive shell):**

```tcl
# Check value of 'CURRENT_DEF'
echo $::env(CURRENT_DEF)

# Command for detailed route using TritonRoute
run_routing
```

**Screenshots of Routing Run:**

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7cfb9f86-426c-4bc6-8b2f-b331c24cb332" />

Screenshot of fast route guide present in openlane/designs/picorv32a/runs/29-10_19-51/tmp/routing directory

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bbc4bcbc-7aa1-4263-9f35-21b6a29e3a33" />

Screenshot of the spef file

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e80a6ecf-14a2-4c55-95ed-d7210016dd4c" />

**Commands to Load Routed DEF in Magic (new terminal):**

```bash
# Change directory to path containing routed def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/29-10_19-51/results/routing/

# Command to load the routed def in magic tool
magic -T .../sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &
```

**Screenshots of Routed Layout:**

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/839197ec-eb23-485f-b5f0-17713d7e0901" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/beed3387-647c-4a2d-8314-fd095bd66676" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3195ae58-b0b5-44d5-bd0e-2ba135ea1493" />

### 3\. Post-Route OpenSTA Timing Analysis

Performed a final timing analysis using the extracted SPEF file to get the most accurate results.

**Commands (in OpenLANE interactive shell):**

```tcl
# Command to run OpenROAD tool
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/29-10_19-51/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/29-10_19-51/results/routing/picorv32a.def

# Creating an OpenROAD database to work with
write_db pico_route.db

# Loading the created database in OpenROAD
read_db pico_route.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/29-10_19-51/results/synthesis/picorv32a.synthesis_preroute.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design and library
link_design picorv32a

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Read SPEF
read_spef /openLANE_flow/designs/picorv32a/runs/29-10_19-51/results/routing/picorv32a.spef

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Exit to OpenLANE flow
exit
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1d31600f-faf9-4410-bf0f-f44b79258507" />

**Screenshots of Post-Route Analysis:**

No DRC violations:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/76f39d92-0d51-46e5-a09e-c8be5f98e49e" />

STA for signoff:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5d2eeec6-667e-4cd2-9573-062022d0cd8f" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a70870d4-4978-4e6d-9975-fef99da0ad44" />

### Key Learnings (Day 5)

  * [Your key learning point 1...]
  * [Your key learning point 2...]
  * [Your key learning point 3...]

### Command Explanations (Day 5)

  * `gen_pdn`: (OpenLANE Tcl) Generates the Power Distribution Network (power and ground straps/rings) based on the settings in `config.tcl`.
  * `run_routing`: (OpenLANE Tcl) Runs the detailed router (TritonRoute) to connect all the standard cells and I/O pins.
  * `magic ... def read picorv32a.def &`: (Bash) Loads the *final* routed DEF file, which contains all placed cells, the PDN, and all signal routes.
  * `python3 main.py ...`: (Bash) Runs the external Python-based SPEF extractor to generate parasitic data from the final LEF and DEF files.
  * `read_spef <path>`: (OpenROAD Tcl) Reads a SPEF (Standard Parasitic Exchange Format) file. This annotates the design with realistic parasitic resistance (R) and capacitance (C) values from the physical routing, enabling a highly accurate final timing analysis.
  * `report_checks ...`: (OpenROAD Tcl) Generates the final timing report, which is now "post-route" and "parasitic-aware," representing the most accurate timing signoff.

-----

## Notes on Digital and Analog Block Interaction

-----

## Observations on DRC, LVS, and STA Inter-dependencies

```
```
