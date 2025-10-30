# Day 4: Pre-layout Timing Analysis and Clock Tree Synthesis

## 1. Fix DRC Errors and Verify Custom Cell Design

**Conditions for Standard Cell Verification:**

**Condition 1:** Input and output ports must align at vertical and horizontal track intersections.

**Condition 2:** Standard cell width must be odd multiples of horizontal track pitch.

**Condition 3:** Standard cell height must be even multiples of vertical track pitch.

**Commands to Open Custom Inverter Layout:**
```bash
# Navigate to vsdstdcelldesign directory
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign

# Open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```

**Screenshot:** tracks.info of sky130_fd_sc_hd

**Commands for Tkcon Window to Configure Grid:**
```tcl
# View grid command syntax
help grid

# Configure grid values
grid 0.46um 0.34um 0.23um 0.17um
```

**Screenshot:** Commands execution

**Condition 1 Verification:** Ports align at track intersections ✓

**Condition 2 Verification:**
- Horizontal track pitch = 0.46 μm
- Standard cell width = 1.38 μm = 0.46 × 3 ✓

**Condition 3 Verification:**
- Vertical track pitch = 0.34 μm
- Standard cell height = 2.72 μm = 0.34 × 8 ✓

## 2. Save Finalized Layout with Custom Name

**Command for Tkcon Window:**
```tcl
# Save layout with custom name
save sky130_vsdinv.mag
```

**Command to Open Saved Layout:**
```bash
# Open custom inverter layout
magic -T sky130A.tech sky130_vsdinv.mag &
```

**Screenshot:** Newly saved layout

## 3. Generate LEF from Layout

**Command for Tkcon Window:**
```tcl
# Generate lef file
lef write
```

**Screenshots:** Command execution and newly created lef file

## 4. Copy LEF and Library Files to 'picorv32a' Design

**Commands to Copy Required Files:**
```bash
# Copy lef file
cp sky130_vsdinv.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# Verify lef file copy
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# Copy library files
cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# Verify library files copy
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```

**Screenshot:** Commands execution

## 5. Edit 'config.tcl' to Include Custom Cell

**Commands to Add to config.tcl:**
```tcl
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

**Screenshot:** Updated config.tcl file

## 6. Run Synthesis with Custom Inverter Cell

**Commands to Invoke OpenLANE Flow:**
```bash
# Navigate to openlane directory
cd Desktop/work/tools/openlane_working_dir/openlane

# Start docker container
docker

# Launch interactive OpenLANE flow
./flow.tcl -interactive

# Load required packages
package require openlane 0.9

# Prepare design
prep -design picorv32a

# Include custom lef files
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Execute synthesis
run_synthesis
```

**Screenshots:** Command execution sequence

## 7. Optimize Design Parameters to Fix Violations

**Current Design Values Before Optimization:** [Note timing values]

**Commands to Improve Timing:**
```tcl
# Prepare design with overwrite
prep -design picorv32a -tag 24-03_10-03 -overwrite

# Include custom lef files
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Check and modify synthesis strategy
echo $::env(SYNTH_STRATEGY)
set ::env(SYNTH_STRATEGY) "DELAY 3"

# Check buffering status
echo $::env(SYNTH_BUFFERING)

# Enable synthesis sizing
echo $::env(SYNTH_SIZING)
set ::env(SYNTH_SIZING) 1

# Verify driving cell
echo $::env(SYNTH_DRIVING_CELL)

# Execute synthesis
run_synthesis
```

**Screenshot:** merged.lef showing custom inverter macro

**Screenshots:** Command execution sequence

**Result:** Area increased, worst negative slack reduced to 0

## 8. Run Floorplan and Placement

**Floorplan Command:**
```tcl
# Execute floorplan
run_floorplan
```

**Screenshots:** Command execution

**Alternative Floorplan Commands (if errors occur):**
```tcl
# Execute floorplan steps individually
init_floorplan
place_io
tap_decap_or
```

**Screenshots:** Alternative commands execution

**Placement Command:**
```tcl
# Execute placement
run_placement
```

**Screenshots:** Placement execution

**Commands to View Placement in Magic:**
```bash
# Navigate to placement results directory
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/24-03_10-03/results/placement/

# Load placement def in magic
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

**Screenshots:** Placement def in magic, custom inverter with proper abutment

**Tkcon Command to View Internal Layers:**
```tcl
# View internal connectivity
expand
```

**Screenshot:** Power pin abutment with library cells

## 9. Post-Synthesis Timing Analysis with OpenSTA

**OpenLANE Flow Commands:**
```bash
# Navigate to openlane directory
cd Desktop/work/tools/openlane_working_dir/openlane

# Start docker
docker

# Launch interactive flow
./flow.tcl -interactive

# Load packages
package require openlane 0.9

# Prepare design
prep -design picorv32a

# Include custom lef
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Enable sizing
set ::env(SYNTH_SIZING) 1

# Run synthesis
run_synthesis
```

**Screenshot:** Commands execution

**Create pre_sta.conf File:** In openlane directory

**Create my_base.sdc File:** In openlane/designs/picorv32a/src directory (based on openlane/scripts/base.sdc)

**STA Analysis Commands:**
```bash
# Navigate to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Invoke OpenSTA
sta pre_sta.conf
```

**Screenshots:** STA execution and results

**Optimization with Reduced Fanout:**
```tcl
# Prepare design with overwrite
prep -design picorv32a -tag 25-03_18-52 -overwrite

# Include custom lef
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Enable sizing
set ::env(SYNTH_SIZING) 1

# Reduce maximum fanout
set ::env(SYNTH_MAX_FANOUT) 4

# Verify driving cell
echo $::env(SYNTH_DRIVING_CELL)

# Run synthesis
run_synthesis
```

**Screenshots:** Commands execution and improved results

## 10. Timing ECO Fixes

**ECO Fix 1: Replace OR2 Gate Driving 4 Fanouts**
```tcl
# Report net connections
report_net -connections _11672_

# Check syntax
help replace_cell

# Replace with higher drive strength
replace_cell _14510_ sky130_fd_sc_hd__or3_4

# Generate timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

**Result:** Slack reduced

**ECO Fix 2: Replace Another OR2 Gate**
```tcl
# Report connections
report_net -connections _11675_

# Replace cell
replace_cell _14514_ sky130_fd_sc_hd__or3_4

# Generate report
report_checks -fields {net cap slew input_pins} -digits 4
```

**Result:** Slack reduced

**ECO Fix 3: Replace OR2 Driving OA Gate**
```tcl
# Report connections
report_net -connections _11643_

# Replace with OR4_4
replace_cell _14481_ sky130_fd_sc_hd__or4_4

# Generate report
report_checks -fields {net cap slew input_pins} -digits 4
```

**Result:** Slack reduced

**ECO Fix 4: Replace Another OR2 Driving OA Gate**
```tcl
# Report connections
report_net -connections _11668_

# Replace cell
replace_cell _14506_ sky130_fd_sc_hd__or4_4

# Generate report
report_checks -fields {net cap slew input_pins} -digits 4
```

**Result:** Slack reduced

**Verification Command:**
```tcl
# Verify replacement
report_checks -from _29043_ -to _30440_ -through _14506_
```

**Screenshot:** Replaced instance verification

**ECO Results:** WNS improved from -23.9000 to -22.6173 (reduction of ~1.2827 ns)

## 11. Update Netlist and Continue PnR Flow

**Backup Original Netlist:**
```bash
# Navigate to synthesis results
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/

# List directory contents
ls

# Create backup
cp picorv32a.synthesis.v picorv32a.synthesis_old.v

# Verify backup
ls
```

**Screenshot:** Backup commands execution

**Overwrite Netlist:**
```tcl
# Check syntax
help write_verilog

# Overwrite netlist
write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/picorv32a.synthesis.v

# Exit OpenSTA
exit
```

**Screenshot:** Write verilog execution

**Verification:** Instance _14506_ replaced with sky130_fd_sc_hd__or4_4

**Continue with Clean Design:**
```tcl
# Prepare design
prep -design picorv32a -tag 24-03_10-03 -overwrite

# Include custom lef
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# Set synthesis strategy
set ::env(SYNTH_STRATEGY) "DELAY 3"

# Enable sizing
set ::env(SYNTH_SIZING) 1

# Run synthesis
run_synthesis

# Execute floorplan steps
init_floorplan
place_io
tap_decap_or

# Run placement
run_placement

# Fix CTS library error if needed
unset ::env(LIB_CTS)

# Run clock tree synthesis
run_cts
```

**Screenshots:** Complete flow execution

## 12. Post-CTS Timing Analysis with OpenROAD

**OpenROAD Timing Analysis Commands:**
```tcl
# Launch OpenROAD
openroad

# Read lef file
read_lef /openLANE_flow/designs/picorv32a/runs/24-03_10-03/tmp/merged.lef

# Read def file
read_def /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.cts.def

# Create database
write_db pico_cts.db

# Load database
read_db pico_cts.db

# Read post-CTS netlist
read_verilog /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.synthesis_cts.v

# Read library
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design
link_design picorv32a

# Read SDC
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Set propagated clocks
set_propagated_clock [all_clocks]

# Check command syntax
help report_checks

# Generate timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Exit to OpenLANE
exit
```

**Screenshots:** OpenROAD analysis execution and timing reports

## 13. Analyze Impact of Removing 'sky130_fd_sc_hd__clkbuf_1'

**Commands to Modify CTS_CLK_BUFFER_LIST:**
```tcl
# Check current buffer list
echo $::env(CTS_CLK_BUFFER_LIST)

# Remove clkbuf_1 from list
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]

# Verify removal
echo $::env(CTS_CLK_BUFFER_LIST)

# Check current def
echo $::env(CURRENT_DEF)

# Set placement def
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/placement/picorv32a.placement.def

# Re-run CTS
run_cts

# Verify buffer list
echo $::env(CTS_CLK_BUFFER_LIST)

# Launch OpenROAD
openroad

# Read files and create database
read_lef /openLANE_flow/designs/picorv32a/runs/24-03_10-03/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.cts.def
write_db pico_cts1.db
read_db pico_cts.db

# Read netlist and library
read_verilog /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design
link_design picorv32a

# Read SDC
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Set propagated clocks
set_propagated_clock [all_clocks]

# Generate timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Report skew
report_clock_skew -hold
report_clock_skew -setup

# Exit OpenROAD
exit

# Restore buffer list
echo $::env(CTS_CLK_BUFFER_LIST)
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]
echo $::env(CTS_CLK_BUFFER_LIST)
```

**Screenshots:** Complete analysis with modified buffer list and timing reports
