# Day 3: Design library cell using Magic Layout and ngspice characterization

## SPICE Deck Creation for CMOS Inverter

### VTC- SPICE Simulations

The initial phase involves creating a SPICE deck, which contains netlist connectivity information. It includes simulation inputs and deck points for capturing outputs.

**Component Connectivity**: Define substrate pin connectivity, which adjusts the threshold voltage for both PMOS and NMOS transistors. The output load capacitance depends on various factors, here we are assuming
a common value.

<img width="543" height="473" alt="image" src="https://github.com/user-attachments/assets/08294d82-5701-48eb-91f1-6214891deeca" />

**Component Values**: Specify width and length values for PMOS and NMOS, with both transistors having identical sizing. In general we take PMOS width greater than the NMOS width. In this phase we also assign Vin 
and Vdd voltages. 

<img width="540" height="520" alt="image" src="https://github.com/user-attachments/assets/32ceee98-517c-43b6-80ba-e9940b41ecc2" />

**Identify the Nodes**: Nodes represent points between components and are essential for netlist definition.

<img width="700" height="577" alt="image" src="https://github.com/user-attachments/assets/e200388e-ad8d-4dda-81e0-196cce21bae1" />

**Name the Nodes**: Assign names to these nodes: Vin, Vss, Vdd, out.

<img width="591" height="531" alt="image" src="https://github.com/user-attachments/assets/7201d831-8edc-4275-b4cc-1b0ff8e52b16" />

SPICE deck writing follows this format for NMOS or PMOS:

name Drain - Gate - Source - Substrate width length 

For M1 MOSFET: drain connects to out node, gate connects to in node, PMOS transistor substrate and source connect to Vdd node.

For M2 MOSFET: drain connects to out node, gate connects to in node, NMOS source and substrate connect to 0.

<img width="662" height="180" alt="image" src="https://github.com/user-attachments/assets/4be60cad-4460-4f58-800d-f1002539e3cd" />

## SPICE Simulation Lab for CMOS Inverter

After describing CMOS inverter connectivity, we define other component connections like load capacitor and source. The output load capacitor connectivity is as follows:

Connected between out and node 0 with a value of 10ff. Supply voltage (Vdd) connects between Vdd and node 0 with 2.5V value. Input voltage connects between Vin and node 0, also at 2.5V.

<img width="401" height="175" alt="image" src="https://github.com/user-attachments/assets/25a3689a-aadf-424c-81ea-535bacbf3ef4" />

Simulation commands sweep Vin from 0 to 2.5V with 0.05V steps to observe Vout variations with changing Vin.

<img width="502" height="143" alt="image" src="https://github.com/user-attachments/assets/eec52872-0a4d-4113-af8a-4ac9474771a6" />

The final step involves model files containing complete NMOS and PMOS descriptions.

<img width="652" height="107" alt="image" src="https://github.com/user-attachments/assets/36f46b3a-f4d3-4d23-9cd6-53bef630c4c7" />

<img width="1251" height="602" alt="image" src="https://github.com/user-attachments/assets/8971224b-319f-444a-9a2b-a2e121ac0ea5" />

Running SPICE simulation with these values generates the corresponding graph.

<img width="807" height="657" alt="image" src="https://github.com/user-attachments/assets/7622a4ab-6b38-458d-86b0-b0d53e455412" />

Another simulation changes PMOS width to 2.5 times the NMOS width, producing a different graph.

<img width="797" height="647" alt="image" src="https://github.com/user-attachments/assets/f58f2b8e-18f0-4f09-a942-cb7f12c28f49" />

The distinction between graphs: the second graph's transfer characteristic centers exactly in the middle, while the first graph's characteristic is shifted left from center.

## Switching Threshold Vm

Both different-width models serve specific applications. Comparing waveforms reveals identical shapes regardless of voltage levels, demonstrating CMOS robustness. When Vin is low, output is high; when Vin is high,
output is low. This characteristic persists across all CMOS configurations with varying NMOS or PMOS sizes, explaining CMOS logic's widespread use in gate design.

Switching threshold Vm (the device's level-switching point) defines inverter robustness. It's the point where Vin equals Vout.

<img width="1011" height="696" alt="image" src="https://github.com/user-attachments/assets/eeabf75a-9189-4e60-a5cb-00d6b570ea3d" />

At Vm~0.9V, Vin equals Vout. This represents a critical point where both PMOS and NMOS may turn on simultaneously, creating high leakage current risk (direct current flow from power to ground).

Comparing both graphs clarifies the switching threshold voltage concept.

<img width="1245" height="653" alt="image" src="https://github.com/user-attachments/assets/94499876-2420-4605-b9d1-a9ed1efd2bb9" />

The graph below indicates which region PMOS and NMOS occupy, showing different current flow directions for NMOS and PMOS. At the switching threshold both the MOS are in saturation region. With the main condition 
that the current in the NMOS is negative of the current in the PMOS

<img width="931" height="412" alt="image" src="https://github.com/user-attachments/assets/22a21e34-f42b-4324-b771-6c3a579bc7bd" />

## Static and Dynamic Simulation of CMOS Inverter

Dynamic simulation reveals CMOS inverter rise and fall delays and their variation with Vm. This simulation maintains all parameters same except the input, which becomes a pulse, with .tran as the simulation
command.

<img width="1252" height="547" alt="image" src="https://github.com/user-attachments/assets/64619bff-20ce-42a8-a222-4ff5e41393f2" />

<img width="800" height="645" alt="image" src="https://github.com/user-attachments/assets/0a97aa96-d47b-4ba8-b4a1-812e9538b551" />

A Time vs Voltage graph enables rise and fall delay calculation.

## Labs:

### Cloning the Standard Cell Repository

To get the standard cell design files, we first clone the Git repository. We copy the repository's clone address and run the `git clone` command in the OpenLANE terminal. 
This downloads and creates a new folder named `vsdstdcelldesign` inside the main OpenLANE directory.

If we look inside this `vsdstdcelldesign` folder, we'll find all the library files, including the `.mag` layout files for the standard cells.

### Viewing the Inverter Layout in Magic

Our next goal is to open the `.mag` file for a standard cell, like an inverter, to see which layers were used to build it.

Before we can open the layout file in Magic, we must provide the corresponding **technology file (`.tech`)**. To make this easier, we first copy the main tech file from its PDK location (using the `cp` command)
and paste it directly into our `vsdstdcelldesign` folder.

Now that the tech file is in our local directory, we can launch Magic without needing to write the full path to the tech file. The command becomes much simpler. Once Magic launches, it displays the complete
physical layout of the CMOS inverter, showing all the metal and diffusion layers.

## Inception of Layout – CMOS Fabrication Process

The steps involved in a fabrication of CMOS inverter:

**1) Selecting a Substrate**: Use p-type silicon substrate with high resistivity (5-50 ohm), well-doped, with (100) orientation. The doping of the substrate is generally less when comapred to the wells.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/097a7bd6-49cb-4ed9-9bb3-ca69fe5228e3" />

**2) Creating Active Region for Transistors**: Regions where PMOS and NMOS will be formed. On the p-type substrate, create small pockets called active regions where PMOS and NMOS transistors will be fabricated.
Create isolation between each pocket.

Form the isolation layer by depositing an $SiO_2$ layer (~40nm) on the substrate, followed by depositing an $Si_3$ $N_4$ layer (~80nm) on the SiO2 layer.

<img width="1083" height="493" alt="image" src="https://github.com/user-attachments/assets/4776e95a-cfd8-4819-8ea7-793702569c6c" />

Before creating pockets, identify the required regions. Deposit a photoresist layer (~1μm) and create "mask1" using UV light.

<img width="1105" height="562" alt="image" src="https://github.com/user-attachments/assets/a69a7485-6265-47f0-afcd-4143d97d95e1" />

Unwanted areas are exposed using UV light, and the exposed pattern gets washed away. This is called as photolithography. 

<img width="857" height="497" alt="image" src="https://github.com/user-attachments/assets/c394721a-0fc6-4946-a389-d0e995b7872d" />

Next, remove the mask and etch the $Si_3$ $N_4$ layer on exposed areas.

<img width="848" height="407" alt="image" src="https://github.com/user-attachments/assets/74d402c9-ac1a-43b8-a1dd-0da09907b598" />

Remove photoresist through chemical reaction, as the $Si_3$ $N_4$ layer now acts as protection for the $SiO_2$ layer. Place in an oxidation furnace. The LOCOS (local oxidation of silicon) process grows the 
exposed $SiO_2$ part, forming bird's beak structures. This grown $SiO_2$ provides perfect isolation between PMOS and NMOS, preventing transistor interaction.

<img width="1062" height="527" alt="image" src="https://github.com/user-attachments/assets/367234bd-74e4-4742-8618-148e968d32b8" />

Remove $Si_3$ $N_4$ using hot phosphoric acid.

<img width="896" height="360" alt="image" src="https://github.com/user-attachments/assets/dcfa3038-8269-4311-9a65-6b218895a226" />

**3) N-well and P-well Formation**: P-well and N-well cannot form simultaneously. Protect one region with photoresist while forming the other. Use "mask 2" and UV light for photoresist patterning to form P-well.

<img width="967" height="560" alt="image" src="https://github.com/user-attachments/assets/5adb0d7c-4bab-4c8d-91e7-c0fc674cd99c" />

The P-well formation area is now exposed. Remove the mask and apply **Ion implantation** (~200keV) using Boron to form P-well. This creates P implant initially. High-temperature annealing converts it to P-well.

<img width="1188" height="557" alt="image" src="https://github.com/user-attachments/assets/f689fffe-bc56-47dd-abd6-ddc121ae0636" />

Repeat similar steps for N-well formation using "mask 3" and Phosphorus ions.

<img width="1087" height="561" alt="image" src="https://github.com/user-attachments/assets/37885233-f3c9-4d01-88a6-6bd746ca30ba" />

Well depths aren't yet defined, so use high-temperature furnace (drive-in diffusion) to establish well depths. This process is called twin tub process.

<img width="908" height="378" alt="image" src="https://github.com/user-attachments/assets/b9e5651b-fba5-4462-928a-b486c592258b" />

**4) Gate Formation**: The gate terminal is crucial for PMOS and NMOS as it controls threshold voltage. Doping concentration and oxide capacitance regulate threshold voltage, so maintain proper doping 
concentration using "mask 4" and ion implantation of Boron at lower energy (~60keV).

<img width="957" height="565" alt="image" src="https://github.com/user-attachments/assets/24a16cc1-cdb6-4091-a699-d49496bd84ac" />

Repeat for N-well using "mask 5" and Arsenic/Phosphorous ions.

<img width="903" height="570" alt="image" src="https://github.com/user-attachments/assets/8bc37f03-3c54-4926-b7e4-baa84f27d3cd" />

Next, fix the oxide layer. First remove the damaged oxide layer using HF solution, then re-grow high-quality oxide with consistent thickness (~10nm).

The doping concentration was maintained by the ions implantation and the oxide thickness is maintained by the re growth of the oxide layer.

Final step: Deposit polysilicon layer over oxide with increased impurities (Arsenic) for low-resistance gate terminal. Etch this polysilicon using mask 6 and photoresist. Remove the mask and perform 
etching in the remaining region, remove photoresist to reveal the gate terminal.

<img width="910" height="552" alt="image" src="https://github.com/user-attachments/assets/158dc439-fbe8-4053-9d3b-9a3ee7949be2" />

<img width="890" height="388" alt="image" src="https://github.com/user-attachments/assets/60908541-3e0a-4827-993f-d216b991c1a1" />

**5) Lightly doped drain (LDD) Formation**: The goal is P+, P- (lightly doped profile), N doping profile in PMOS and N+, N-, P doping profile for NMOS. The reasons for lightly doped drain are:

- Hot electron effect: As the device size reduces, the main idea is not to change the power supply. Due to this the E-field (E = V/d) increases and the high energy thus break the Si-Si bonds (3.2eV). 
- Short channel effect: The drain field might penetrate the channel. 

For LDD formation, perform ion implantation in P-well using "mask 7" with phosphorus for light doping (P-).

<img width="882" height="556" alt="image" src="https://github.com/user-attachments/assets/36d67191-31e5-4abb-aba3-f905a87f1aa9" />

Repeat for N-well using "mask 8" and Boron ions.

<img width="898" height="540" alt="image" src="https://github.com/user-attachments/assets/8cdda4a1-356c-4a9c-9ed5-d407a1e3edd0" />

We create spacers to protect the actual P-implant and N-implant structure by depositing thick $SiO_2$ or $Si_3$ $N_4$ layer over the gate terminal.

<img width="882" height="426" alt="image" src="https://github.com/user-attachments/assets/2d5b2d81-ae94-4abe-81d4-571034f0b53d" />

Perform plasma anisotropic etching (directional etching) to form side-wall spacers around the gate terminal .

<img width="906" height="380" alt="image" src="https://github.com/user-attachments/assets/8a7bd00c-9a95-4703-a812-9f6515bb6965" />

**6) Source-Drain Formation**

Deposit very thin screen oxide layer to prevent channeling effects (if vector velocity of ions matches the crystalline structure of P-substrate the ions might get implanted deep into the substrate).

<img width="1150" height="498" alt="image" src="https://github.com/user-attachments/assets/084039d0-1844-409d-af83-004ed4d2bff8" />

For drain and source formation, perform ion implantation of Arsenic at 75keV to create N+ implant using "mask 9" in P-well for PMOS. The side wall spaces is to ensure that the LDD is intact.

<img width="1012" height="555" alt="image" src="https://github.com/user-attachments/assets/c7738715-1f46-491e-9d19-2b7ba6db6ed7" />

Repeat for NMOS using "mask 10" and boron ions in N-well at 50keV to create P+ implant.

<img width="1061" height="546" alt="image" src="https://github.com/user-attachments/assets/0d45a419-0cff-4d56-9f3a-a714b22f1fff" />

Place the partially completed CMOS in high-temperature (1000°C) annealing. P+ and N+ implants become source and drain. Annealing allows the P+ and the N+ regions to get penetrated deeper into the N-well 
and P-well respectively.

<img width="1198" height="572" alt="image" src="https://github.com/user-attachments/assets/7686b13a-d51c-4bad-888a-062581149a46" />

**7) Steps for Contacts and Local Interconnects**: Remove thin screen oxide through etching. Deposit titanium (Ti) via sputtering due to Ti's very low resistivity. Sputtering is hitting Titanium metal with Argon
gases. The titanium metal particles are deposited on the substrate.

<img width="1082" height="518" alt="image" src="https://github.com/user-attachments/assets/25e0c904-7adf-4150-8eb6-9b1801daad62" />

We have to create reaction between Ti layer and CMOS source, gate, and drain. Heat wafer at 650-700°C in $N_2$ ambient for 60 seconds. This produces titanium silicide ($TiSi_2$) which has low resistance on the
wafer. Another reaction between Ti and N produces TiN for local communication.

<img width="1081" height="527" alt="image" src="https://github.com/user-attachments/assets/1c08bde0-931f-4386-a0cb-bf6f3fe874b1" />

Using "mask 11" and photoresist, etch TiN to create specific contacts. Remove TiN using RCA cleaning. RCA solution consists of de-ionized water (5 parts), Ammonium hydroxide ($NH_4$OH, 1 part), Hydrogen peroxide 
($H_2$ $O_2$, 1 part) which etches the TiN

<img width="908" height="577" alt="image" src="https://github.com/user-attachments/assets/7f555da2-3af7-4314-90cc-61c59a9c817d" />

Local interconnects form after etching and photoresist removal.

<img width="937" height="533" alt="image" src="https://github.com/user-attachments/assets/3e448d7a-a0ba-46aa-9682-3605840e09d9" />

**8) Higher Level Metal Formation**: These steps resemble previous ones. Notice the non-planar surface, which creates metal discontinuity problems for metal interconnects. Planarize the surface by depositing thick
$SiO_2$ with impurities (Phosphorous). Impurities are used for various reasons mainly to ensure that the impurities act as the mobile Na ions present in $SiO_2$, then use CMP (chemical mechanical polishing) for 
planarization.

<img width="1125" height="551" alt="image" src="https://github.com/user-attachments/assets/712cf2a3-4c9d-41e4-8b57-80cecaabafa4" />

Using "mask 12" and photoresist, etch the SiO2 layer for metal deposition.

<img width="877" height="576" alt="image" src="https://github.com/user-attachments/assets/2ab64fb5-e398-4484-9d83-1807cc7496e6" />

Remove photoresist and deposit thin TiN layer (~10nm) over the wafer. TiN provides excellent adhesion for SiO2 and acts as a barrier between metal interconnect bottom and top layers.

<img width="1007" height="537" alt="image" src="https://github.com/user-attachments/assets/41362fd8-e7cd-4f3a-b441-85501931a1d9" />

Deposit blanket tungsten (W) layer over the wafer, then perform CMP for surface planarization.

W acts as contact holes connecting to higher metal layers, so deposit Al (aluminum) layer.

<img width="1131" height="521" alt="image" src="https://github.com/user-attachments/assets/ea814867-dbb7-4618-8b6f-011cf660dd2f" />

Using mask 13 and photoresist, etch Al layer through plasma etching to form contacts at specific locations.

<img width="881" height="462" alt="image" src="https://github.com/user-attachments/assets/9b571e72-b1f2-4576-887e-c26368a3790a" />

This completes the first metal interconnect level. Repeat the process for second metal interconnect level using "mask 14" for SiO2 etching and "mask 15" for Al layer etching.

<img width="905" height="571" alt="image" src="https://github.com/user-attachments/assets/6eb0a6d1-0fec-423e-a418-9b0dacf41fa9" />

The upper Al layer is thicker than the lower Al layer (resistance decreases). Finally, deposit $SiO_2$ or $Si_3$ $N_4$ layer for chip protection. Then "mask 16" is used to take out the metal contacts.

<img width="908" height="590" alt="image" src="https://github.com/user-attachments/assets/f68d61da-9e6a-4ba9-a098-f7350b113f50" />

The CMOS appears as shown after fabrication completion.

<img width="981" height="691" alt="image" src="https://github.com/user-attachments/assets/107c6cb6-0cc5-40f0-b205-f98327b62b26" />

