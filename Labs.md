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

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d2e5fa4e-f795-4806-9d00-943b8067b069" />

These will create a `runs` folder in the `picorv32a` folder which contains the reports, results, logs, etc specific to the time when the `prep` command was ran.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7c42104e-af08-471b-aab0-4d5a2a6a9ee3" />

This is the netlist which has been created after the synthesis which is located in the `../picorv32a/runs/29-10_19-51/results/synthesis` folder:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e6a918cd-4d39-44db-a286-09767b641a3b" />

The STA analysis of the synthesized netlist can be found in the same folder:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/980bdbc1-269b-4da3-8deb-c1fe0fbc5796" />

### 3. Calculate the Flop Ratio

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7d9fc647-c4b0-4332-b162-336d0b5cdfc0" />

**Calculation:**
* **DFFs:** 1613
* **Total Cells:** 14876
* **Ratio:** $1613 / 14876 \approx 0.1084$
* **Percentage:** $10.84\%$

### Key Learnings (Day 1)

* **Understanding the OpenLANE Interactive Flow:** Learned the basic Tcl commands to run a design through OpenLANE interactively: `package require openlane`, `prep -design <design_name>`, and `run_synthesis`.
* **Navigating the File Structure:** Became familiar with the OpenLANE directory structure, specifically how designs are organized in the `designs/` folder and how all outputs (logs, results, reports) are generated inside a timestamped `runs/` directory.
* **Configuration Files:** Understood the config hierarchy, where a design's `config.tcl` sources a PDK-specific config file (e.g., `sky130A..._config.tcl`) to set up the flow.
* **Locating Synthesis Outputs:** Learned to find the most critical synthesis results:
    * **Gate-Level Netlist:** The synthesized design logic (e.g., `.../results/synthesis/picorv32a.v`).
    * **Statistics Report:** The post-synthesis cell report (e.g., `.../reports/synthesis/1-synthesis.stat.txt`).
    * **STA Report:** The static timing analysis results (`.sta.rpt`).
* **Basic Design Analysis (Flop Ratio):** Learned to parse the statistics report to find the total cell count and the specific count of flip-flops (like `sky130_fd_sc_hd__dfxtp_1`).
* **Flop Ratio as a Metric:** Calculating the flop ratio gives a quick, high-level insight into the design's composition, showing the ratio of sequential logic (flip-flops) to the total logic. **10.84%** indicates that roughly 1 in 10 cells is a flip-flop, which is typical for a processor core that balances registers and control/datapath logic.

### Command Explanations (Day 1)
  * `docker`: Starts the OpenLANE Docker container.
  * `./flow.tcl -interactive`: Starts the OpenLANE flow in interactive mode.
  * `package require openlane 0.9`: Loads the necessary OpenLANE Tcl package.
  * `prep -design picorv32a`: Prepares the design directory and links configuration/source files.
  * `run_synthesis`: Runs the synthesis flow, converting RTL to a gate-level netlist.

-----

## Day 2 Labs: Floorplan & Placement

This lab covers the floorplanning and placement stages for the 'picorv32a' design.

### 1\. Variables in Floorplan
In the `Desktop/work/tools/openlane_working_dir/openlane/configuration` we can find a README.md file. In this file the description of the variables which can be changed to change how the floorplan runs. These variables can be changed based on the design requirements.

<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/1b4caeca-2a79-41d3-b8df-df4a93e5e39a" />

### 2\. Run 'picorv32a' Design Floorplan

**Commands (in OpenLANE interactive shell):**

```tcl
# Continuing after the run_synthesis
run_floorplan
```

<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/8cc88d90-7002-4934-9802-5612ba60a079" />

<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/de389909-9f02-47b8-b1ca-e74538f3c6fe" />

We can observe in the `sky130A_sky130_fd_sc_hd_config.tcl` present in the `picorv32a` folder that some of the variables related to the floorplan have been changed which will affect the values in the default openlane settings. To ensure that the changes made in the `sky130A_sky130_fd_sc_hd_config.tcl` file are affected during the run, we can check the `config.tcl` file present in the `runs` folder.

`sky130A_sky130_fd_sc_hd_config.tcl` file:
<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/b96490e4-1863-421b-8992-0a185befdcfd" />

`config.tcl` file:
<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/9c63923d-2449-4638-9e8b-d95c2cee7778" />

### 3\. Calculate Die Area

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/66d5b8d2-013e-482d-b93f-9b22fadbe49b" />

From the `.def` file `DIEAREA ( 0 0 ) ( 660685 671405 )`:
$$ \text{Die Width} = \frac{660685}{1000} = 660.685 \text{ µm} $$
$$ \text{Die Height} = \frac{671405}{1000} = 671.405 \text{ µm} $$
$$ \text{Area} = 660.685 \times 671.405 \approx 443587.21 \text{ µm}^2 $$

### 4\. Explore Floorplan in Magic

**Commands (in new terminals):**

```bash
# To view Floorplan
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/29-10_19-51/results/floorplan/
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```

Basic commands in Magic:
- `S` -> Select a particular cell/wire or entire design.
- `V` -> To fit the selected region to the screen
- `Z` -> To zoom in
- `Shift + z` -> zoom out
- `what` -> After selecting a particular cell/wire by entering the what command in the command window of the magic we get the name of the cell/wire.

Floorplan:

<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/4ab07b45-9eea-4e98-84d2-d3b3173eff4c" />

<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/c3fc5d3a-4902-4002-96be-aebb6d9365e2" />

<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/b8a7b18e-e77d-4c88-8f0a-bd538be2a003" />

<img width="1920" height="983" alt="image" src="https://github.com/user-attachments/assets/7f8ba152-9858-4ae4-a4fd-cf905164ac84" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/78004918-dfab-402f-a16a-f4c6c5447c1b" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a17666d5-4c3b-4205-933f-1fac88cc58bf" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9b78e855-f2d2-4b93-9369-9511792d7a54" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/50d32047-c28d-4906-98d6-beb223389c9e" />

### 5\. Run 'picorv32a' Design Placement

**Commands (in OpenLANE interactive shell):**

```tcl
# Continuing after the run_floorplan
run_placement
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/aa49d175-cce8-4954-b1a5-af7c62909989" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7ca35939-1f6a-485c-b604-5c5e92ade25d" />

### 6\. Explore Placement in Magic

**Commands (in new terminals):**

```bash
# To view Placement
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/29-10_19-51/results/placement/

# Command to load the placement def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

Placement:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6e502a17-b6bd-4052-b10b-982cd8d47cfa" />

Standard cells placed legally:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/be515545-5e76-4c84-ad5f-e2401af1c1f6" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7e20c4bb-1c06-405a-a6bb-a6b3edb62d8f" />

### Key Learnings (Day 2)

* **Understanding the Floorplan:** The `run_floorplan` command is the first step in physical design. It defines the total **Die Area**, creates the **Core Area** (where standard cells will go), and places the **I/O pins** on the periphery.
* **Visual Inspection with Magic:** Learned how to use the `magic` layout viewer to load the technology file (`.tech`), the library abstracts (**LEF**), and the design layout (**DEF**). This allows to visually verify each stage of the PnR flow.
* **Reading a DEF File:** Learned to find key information in the `.def` file, specifically the `DIEAREA` coordinates.
* **DBU-to-Micron Conversion:** Learned the standard conversion factor (1000 DBU per micron for SKY130) to calculate the physical die dimensions (e.g., $660.685 \text{ µm} \times 671.405 \text{ µm}$).
* **Understanding Placement:** The `run_placement` command populates the empty core area. It takes all the standard cells (gates, flops) from the synthesis netlist and places them into legal, non-overlapping "standard cell rows."
* **Configuration Overrides:** understood the configuration hierarchy. Variables set in the design-specific `sky130A..._config.tcl` (like `FP_IO_MODE`) take precedence over default settings and are recorded in the final `runs/.../config.tcl` for that specific execution.

### Command Explanations (Day 2)

  * `run_floorplan`: Defines the chip's core dimensions, places I/O pins, and inserts well taps/decap cells.
  * `magic -T ... lef read ... def read ... &`: Opens the Magic layout tool, reading the LEF (abstract library) and DEF (design placement) files.
  * `run_placement`: Legally places all standard cells within the core boundaries.

This is changing the I/O mode from 1 to 2 will change the initial I/O configuration of equi-distant.

Before chanigng:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/11b5e0ca-e415-449d-b8d7-ae25450dcafd" />

After changing: 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/925e187f-0fec-48a0-ac4c-e034743ea583" />

-----

## Day 3 Labs: Custom Inverter Design & DRC Fix

This lab involves analyzing a custom inverter standard cell, performing post-layout simulation, and correcting DRC rules in the Magic tech file.

### 1\. Clone & View Custom Inverter

**Commands:**

```bash
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the git repo for the inverter
git clone [https://github.com/nickson-jose/vsdstdcelldesign]https://github.com/nickson-jose/vsdstdcelldesign)

# Go to the vsdstdcelldesign directory
cd vsdstdcelldesign

# Copy the Magic tech file to open the inverter in Magic
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# To open the inverter in the Magic
magic -T sky130A.tech sky130_inv.mag &
```
Cloning of the git repository and copying the tech file to the path:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/19b00884-be30-40b7-b63e-ea9c1fbaf022" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/64956078-7b9d-42e9-b7b1-122eeba4f1ab" />

Opening the inverter in the Magic:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/041ae052-a85d-4783-ad62-a761d192cd24" />

Understanding different layers in the Inverter:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3aab8e67-f6bc-4409-becf-5bcc1a31c40f" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2c3b4a59-499a-4b69-9eca-1de02fbf53ee" />

By clicking `S` 3 times hovering on a particular node the entire connectivity of the node is shown. Here I have seen the connectivity for `VPWR`, `VGND` and `Y`:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/04acf4ae-31c6-4462-91ea-8c7264bb274e" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3256578e-54d2-4a17-85a1-b65abe99b1c5" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ff53a3f8-9e4a-4f3c-927f-a4f68707f0ce" />

### 2\. SPICE Extraction of Inverter

**Commands (in tkcon window):**

```tcl
# To extract a spice from the layout
# COnfirms the location for the spice file to be generated.
pwd

# For converting into a .ext (extraction file)
extract all

# For converting into a spice netlist along with parasitic capacitance and resistance
ext2spice cthresh 0 rthresh 0
ext2spice
```

Converting into a extraction file:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0d4d865c-d7af-4cf1-a108-50608e69f615" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d1ab400f-3ea2-4905-986a-b7f6df7d3ffa" />

Converting into a spice netlist:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3176db28-fcd4-4342-89b5-6b248daaff63" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/37541f8c-1d32-4a9e-bc31-b27002518477" />

Spice netlsit generated:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/891d5514-b085-43d3-abe4-f150b31926a8" />

From the spice netlist we can understand the connecting of the cells which results in a inverter. 
We can enable the grid in the magic by pressing `g`.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8bbda704-8f90-4d9b-a997-40e63bc44813" />

After have to add the library from which the standard nmos and pmos are used. We have to change the nmos and pmos name as per the .lib file. We also have to provide supply voltages. The connections with the supply voltages is present but the supply is not present in the netlist. A is a pulse while VPWR, VGND is a constant voltage. We can also include the analysis which we need to perform. Here we are running a transient analysis. We also change any parasitic values if needed.

The updated netlist:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/28a4912b-7cad-4224-be4a-8b7791bd3b29" />

### 3\. Post-Extraction ngspice Simulations

**Commands:**

```bash
#Invoking ngspice and plotting the transient analysis
ngspice sky130_inv.spice
plot y vs time a
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f4fb88b5-ff87-4101-ba6a-bf619c4ed2e4" />

Waveform of the inverter is as expected:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2e3e38cb-d050-4e9c-8340-4b46e7c53795" />

Calculating the Rise time, fall time, Rise cell delay and the Fall cell delay using the commands as used in Week4.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/be72b634-25e8-4834-8067-1e44bc818ddd" />

**Timing Calculations:**

  * **Rise Transition Time:** $T_{rise} = \mathbf{137.774 \text{ ps}}$
  * **Fall Transition Time:** $T_{fall} = \mathbf{50.345 \text{ ps}}$
  * **Rise Cell Delay:** $D_{rise} = \mathbf{108.789 \text{ ps}}$
  * **Fall Cell Delay:** $D_{fall} = \mathbf{49.429 \text{ ps}}$

### 4\. Fix DRC Rules in Magic Tech File

**Setup Commands:**

To understand the basic rules and learning more about magic we can look into the website: [Magic](http://opencircuitdesign.com/magic/)

To learn more about the tecchnology file we can look into the website: [sky130 pdk](https://skywater-pdk.readthedocs.io/en/main/)

```bash
# Go to the home directory and then download the drc_test.tgz
cd
wget [http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz](http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz)

# Extracting the drc_tests folder
tar xfz drc_tests.tgz
cd drc_tests

# Command to view .magicrc file
gvim .magicrc

# Opening magic 
magic -d XR &
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e47e5eec-a133-4174-8be0-df66905d8fbf" />

.magicrc file:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8a0e06b5-d4df-44c9-b993-047098138d87" />

**DRC Rules:**

  * **`met3`:** Checking the DRC errors present in the `met3.mag`. The file is located in the current working directory and can be invoked into the MAGIC using the GUI. The error here is specifically m3.2 which is a spacing error. The details of the rule can be seen in this website: [DRC rules m3](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#m3)

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/135a7540-3e24-4b4e-a102-7e6b5082445b" />

    Checking the DRC in the m3.4 which is the enclosement of the via within m3: 

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ac93e6f4-da1b-4921-8e08-d24dc9bb99a2" />

    Using the box command we can see the distance and then check with the rule to find the DRC error.
    
    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1c29e643-15c7-4d0d-94e7-19526c4930ec" />

  * **`poly.9` (Simple Rule):** Fixed incorrect poly spacing rule. More details regarding the poly rule can be seen in this website [DRC rules poly](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#poly)

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6b87166d-3cb6-4e7b-b013-6498a8d7ee41" />

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4e351c38-0ad5-47ee-82fb-d726b3f6a1ba" />

    The rule poly.9 shows no drc violation even though spacing < 0.48u. This must be changed in the sky130A.tech drc rules section.
    
    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4ee8665c-3352-43f1-9eab-11229de9a046" />
    
    Updating the sky130A.tech file:

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/86f6c8fa-0941-4807-b695-12647b12dbc9" />

    Loading the tech file and checking the error can be done with the commands:

        ```bash
        # Loading updated tech file
        tech load sky130A.tech

        # Must re-run drc check to see updated drc errors
        drc check

        # Selecting region displaying the new errors and getting the error messages 
        drc why
        ```
    
    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1702a79d-58e2-4f0f-9d47-763e51f6c585" />

    Correcting another poly rule which is not mentioned in the sky130A.tech

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5c4931be-8b07-4a46-89a2-95b42fe9af5e" />

    Updation is the file under `poly`:

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c8094853-e0f1-4cf2-bc16-f613a448d7a0" />

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/08874438-80bf-4a5e-b477-8a45d09ce497" />

  * **`nwell.4` (Complex Rule):** Incorrectly implemented nwell.4 rule no drc violation even though no tap present in nwell. Details regarding the nwell rule can be viewed in the website [DRC rules nwell](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#nwell). Updating the .tech file:

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3447b715-da4d-40e2-b680-e7e4925d36fc" />

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/41f09623-fbe5-4dbb-bbd7-dc93e0761b81" />

    Then using these commands we can check the error:
    ```bash
    # Loading updated tech file
    tech load sky130A.tech

    # Change drc style to drc full
    drc style drc(full)

    # Must re-run drc check to see updated drc errors
    drc check

    # Selecting region displaying the new errors and getting the error messages 
    drc why
    ```

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8f2aff73-99bd-4c94-8d99-2782e637cec4" />

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2db2928e-a038-4cc0-9339-a440085916e9" />

    Corrected other nwell's by adding a tap into them:

    <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/06ab37c0-710c-4b40-a760-4bd48aa4da47" />

### Key Learnings (Day 3)

* **Layout-to-Simulation Flow:** Successfully run a full post-layout verification flow: viewing the layout (`.mag`), extracting a netlist (with parasitics), creating a testbench, and simulating it in `ngspice`.
* **Parasitic-Aware Extraction:** Learned the importance of the `ext2spice cthresh 0 rthresh 0` command. This tells Magic to extract parasitic capacitances (and resistances), which is essential for getting accurate, real-world timing data, as opposed to an ideal, pre-layout simulation.
* **SPICE Testbench Creation:** Learned that a raw extracted `.spice` file is not runnable by itself. It must be edited to:
    1.  **`.include`** the foundry model files (for the NMOS/PMOS transistors).
    2.  Add **voltage sources** (a DC supply for `VPWR` and a `PULSE` source for the input `A`).
    3.  Add **analysis commands** (like `.tran`) to tell the simulator what to do.
* **Cell Characterization:** Using `ngspice` we can now perform basic cell characterization, to measuring key performance metrics like transition times ($T_{rise}$/$T_{fall}$) and propagation delays ($D_{rise}$/$D_{fall}$).
* **DRC Tech File Debugging:**  
    * Learned to use test layouts (`drc_tests`) to find bugs in the `sky130A.tech` file.
    * Learned the iterative debugging loop: `drc check` -> `drc why` -> edit `.tech` file -> `tech load` -> `drc check` again.
    * Learned how to fix both simple (`spacing`) and complex (`cthresh`) rules.
    * Learned that complex rules (like `nwell.4`, which checks for taps) require `drc style drc(full)` to be evaluated.

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

Inside the `/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd` directory we can find the tracks.info file which tells us the routing tracks used during the routing process. The metal layers used in the inverter are `li1` so the dimensions of the inverter must be in the multiples of `li1` dimensions mentioned in the `tracks.info`. We should also ensure that the ports must be at the intersection of the horizontal and the vertical tracks for correct routing. This is already present. 

Commands for tkcon window to set grid as tracks of `li1` layer:

```bash
# Get syntax for grid command
help grid

# Set grid values accordingly
grid 0.46um 0.34um 0.23um 0.17um
```

Grid before running the commands:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d394729f-8394-408e-8fd2-5d48ff6cc19a" />

Grid after running the commands:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d787d62b-c41f-4794-b631-c53cad54d36c" />

The width pitch and the height pitch must be in the multiples of the grid which is verified by checking the number of tracks covering the design.

  * $$ \text{Width} = 1.38 \text{ µm} = 0.46 \text{ µm} \times 3 $$ (Verified)
  * $$ \text{Height} = 2.72 \text{ µm} = 0.34 \text{ µm} \times 8 $$ (Verified)

### 2\. Generate LEF from Layout

**Commands (in tkcon window):**

```tcl
# Command to save as
save sky130_vsdinv.mag

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_vsdinv.mag &
```

Saving the .mag file:
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0a30447a-293a-499e-99e2-c3ad544d7916" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a9e92c4a-2a4b-40de-824a-a20da9b53cb3" />

The `.lef` file must be generated. After opening the saved `.mag` file in the magic we can enter the command `lef write` this generates the `.lef` file. 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/482bd453-c3d8-451f-a422-09693659ff93" />

### 3\. Copy LEF and LIB files to 'picorv32a' Source

**Commands (in bash terminal):**

```bash
# Copy lef file
cp sky130_vsdinv.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# List and check whether it's copied
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# Copy lib files
cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# List and check whether it's copied
cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
ls -ltr
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1977a545-322b-4392-9771-1345db13ec89" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a1dcad27-763f-4658-9268-d23c7890b910" />

We can see in the standard library that the `sky130_vsdinv` cell characterization is present in the `sky130_fd_sc_hd__typical.lib` file. 

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ebb8d84e-0140-4568-b7e3-00e86bdce324" />

### 4\. Edit 'config.tcl' to Include Custom Cell

Added `changes in the `config.tcl`:
```tcl
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/sky130_vsdinv.lef]
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6b52ff8d-3a1a-41d9-bd87-1657d8dcbcec" />

### 5\. Run Synthesis & Optimize

**Commands (in OpenLANE interactive shell):**

```tcl
# Change directory to openlane flow directory
cd Desktop/work/tools/openlane_working_dir/openlane

# Since we have aliased the long command to 'docker' we can invoke the OpenLANE flow docker sub-system by just running this command
docker

# Now that we have entered the OpenLANE flow contained docker sub-system we can invoke the OpenLANE flow in the Interactive mode using the following command
./flow.tcl -interactive

# Now that OpenLANE flow is open we have to input the required packages for proper functionality of the OpenLANE flow
package require openlane 0.9

# Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a'
# We use the -overwrite tag to change the already existing folder in the runs this is done to avoid large number of folders being generated.
prep -design picorv32a -tag 29-10_19-51 -overwrite

# Adiitional commands to include newly added lef to openlane flow
set lefs [glob $::env(DESIGN_DIR)/src/sky130_vsdinv.lef]
add_lefs -src $lefs

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8624bebb-5fa3-4691-a1b7-a5d1839b2367" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cf4396f3-4d8b-44e7-8f52-a338fa2445e0" />

### 6\. Run Floorplan & Placement with Custom Cell

**Commands (in OpenLANE interactive shell):**

```tcl
init_floorplan
place_io
tap_decap_or
run_placement
```

**Result:** The custom inverter was successfully placed and abutted.

### 7\. Remove/reduce the newly introduced timing violations.

- Changing the `SYNTH_STRATEGY` using: `set ::env(SYNTH_STRATEGY) "DELAY 3"`

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/67737989-4740-4369-9a41-1e7c2e463582" />

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ebce52ff-cb5b-4b88-9e0a-b1e1bb7ad28f" />

- Changing the `SYNTH_SIZING` using: `set ::env(SYNTH_SIZING) 1`

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/eae53c80-68f8-43d2-9ffd-ab4b6ad262c2" />

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/06b007dc-390b-4121-b844-cdb0fe4dbe2d" />

  After running the `run_synthesis` again we can observe that the WNS becomes 0 but the area is increased. This is due to the change in the `SYNTH_STRATEGY` which focuses on decreasing the delay while increasing the area.

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/aa733290-4b5a-43ba-bad8-82535314aa00" />

  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f4567348-3695-4ddd-a131-ea150ed75893" />


### 8\. Run floorplan and placement and verify the cell is accepted in PnR flow.

Now that our custom inverter is properly accepted in synthesis we can now run floorplan using following command

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2f7b22be-ae10-43fd-b3a8-e31b6d91e2ed" />

```tcl
run_floorplan
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b7c025b2-38ed-4c78-a9dd-c451f24d35dd" />

Since we are facing an error while using run_floorplan command, we can use the following set of commands available based on information from Desktop/work/tools/openlane_working_dir/openlane/scripts/tcl_commands/floorplan.tcl and also based on Floorplan Commands section in Desktop/work/tools/openlane_working_dir/openlane/docs/source/OpenLANE_commands.md

```tcl
# Follwing commands are alltogather sourced in "run_floorplan" command
init_floorplan
place_io
tap_decap_or
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/973ecd5b-ccf3-4c03-ad28-03a51b54ca45" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c991f634-9495-43da-b53b-76cb272e05a8" />

After the floorplan we run the Placement with the command:

```tcl
run_placement
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/611ccc4b-96b4-4387-9f2d-98a963fef1ea" />

In the MAGIC we can observe that the cell sky130_vsdinv is present after the placement. These are the commands used to open the MAGIC tool:

```tcl
# Change directory to path containing generated placement def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/24-03_10-03/results/placement/

# Command to load the placement def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c2d17645-91ff-402b-9867-8ff026ba7114" />

With the internal layout we can view the inverter by selecting the inverter and then entering this command:

```tcl
# Command to view internal connectivity layers
expand
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e2a39c76-e53c-418b-af3a-dc77be8a2d94" />

### 9\. Post-Synthesis STA with OpenSTA tool.

Since we were having 0 wns after improved timing run we are going to do timing analysis on initial run of synthesis which has lots of violations and no parameters were added to improve timing. First we need to run the synthesis without changing the `SYNTH_STRATEGY`. Then we need to create pre_sta.conf for STA analysis in openlane directory.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/32d5ae5c-24a9-4f8d-9519-6801ec713448" />

Newly created base.sdc for STA analysis in `openlane/designs/picorv32a/src` directory based on the file `openlane/scripts/base.sdc`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/04b44e40-d22c-4114-a60c-38a81c679a72" />

**Commands (in bash terminal):**

```bash
sta pre_sta.conf
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c2334b0c-9e44-44ef-9a3b-2aad5ee0f690" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bda7d0bc-c670-4673-8f68-223e1d1fb4c3" />

OR gate of drive strength 2 is driving 4 fanouts. This might be the cause for the violation. We can replace the cell with a higher drive strength.

**Commands (in STA shell):**

```tcl
# Reports all the connections to a net
report_net -connections _11672_

# Checking command syntax
help replace_cell

# Replacing cell
replace_cell _14510_ sky130_fd_sc_hd__or3_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3ca257e4-daf9-4a65-a77b-c4ffe5abcde5" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cae07002-2ee5-4a0d-ab9f-a258e013c399" />

Improved the WNS from -23.90 to -23.50:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ea254bb8-f7f5-4689-a953-3e06002bc6a2" />

Still the slack is not acceptable, so we can try replacing the cells with a higher delay/ larger slew/ higher fanout cells. Here I am replacing an OR gate of drive strength 2 is driving 4 fanouts:

**Commands (in STA shell):**

```tcl
# Reports all the connections to a net
report_net -connections _11675_

# Replacing cell
replace_cell _14514_ sky130_fd_sc_hd__or3_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

We can observe the WNS improved from -23.50 to -23.14:
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4c4e04b7-e0f1-4ce8-b9de-3c1f0bca5d21" />

The OR gate of drive strength 2 driving OA gate has more delay which is replaced by a OR gate which has a higher fanout.

```bash
# Reports all the connections to a net
report_net -connections _11668_

# Replacing cell
replace_cell _14506_ sky130_fd_sc_hd__or4_4

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1b1f3026-1f41-41d0-8063-c8f8d07da97d" />

**Result:** WNS improved from -23.9000 ns to -22.6173 ns.

We can also check in the netlist that the cells have been replaced.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/721f494e-d883-406c-b547-6e6a3d7e5d26" />

### 10\. Implement the floorplan, placement and cts for the new netlist

Now to insert this updated netlist to PnR flow and we can use write_verilog and overwrite the synthesis netlist but before that we are going to make a copy of the old old netlist

Commands to make copy of netlist

```
# Change from home directory to synthesis results directory
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/

# List contents of the directory
ls

# Copy and rename the netlist
cp picorv32a.synthesis.v picorv32a.synthesis_old.v

# List contents of the directory
ls

# Check syntax
help write_verilog

# Overwriting current synthesis netlist
write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/picorv32a.synthesis.v

# Exit from OpenSTA since timing analysis is done
exit
```

### 12\. Ran the full PnR flow on the new design.

We have be loaded in PnR the earlier 0 violation design on which we are continuing with the clean design to further stages.

**Commands:**
```tcl
# Now once again we have to prep design so as to update variables
prep -design picorv32a -tag 29-10_19-51 -overwrite

# Addiitional commands to include newly added lef to openlane flow merged.lef
set lefs [glob $::env(DESIGN_DIR)/src/sky130_vsdinv.lef]
add_lefs -src $lefs

# Command to set new value for SYNTH_STRATEGY
set ::env(SYNTH_STRATEGY) "DELAY 3"

# Command to set new value for SYNTH_SIZING
set ::env(SYNTH_SIZING) 1

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis

# Follwing commands are alltogather sourced in "run_floorplan" command
init_floorplan
place_io
tap_decap_or

# Now we are ready to run placement
run_placement

# Incase getting error
unset ::env(LIB_CTS)

# With placement done we are now ready to run CTS
run_cts
```
Synthesis:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d38511c5-1893-40e1-b527-0d24039b6dd2" />

Floorplan:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/01a62b1e-bef2-47dd-92d7-370c93199aa4" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/155394de-3c37-4d8b-9093-00a3a64573c2" />

Placement:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/dd4c7dab-cb66-4cfb-b01c-4e658c404e85" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c4521b9e-ed24-40ea-8b31-790945341565" />

CTS:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5f5042c6-f143-4999-9889-b1fde8e0aa56" />

### 13\. Explore post-CTS OpenROAD timing analysis by removing 'sky130_fd_sc_hd__clkbuf_1' cell from clock buffer list variable 'CTS_CLK_BUFFER_LIST'.

**Commands (in OpenLANE interactive shell):**

```tcl
# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Removing 'sky130_fd_sc_hd__clkbuf_1' from the list
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Checking current value of 'CURRENT_DEF'
echo $::env(CURRENT_DEF)

# Setting def as placement def
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/placement/picorv32a.placement.def

# Run CTS again
run_cts

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Command to run OpenROAD tool
openroad

# Reading lef file
read_lef /openLANE_flow/designs/picorv32a/runs/24-03_10-03/tmp/merged.lef

# Reading def file
read_def /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.cts.def

# Creating an OpenROAD database to work with
write_db pico_cts1.db

# Loading the created database in OpenROAD
read_db pico_cts.db

# Read netlist post CTS
read_verilog /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.synthesis_cts.v

# Read library for design
read_liberty $::env(LIB_SYNTH_COMPLETE)

# Link design and library
link_design picorv32a

# Read in the custom sdc we created
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

# Setting all cloks as propagated clocks
set_propagated_clock [all_clocks]

# Generating custom timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

# Report hold skew
report_clock_skew -hold

# Report setup skew
report_clock_skew -setup

# Exit to OpenLANE flow
exit

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)

# Inserting 'sky130_fd_sc_hd__clkbuf_1' to first index of list
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]

# Checking current value of 'CTS_CLK_BUFFER_LIST'
echo $::env(CTS_CLK_BUFFER_LIST)
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b972e682-f629-40b9-aab2-c08c28bc0009" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a1bf312b-2669-4eba-a215-a5b029ba2833" />

### Key Learnings (Day 4)

* **Standard Cell PnR-Readiness:** Learned that for a custom cell to be used in PnR, it's not enough to just have a layout. The layout must:
    * Have dimensions (width and height) that are **integer multiples of the PDK's track pitches** (`tracks.info`).
    * Have its I/O pins placed **on-grid** (at the intersection of tracks).
    * Successfully verified this using the `grid` command in Magic.
* **Integrating Custom Cells:** Learned the full procedure to add a new cell to the OpenLANE flow:
    1.  **Generate LEF:** Create the layout abstract using `lef write`.
    2.  **Copy Files:** Place the new `.lef` and the characterization `.lib` files into the design's `src` folder.
    3.  **Update Config:** Modify the `config.tcl` to point `::env(LIB_SYNTH)` to the local `.lib` file and add the new LEF file path to `::env(EXTRA_LEFS)`.
* **Visual Verification in PnR:** Verified the integration worked by running `run_placement` and then using the `expand` command in Magic to see layout of the `sky130_vsdinv` cell, confirming it was placed as a full, un-abstracted cell.
* **Automated Timing Optimization:** Learned to use `::env(SYNTH_STRATEGY)` and `::env(SYNTH_SIZING)` to tell the synthesis tool to prioritize **timing (delay) over area**. This is a fundamental **timing vs. area tradeoff**.
* **Manual Timing ECO (Engineering Change Order):**
    * Learned to use the standalone **OpenSTA** (`sta`) tool for deep-dive timing analysis.
    * Performed a manual ECO by finding a failing path (`report_checks`), investigating its fanout (`report_net`), and manually swapping a weak cell with a stronger one (`replace_cell ... or3_4`).
    * Saved this new, manually-fixed netlist using `write_verilog`.
* **Running Clock Tree Synthesis (CTS):** Successfully ran the `run_cts` command, which inserts buffers to build the clock tree and balance latencies. Learned that this step is controllable, for example by editing the `::env(CTS_CLK_BUFFER_LIST)`.
* **Full Post-Layout STA (OpenROAD):** Learned how to run a final, high-accuracy STA check inside OpenROAD after CTS. This involves:
    1.  Loading the design database (`read_lef`, `read_def`, `read_verilog`).
    2.  Loading the timing libraries (`read_liberty`).
    3.  Linking the design (`link_design`).
    4.  Reading the constraints (`read_sdc`).
    5.  **Crucially,** running `set_propagated_clock` to tell the tool to use realistic, calculated clock delays instead of ideal ones.

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

* **Generating the Power Grid (PDN):** The `gen_pdn` command is responsible for creating the power and ground "scaffolding" of the chip. We can see in Magic how this lays down a grid of higher-level metal (like `met4`, `met5`) straps and rings that will deliver `VPWR` and `VGND` to the standard cell rows.
* **Detailed Routing:** The `run_routing` command uses the lower-level metal layers (like `li1`, `met1`, `met2`, `met3`) to create the physical wires that connect all the cell pins, following the logical netlist.
* **Parasitic Extraction:** After routing, the design is no longer just a logical netlist; it's a physical layout where every wire has a real length, width, and proximity to other wires. This creates parasitic **resistance (R)** and **capacitance (C)**. The SPEF file (`.spef`) is the standard format for storing these R and C values for every single net in your design.
* **The Final Signoff STA:** The post-route STA (Section 3) is the **most accurate timing analysis** possible.
    * Unlike pre-synthesis or post-synthesis STA, which *estimates* wire delays, this final analysis uses the **`read_spef`** command.
    * This command annotates every net in the design with its *actual, measured* parasitic R and C values from the real layout.
    * This provides a highly accurate, "signoff-quality" report. Your final screenshot, showing positive slack (WNS = 0.00), confirms that your design **meets timing** even after all physical and parasitic effects are accounted for.

### Command Explanations (Day 5)

  * `gen_pdn`: (OpenLANE Tcl) Generates the Power Distribution Network (power and ground straps/rings) based on the settings in `config.tcl`.
  * `run_routing`: (OpenLANE Tcl) Runs the detailed router (TritonRoute) to connect all the standard cells and I/O pins.
  * `magic ... def read picorv32a.def &`: (Bash) Loads the *final* routed DEF file, which contains all placed cells, the PDN, and all signal routes.
  * `read_spef <path>`: (OpenROAD Tcl) Reads a SPEF (Standard Parasitic Exchange Format) file. This annotates the design with realistic parasitic resistance (R) and capacitance (C) values from the physical routing, enabling a highly accurate final timing analysis.
  * `report_checks ...`: (OpenROAD Tcl) Generates the final timing report, which is now "post-route" and "parasitic-aware," representing the most accurate timing signoff.
