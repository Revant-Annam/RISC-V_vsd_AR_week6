# Cell Design and Characterization Flows

## Inputs for Cell Design Flow

In Cell Design Flow, components like gates, flip-flops, and buffers are termed 'Standard Cells'. These standard cells reside in a collection called the 'Library', which contains multiple cells with different 
functionality and also multiple cells with identical functionality but varying sizes. The standard cells also vary with the threshold voltage. 

Examining an inverter from the library, the cell design flow proceeds as follows:

The inverter must be represented through its shape, drive strength, power characteristics, and more. Cell design flow comprises three components:

Inputs

Design steps

Outputs

**1) Inputs**: Cell design requires PDKs, DRC and LVS rules, SPICE models, library specifications, and user-defined specifications. DRC & LVS rules provide tech files containing design rules and actual values 
that can be coded. SPICE models describe threshold voltage equations. User defined specifications like the cell height and width, supply voltage, metal layers. Power rail and ground rail separation determines cell height.
Cell width depends on timing requirements and drive strength.

**2) Design Steps**: Design encompasses three phases: circuit design, layout design, and characterization.

Circuit Design involves two phases: First, implementing the function itself; second, modeling PMOS and NMOS transistors to meet library requirements. The main condition is that the sum of the drain currents is 0.
With that we can model various parameters of the cell with the required switching theshold. These are done using SPICE simulations.

**3) Outputs**

Typical circuit design outputs include CDL (circuit description language) file, GDSII, LEF, and extracted spice netlist (.cir).

### Layout Design Step

In Layout Design, the first phase implements the function using MOS transistors through PMOS and NMOS transistor sets. The second phase extracts PMOS and NMOS network graphs from the implemented design.

After obtaining network graphs, the next phase determines Euler's path—a path traced only once.

Following this, create a stick diagram based on Euler's path, derived from the circuit diagram.

Next, convert the stick diagram into a proper layout following previously discussed rules. Once the layout is complete, specifications like cell width, cell length, drain current, and pin locations are established.

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

Next, compile inputs 1 through 8 into a configuration file for the characterization software "GUNA".

This software generates power, noise, and timing models.

### Timing Threshold Definitions

With back-to-back connected inverters, power sources, and applied stimulus, understanding different waveform threshold points becomes crucial—termed Timing threshold definitions.

The term 'Slew_low_rise_thr' represents values near 0, typically around 20% or possibly 30%.

Slew_high_rise_thr

Slew_low_fall_thr

Slew_high_fall_thr

Examining the input stimulus waveform (first buffer input) alongside the first buffer output, delay thresholds are also available, similar to slew. Extract rise and fall points from waveforms, 
with thresholds typically around 50%.

in_rise_thr

in_fall_thr, typically valued at 50%

out_rise_thr

out_fall_thr

### Propagation Delay and Transition Time

These values enable calculation of further parameters like propagation delay, current, and slews.

Delay calculation requires subtracting in_rise_thr from out_rise_thr. Using a typical 50% value, examining the waveform: 

<p align = "center"> Time delay = Time(out_thr) - time(in_thr). </p>

In the example with in_rise_thr and out_fall_thr at 50%, if the threshold point shifts upward, output precedes input, resulting in negative delay—which is unacceptable. Negative delay stems from poor threshold
point selection, emphasizing the importance of proper threshold point choice.

Another example shows correctly chosen threshold points still yielding negative delay because output precedes input—also unacceptable.

Transition time = time(slew_high_rise_thr) - time(slew_low_rise_thr)

or

Transition time = time(slew_high_fall_thr) - time(slew_low_fall_thr)

Examining a waveform demonstrates slew calculation.

### IO placer revision

We have already completed the initial floorplan and placement steps. However, the floorplan is not final and can be modified. For example, our first floorplan used the default setting, 
which placed I/O pins at equal distances around the core. If we want to change this behavior, we can.

To do this, we first check the configuration variables and find the relevant switch, which is `env(FP_IO_MODE) 1`. We can then override this in our configuration or interactively using the `set` command:

```tcl
set env(FP_IO_MODE) 2
```

After setting the new mode, we **re-run the floorplanning** (`run_floorplan`).

We can then check the new layout by loading the updated `.def` file into Magic. By changing the `FP_IO_MODE`, we can see a clear difference in the pin locations. For example, with the
new setting, the pins are no longer evenly distributed; instead, they are all clustered on the left bottom of the of the die.
