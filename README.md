
## VSD SoC Design and Planning Workshop

## Project Introduction
This repository provides a comprehensive walkthrough of the physical design flow, leveraging the OpenLane toolchain with the Sky130 open-source Process Design Kit (PDK). Developed as part of the Digital VLSI SoC Design and Planning Workshop by VLSI System Design Corporation, this project delivers hands-on experience in chip design, from initial Register Transfer Level (RTL) code to the final GDSII manufacturing file.

OpenLane is a robust, open-source automation framework that transforms RTL designs into GDSII layouts. It integrates essential tools such as Yosys, OpenROAD, Magic, Netgen, Fault, and OpenPhySyn, ensuring efficient and automated generation of clean, manufacturable chip layouts without the need for manual intervention. The flow is specifically optimized for the Skywater 130nm PDK, making it ideal for developing both hard macros and complete chips.


## Repository Structure

**Day 1:**  
Includes an introduction to RISC-V, the RTL to GDSII flow, and the OpenLane tool overview. Lab work covers synthesis, slack analysis, and estimation of flip-flop ratio.

**Day 2:**  
Focuses on floorplanning and placement concepts. Labs include floorplanning, placement, area estimation, and library cell characterization using OpenLane.

**Day 3:**  
Covers theoretical aspects like library cell design, CMOS process, and SPICE-based analysis. Labs demonstrate layout creation, SPICE extraction in Magic, and post-layout simulation in ngspice.

**Day 4:**  
Explores timing analysis with ideal clocks, delay tables, and clock tree synthesis. The lab applies CTS techniques and verifies timing reports.

**Day 5:**  
Dedicated to lab experiments involving synthesis refinement, timing closure, and clock tree analysis.


  


# DAY-1  
## Inception of Open-Source EDA, OpenLane, and Sky130 PDK

This session covers the foundational blocks of an Integrated Circuit (IC), including the chip, die, core, and interface elements. A QFN-48 package connects the silicon die to the outside world. The die contains the core—where actual logic (like processors or memory) resides—and is surrounded by pads that interface signals between the die and package pins. Foundry IPs like ADCs, DACs, and SRAM are pre-designed analog or mixed-signal blocks, while macro blocks like RISC-V SoCs and SPI interfaces are digital logic structures built on standard cells.

## Introduction to RISC-V

RISC-V is an open-source ISA (Instruction Set Architecture) known for its simplicity, modularity, and free-to-use model. It comprises a minimal base integer instruction set with optional extensions (like multiplication, atomic, or floating-point operations), making it flexible for educational and industrial use cases. Its open nature makes it ideal for customization, enabling hardware innovation without licensing barriers.

## RTL to GDSII Flow Overview

The design journey starts with **synthesis**, which converts RTL code (e.g., Verilog) into a gate-level netlist using standard cells defined by the Sky130 PDK. This netlist undergoes logical and area optimizations.  

Next, in **floorplanning**, the physical structure of the chip is defined—setting the die size, core area, and macro placements. Power distribution is established using upper metal layers to minimize IR drops.  

**Placement** involves arranging cells inside the core area. Global placement estimates rough positions to reduce wirelength, while detailed placement ensures legal placement adhering to design rules.  

In **Clock Tree Synthesis (CTS)**, the clock signal is buffered and routed in a balanced manner to all flip-flops using structured trees like H-trees or X-trees, minimizing skew and jitter.  

**Routing** connects pins and nets across placed cells using defined metal layers. Global routing defines approximate paths; detailed routing finalizes exact wiring with vias and spacing rules compliant with the PDK’s six-layer metal stack.  

**Verification before sign-off** ensures manufacturing viability. This includes DRC (Design Rule Check) for geometric constraints, LVS (Layout vs. Schematic) to confirm logical equivalence, and STA (Static Timing Analysis) to verify timing across all PVT corners.

🔗 [GDSII file format](https://anysilicon.com/semipedia/gdsii/)  
🔗 RTL to GDSII using [OpenLane](https://efabless.com/openlane)


## OpenLane Flow

<img width="848" height="=500" alt="182759711-6b9352ec-7652-4589-af31-53a409eb2830" src="https://github.com/user-attachments/assets/f3cdde11-73e1-496e-9486-3d02f62ff65d"/>


OpenLane is an automated RTL to GDSII flow that utilizes various components such as OpenROAD, Yosys, Magic, Netgen, and custom scripts for design exploration and optimization.

The flow performs all ASIC implementation steps from RTL down to GDSII. OpenLane Interactive Mode Commands include a variety of functions such as setting the current netlist, running logic verification, setting the current def file, preparing lef files, preparing a liberty file, generating an exclude list file, sourcing configurations, specifying a path to save config_in.tcl, preparing a run, specifying a design folder, overwriting an existing run, specifying a path to save the run, specifying a name for a specific run, creating a tcl configuration file for a design, setting the verilog source code file, specifying the design's configuration file, setting a verbose output level, generating a padframe, saving views of a given run, changing save paths for various file types, labeling pins of a macro def, generating a verilog netlist from a def file, creating an obstruction, setting tracks on a layer, extracting core dimensions, running SPEF extraction and Static Timing Analysis, running antenna checks, saving environment variables, running OpenSTA timing analysis, checking for unmapped cells, checking for assign statements, and checking if the LEF was properly read.

- *Antenna Rules Violation* : long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to either use bridging or antenna diode insertion to leak away the charges.  

### OpenLane Directory Hierarchy:

``` 
├── OOpenLane             -> directory where the tool can be invoked (run docker first)
│   ├── designs          -> All designs must be extracted from this folder
│   │   │   ├── picorv32a -> Design used as case study for this workshop
│   |   |   ├── ...
|   |   ├── ...
├── pdks                 -> contains pdk related files 
│   ├── skywater-pdk     -> all Skywater 130nm PDKs
│   ├── open-pdks        -> contains scripts that makes the commerical PDK (which is normally just compatible to commercial tools) to also be compatible with the open-source EDA tool
│   ├── sky130A          -> pdk variant made especially compatible for open-source tools
│   │   │  ├── libs.ref  -> files specific to node process (timing lib, cell lef, tech lef) for example is `sky130_fd_sc_hd` (Sky130nm Foundry Standard Cell High Density)  
│   │   │  ├── libs.tech -> files specific for the tool (klayout,netgen,magic...) 
```

Inside a specific design folder contains a `config.tcl` which overrides the default settings on OpenLANE. These configurations are specific to a design (e.g. clock period, clock port, verilog files...). The priority order for the OpenLANE settings:
1. sky130_xxxxx_config.tcl in `OpenLane/designs/[design]/`
2. config.tcl in `OpenLane/designs/[design]/`
3. Default values in `OpenLane/configuration/`



### Lab [Day 1] - Determine Flip-flop Ratio:
The task is to find the flip-flop ratio ratio for the design `picorv32a`. This is the ratio of the number of flip flops to the total number of cells. For the OpenLane installation, the steps are very straight forward and can be found on the [OpenLane repo](https://github.com/The-OpenROAD-Project/OpenLane).

**1. Run OpenLANE:**
 - `$ make mount` = Open the docker platform inside the `openlane/`
 - `% flow.tcl -interactive` = run script for automating the whole RTL to GDSII flow but in step by step `-interactive` mode
 - `% package require openlane 0.9` == retrives all dependencies for running v0.9 of OpenLANE  
![FScreenshot](https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-1/Screenshot%202025-07-31%20123701.png?raw=true)


 
**2. Design Setup Stage:**
 - `% prep -design picorv32a` = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a `run/` folder inside the specific design directory which contains the command log files, results, and the reports dump by each tools. These folders will be empty for now except for lef files generated by this design setup stage. This merged the [cell LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.lef` and [technology LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.tlef` generating `merged.nom.lef` inside `run/tmp/`
 
![Synthesis Report - Cell Counts](https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-1/Screenshot%202025-07-31%20123914.png?raw=true)



**3. Run synthesis:**
 - `% run_synthesis` = Run yosys RTL synthesis, ABC scripts (for technology mapping), and OpenSTA.  
 
![Flip-Flop Ratio Formula](https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-1/flipflop-ratio.jpg?raw=true)



**The flipflop ratio is (number of flip flops)/(total number of cells) is 1613/14876 = 0.10843. Or 10.843%**



# Day 2 
# Good Floorplan vs Bad Floorplan and Introduction to Library Cells

#### Floorplan

The floorplanning process begins with determining the core and die dimensions, where the core is the central region where standard logic cells are placed. The size of the core depends on the standard cell dimensions present in the netlist, and its utilization factor—typically between 0.5 and 0.6—represents the ratio of the netlist area to the total core area, leaving enough space for routing and additional filler or buffer cells. Another important metric is the aspect ratio, calculated as height divided by width; an aspect ratio of 1 indicates a square-shaped core, which is often ideal for routing efficiency.

Next, preplaced cells, which include complex and reusable IP blocks like memories or clock-gating cells, are defined by the user and fixed prior to the automated placement and routing steps. These cells cannot be moved by tools afterward. To ensure power integrity, decoupling capacitors are placed near preplaced cells to stabilize the local voltage supply, especially in the presence of long power lines where resistance and inductance can cause IR drop. These capacitors provide localized charge during fast switching events, keeping the voltage within acceptable noise margins.

However, decoupling capacitors alone are not enough for stable power delivery across the entire chip. To combat issues like ground bounce (from many cells switching to logic 0) and voltage droop (from simultaneous switching to logic 1), a robust power mesh is employed. This mesh distributes VDD and VSS taps uniformly across the chip to ensure consistent current flow and reduce voltage fluctuations.

Finally, pin placement is performed, where I/O ports are positioned between the core and the die boundary based on the location of the logic blocks they connect to. Clock pins are given wider tracks to reduce resistance and ensure reliable clock distribution across the design. Additionally, logical cell placement blockages are defined in areas around the pins to prevent standard cells from being placed over or too close to pin locations, which ensures successful routing and prevents congestion near the die edge.


#### Placement

Placement in digital ASIC design begins with netlist binding, where the logical representation of a circuit is mapped to physical standard cell shapes from a library. Each gate, flip-flop, or logic element in the synthesized netlist is associated with a predefined layout view from the standard cell library. During the initial placement phase, these mapped cells are positioned within the core area of the chip, with key emphasis on proximity to I/O pins for minimizing signal delays. Critical signal paths, such as those between flip-flops (e.g., FF1 to FF2), are shortened by placing related components closer together, and buffer insertion may be used for better signal integrity. Wire-length and capacitance estimates further influence this phase to optimize for performance metrics like timing, power, and noise. Final placement optimization then refines this layout by considering real design constraints, still assuming an ideal clock, and aims to minimize path delays and power usage. At this point, the design has transitioned from logical mapping and floorplanning to a physical representation, with standard cell rows prepared and components now optimally placed for routing.






## Lab 2 

#### Floor Plan

- Run Floorplan : ```run floor_plan```

- To view our floorplan in Magic we need :

  1. Magic technology file (sky130A.tech)
  2. Def file of floorplan
  3. Merged LEF file

  head over to the following directory to view the results of floorplan using Magic :

  ```cd /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/results/floorplan```

  To invoke magic use the command :

  ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &```
  
 <img width="700" alt="Day 2 - OpenLane Home View" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-2/Screenshot%202025-07-31%20144352.png?raw=true" />


  *Result* : 
  
<img width="720" alt="Day 2 - Design Selection in Terminal" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-2/Screenshot%202025-07-31%20144417.png?raw=true" />


  To center the view, press "s" to select whole die then press "v" to center the view. Point the cursor to a cell then press "s" to select it, zoom into it by pressing 'z". Type "what" in tkcon to display          information of selected object. These objects might be IO pin, decap cell, or well taps as shown below.
  
<img width="750" alt="Day 2 - Synthesis Launched" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-2/Screenshot%202025-07-31%20144711.png?raw=true" />

  
  if we zoom, we can see that some of the micro, IO pad, and tap-cells have been placed appropriately.

<img width="770" alt="Day 2 - Synthesis Complete" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-2/Screenshot%202025-07-31%20144751.png?raw=true" />
  
  To get information about the selected object press ```s``` and type ```what``` in console - same as in the above image

#### Place Ment 

- To do a placement in OpenLane : ```run_placement```

- Viewing Placement in Magic :

  ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &```

 <img width="800" alt="Day 2 - Flip-Flop Count and Cell Count" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-2/Screenshot%202025-07-31%20145850.png?raw=true" />


  *results* :
  
<img width="680" alt="Day 2 - Screenshot of Ratio Calc Part 2" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-2/screenshot%2014.57.00.jpg?raw=true" />

<img width="680" alt="Day 2 - Screenshot of Ratio Calc Part 1" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-2/screenshot%20%2014.56.47.jpg?raw=true" />

  
  If we zoom, we the core with all the standard cells placed in between power can ground rail




# Day 3
# Design a Library Cell using Magic Layout and Ngspice Characterization

#### Designing a Library Cell 

  

  
  
#### Follow these steps for simulation in ngspice :
 
  1. Source the circuit file in Ngspice using source <file_name>.cir.
  2. Execute the simulations using the run command.
  3. Use setplot to prepare for plotting.
  4. For DC analysis (as indicated in the .cir file), use dc1 to prepare the DC plot.
  5. Display available vectors using display.
  6. Plot specific vectors, e.g., plot vout vs vin, to visualize the circuit behavior.

#### 16-Mask CMOS Process  
*inverter*

The CMOS fabrication process begins with **substrate selection**, where a `P-type Si wafer` with resistivity between 5–50 Ω·cm and ⟨100⟩ orientation is chosen. It's essential that the substrate doping is less than the well doping to ensure proper well formation.  

Next, in **active region isolation (LOCOS)**, `Mask 1` is used to define photoresist patterns that protect future transistor areas. A layer of `Si₃N₄` (~80nm) is deposited to block oxide growth, followed by the growth of field oxide (`LOCOS`) in exposed regions (~1µm), effectively isolating NMOS and PMOS active regions.  

**Well formation** involves creating the `N-well` for PMOS using `Mask 2` to shield the NMOS area, with phosphorus implanted at `@400 keV`. Similarly, the `P-well` for NMOS is formed using `Mask 3` with boron implants at `@200 keV`, followed by drive-in diffusion to deepen the wells.  

For **gate formation**, threshold voltages are controlled using `Mask 4/5` to implant boron in NMOS and arsenic in PMOS regions. A high-quality gate oxide is formed by etching and regrowing 10nm of `SiO₂`. `Mask 6` is then used to pattern the polysilicon (`Poly-Si`) gates.  

In the **Lightly Doped Drain (LDD)** phase, `Mask 7` is used for `N⁻` (Phosphorus) implantation in NMOS and `Mask 8` for `P⁻` (Boron) implantation in PMOS. Sidewall spacers are formed using `SiO₂` deposition and anisotropic etching to shield LDD regions during further processing.  

During **source/drain formation**, heavily doped regions are created using `Mask 9` for `N⁺` (Arsenic) in NMOS and `Mask 10` for `P⁺` (Boron) in PMOS. A screen oxide is added to prevent implant channeling.  

For **contacts and local interconnects**, a titanium layer is sputtered and then subjected to Rapid Thermal Annealing (`600–700°C`) to form `TiSi₂` over gates and `TiN` for routing. `Mask 11` is used to etch and define the first contact layer.  

Finally, in the **metallization (Al/W)** stage, planarization is achieved with PSG deposition and CMP. `Mask 12` and `Mask 14` are used for via etching, while `Mask 13` and `Mask 15` deposit Aluminum or Tungsten. `Mask 16` defines the top-level contact and pad openings.  

**Key Insights:** PMOS devices are made wider than NMOS (`PMOS Width > NMOS`, typically 2–3×) to balance drive strength. The LDD structure reduces leakage and enhances reliability, with sidewall spacers offering implant protection. `TiSi₂` helps lower gate resistance, while `TiN` is useful for local routing. Overall, the process uses `16 masks` to define all critical structures including wells, implants, gates, contacts, and metal layers.




## Lab 3 

#### Spice extraction of inverter in magic 

- Clone vsdstdcelldesign. Copy the techfile ```sky130A.tech``` from ```pdks/sky130A/libs.tech/magic/``` to directory of the cloned repo. 
<img width="900" height="200" alt="Day 3 - Screenshot 154025" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-3/Screenshot%202025-07-31%20154025.png?raw=true" />

<img width="850" height="160" alt="Day 3 - Screenshot 154038" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-3/Screenshot%202025-07-31%20154038.png?raw=true" />


- View the mag file using magic ```magic -T sky130A.tech sky130_inv.mag &```
<img width="880" height="300" alt="Day 3 - Screenshot 154353" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-3/Screenshot%202025-07-31%20154353.png?raw=true" />

- Make an extract file ```.ext``` by typing extract all in the tkon terminal of magic. 

- Extract the ```.spice``` file from this ext file by typing ```ext2spice cthresh 0 rthresh 0``` then ```ext2spice``` in the tkon terminal.

  <img width="848" height="260" alt="Screenshot 2025-07-26 174858" src="https://github.com/user-attachments/assets/76fb85c3-e007-4032-9925-8800b238c3e0" />

<img width="848" height="721" alt="Day 3 - Screenshot 154632" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-3/Screenshot%202025-07-31%20154632.png?raw=true" />

  
#### Post-Layout Spice simulation (ngspice)

- Open the spice file by typing ```ngspice sky130A_inv.spice``` and Generate a graph using plot y vs time a
  
  <img width="848" height="875" alt="Screenshot 2025-07-26 181256" src="https://github.com/user-attachments/assets/6b11a52d-e7dd-4626-882e-e8d24f331810" />

<img width="848" height="490" alt="Day 3 - Screenshot 155306" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-3/Screenshot%202025-07-31%20155306.png?raw=true" />


  

#### Fix Tech File DRC via Magic


All technology-specific information comes from a technology file. This file includes such information as layer types used, electrical connectivity between types,     design rules, rules for mask generation, and rules for extracting netlists for circuit simulation - [DRC rules for SKY130nm PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root)

Clone custom inverter standard cell design :

```
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the repository with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign

# Change into repository directory
cd vsdstdcelldesign
 
# Copy magic tech file to the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# Check contents whether everything is present
ls

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```

- Inside the drc_tests/ are the .mag layout files and the sky130A.tech.
- Open magic with ```poly.mag``` as input: magic poly.mag from file --> open
- Focus on Incorrect poly.9.the spacing between polyresistor with poly or diff/tap must at least be 0.480um. Using command box in console, we can see that   the distance violated but there is no DRC violations shown. Our goal is to fix the tech file to include that DRC.
<img width="700" height="50" alt="Screenshot 2025-07-26 232040" src="https://github.com/user-attachments/assets/102cbe43-321e-451d-8e0e-82f87013bd23" />


<img width="700" height="400" alt="Screenshot 2025-07-26 221828" src="https://github.com/user-attachments/assets/48b3df34-f795-4d5b-9c47-09462b66b709" />
<img width="800" height="500" alt="Screenshot 2025-07-26 223639" src="https://github.com/user-attachments/assets/d9732a2e-a38e-49a6-9374-ab0a0183dd06" />
<img width="400" height="137" alt="Screenshot 2025-07-26 232204" src="https://github.com/user-attachments/assets/d39c2834-fb34-45d8-8f2f-93e4ffb25187" />

The current sky130A.tech file only includes spacing rules for:

- n-poly resistor to n-diffusion

- p-poly resistor to p-diffusion

We are now adding two new critical spacing rules (highlighted in green):

- Left rule: Spacing between n-poly resistor and regular poly (non-resistor)

- Right rule: Spacing between p-poly resistor and regular poly (non-resistor)


To apply the new poly resistor spacing rules:

Run ```tech load sky130A.tech``` to update the rules.

Use ```drc check``` – violations appear as white dots on poly layers.

Inspect each violation with ```drc find```

New rules cover:

n-poly ↔ poly spacing

p-poly ↔ poly spacing

<img width="400" height="87" alt="Screenshot 2025-07-26 223505" src="https://github.com/user-attachments/assets/a6ec936c-e03e-440c-9729-59cd987577ab" />

# DAY 4 
# Pre-layout Timing Analysis and Importance of Good Clock Tree 

## Theory

To minimize clock skew between endpoints of a clock tree and ensure signals arrive simultaneously, it's essential to maintain equal capacitive loading and uniform buffer sizing at each hierarchical level. Buffers at the same level must drive identical loads and be of the same size to ensure consistent timing delay and prevent mismatches in resistance, W/L ratios, and RC constants. While different levels in the clock tree may use varying buffer sizes and handle different loads, maintaining symmetry within each level ensures balanced cumulative delays across all branches. This design principle effectively eliminates clock skew by synchronizing signal propagation. The delay characteristics of these buffers are captured in liberty files using delay tables, where output slew—defined by the capacitive load and input transition time—serves as the primary delay metric. The input transition time, influenced by the driving buffer’s output and internal delay, forms a cascaded chain of timing dependencies, making accurate modeling essential for skew control and timing predictability.

During pre-layout static timing analysis (STA), ideal clocks are used for early verification. These clocks do not yet reflect physical implementation details such as buffer delays or RC parasitic effects. Instead, interconnect delays are estimated using wire models from the process design kit (PDK), which provide technology-dependent approximations. Although this pre-layout STA does not account for actual routing paths or parasitic extractions, it allows for early detection of major timing issues. Final timing validation is deferred to post-layout STA, where actual buffer placements and extracted RC values from the physical layout are analyzed.

In the clock tree synthesis (CTS) stage, three main objectives guide the process. First, minimizing clock skew involves designing balanced trees with equal wire lengths and matched delays to all clock endpoints, ensuring synchronous signal delivery. Second, maintaining clock signal integrity requires specialized clock buffers that ensure controlled slew rates and symmetrical rise/fall times, compensating for RC effects in long interconnects. Third, mitigating crosstalk is achieved by shielding critical clock nets—typically by placing power (VDD) or ground lines alongside them—to reduce capacitive coupling with neighboring aggressor nets. This shielding approach can also be applied to other timing-critical signals. Collectively, these practices ensure that the clock network remains robust, with consistent timing, strong signal integrity, and minimal noise interference across the chip.



## Lab 4

- Tracks.info used in routing stage (`/desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd`)

<img width="848" height="496" alt="Screenshot 2025-07-31 163116" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-07-31%20163116.png" />

<img width="848" height="500" alt="Screenshot 2025-07-31 163319" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-07-31%20163319.png" />




  
- Type the Command in tkcon window to set grid as tracks of locali layer
   ```grid 0.46um 0.34um 0.23um 0.17um```
<img width="848" height="530" alt="Screenshot 2025-08-01 121144" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20121144.png" />

<img width="848" height="515" alt="Screenshot 2025-08-01 120846" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20120846.png" />



- Two Things to verify : Pins lies on intersections and cell width is 3. We can make use of grids to identify cell width.

- Extracting `LEF` file

  - open the mag file 
    ```magic -T sky130A.tech sky130_inv.mag &```
  - save the inverter by your custom name save `sky130_vsdinv.mag`
    then type `lef write` in tkcon window. This will create files as shown below.
    
  <img width="848" height="485" alt="Screenshot 2025-08-01 120201" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20120201.png" />



    

- Pluging-in Custom Inverter Cell into Openlane

  - Copy the LEF file `sky130_vsdinv.lef` and `sky130_fd_sc_hd__*` from `openlane/vsdstdcelldesign/libs` to `picorv32a/src` directory.
<img width="848" height="518" alt="Screenshot 2025-08-01 122422" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20122422.png" />



 
  - add these commands into `config.tcl` in `Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a`
    ```
    set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"
    set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__fast.lib"
    set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__slow.lib"
    set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"

    set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
    ```
    
<img width="848" height="505" alt="Screenshot 2025-08-01 122915" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20122915.png" />




  - Run docker and prepare the design picorv32a. You may make use of overwrite command : `prep -design picorv32a -tag <date> -overwrite`
 <img width="848" height="520" alt="Screenshot 2025-08-01 123734" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20123734.png" />




  - setting lefs
<img width="848" height="498" alt="Screenshot 2025-08-01 124236" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20124236.png" />



 
    ```
    set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
    add_lefs -src $lefs
    ```
    
  - running synthesis along chip area ```run_synthesis```
 <img width="848" height="510" alt="Screenshot 2025-08-01 124418" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20124418.png" />

<img width="848" height="495" alt="Screenshot 2025-08-01 124443" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20124443.png" />


 
    we get to see --> chip area, wns (worst timing violation) and tns (total negative slack)

    merged.lef in tmp directory with our custom inverter as macro
    <img width="848" height="510" alt="Screenshot 2025-08-01 124418" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-4/Screenshot%202025-08-01%20124418.png" />

- we change some variable to get better timing improvement. This results change in chip area.so we'll change some variables because Those timing values aren't          ideal.

    ```
    echo $::env(SYNTH_STRATEGY)
    
    set ::env(SYNTH_STRATEGY) "DELAY 3"

    echo $::env(SYNTH_BUFFERING)
    // make sure its enabled i.e 1

    echo $::env(SYNTH_SIZING)
   
    set ::env(SYNTH_SIZING) 1

    echo $::env(SYNTH_DRIVING_CELL
    // check whether it's the proper cell or not
    
 <img width="880" height="512" alt="Screenshot 2025-08-01 141231" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20141231.png?raw=true" />


- run floorplan

  run these commands instead :
  ```
  init_floorplan
  place_io
  tap_decap_or
  ```
<img width="890" height="498" alt="Screenshot 2025-08-01 143140" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20143140.png?raw=true" />

- run placement and we obtain `DEF` file
  opening the file using magic by command --> `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read        ../../tmp/merged.lef def read picorv32a.placement.def &`

  <img width="848" height="496" alt="Screenshot 2025-07-31 145850" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-07-31%20145850.png?raw=true" />


 <img width="848" height="498" alt="Screenshot 2025-08-01 143508" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20143508.png?raw=true" />


 <img width="848" height="498" alt="Screenshot 2025-08-01 143705" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20143705.png?raw=true" />

<img width="848" height="498" alt="Screenshot 2025-08-01 143614" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20143614.png?raw=true" />


- Post-Synthesis timing analysis

  Since we are having improved timing of 0 wn. so what we will do is we will do timing analysis on Synthesis that has lot of violations
  - we gradually improve and reduce our slack by replacing cells with better one and to deliver improved delay.
  This is **timing ECO fixes**

  First we do is running the synthesis --> include newly added lef to openlane flow --> set ::env(SYNTH_SIZING) 1 --> set                ::env(SYNTH_MAX_FANOUT) 4 --> run_synthesis

  <img width="848" height="487" alt="Screenshot 2025-08-01 145003" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20145003.png?raw=true" />

  

  Later in `cd Desktop/work/tools/openlane_working_dir/openlane` we write command `sta pre_sta.conf`


  *my_base.sdc*
  
  <img width="848" height="487" alt="Screenshot 2025-08-01 145050" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20145050.png?raw=true" />


 <img width="848" height="486" alt="Screenshot 2025-08-01 153031" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153031.png?raw=true" />


  Replacing some cells to reduce slack

<img width="848" height="486" alt="Screenshot 2025-08-01 153208" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153208.png?raw=true" />


  report_net -connections _11672_ [we see the driver pins] 
  
 <img width="848" height="486" alt="Screenshot 2025-08-01 153233" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153233.png?raw=true" />


  replace_cell _14510_ sky130_fd_sc_hd__or3_4 [we replace some cells with better suited ones for driving 4 fanouts]
  
 <img width="848" height="486" alt="Screenshot 2025-08-01 153314" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153314.png?raw=true" />


  To check the report and see the changes in slack we use command : report_checks -fields {net cap slew input_pins} -digits 4
  Slack reduced
  
<img width="848" height="487" alt="Screenshot 2025-08-01 153424" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153424.png?raw=true" />


  report_checks -from _29052_ -to _30440_ -through _14510_
  
<img width="848" height="487" alt="Screenshot 2025-08-01 153448" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153448.png?raw=true" />


<img width="848" height="487" alt="Screenshot 2025-08-01 153511" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153511.png?raw=true" />


  report_net -connections _11668_
  
  replace_cell _14506_ sky130_fd_sc_hd__or4_4
<img width="848" height="487" alt="Screenshot 2025-08-01 153546" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153546.png?raw=true" />


  Slack Reduced
  
<img width="848" height="487" alt="Screenshot 2025-08-01 153608" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153608.png?raw=true" />


  Intially : -23.90
  now : -22.9860
  reduced around 0.914 ns of violation
 

- Run CTS
  
  - Here we proceed with earlier 0 violation design. we want to proceed with the clean design to further stage.
  - run synthesis,floorplan,placement and then run cts (`run_cts`)

<img width="848" height="487" alt="Screenshot 2025-08-01 153722" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153722.png?raw=true" />

<img width="848" height="487" alt="Screenshot 2025-08-01 153910" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153910.png?raw=true" />


  

- Post-CTS Timing analysis : OpenROAD

  command : `openroad`
  
<img width="848" height="487" alt="Screenshot 2025-08-01 153947" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20153947.png?raw=true" />

<img width="848" height="487" alt="Screenshot 2025-08-01 154014" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154014.png?raw=true" />


<img width="848" height="487" alt="Screenshot 2025-08-01 154032" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154032.png?raw=true" />

<img width="848" height="487" alt="Screenshot 2025-08-01 154052" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154052.png?raw=true" />


  removing 'sky130_fd_sc_hd__clkbuf_1 and running CTS again 
  
<img width="848" height="487" alt="Screenshot 2025-08-01 154212" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154212.png?raw=true" />


<img width="848" height="487" alt="Screenshot 2025-08-01 154254" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154254.png?raw=true" />


  echo $::env(CTS_CLK_BUFFER_LIST) [Checking current value of CTS_CLK_BUFFER_LIST]
  
  <img width="848" height="487" alt="Screenshot 2025-08-01 154321" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154321.png?raw=true" />



  running openROAD command again
  
  ```
  openroad

  read_lef /openLANE_flow/designs/picorv32a/runs/<date>/tmp/merged.lef

  read_def /openLANE_flow/designs/picorv32a/runs/<date>/results/cts/picorv32a.cts.def

  write_db pico_cts1.db

  Loading the created database in OpenROAD
  read_db pico_cts.db

  read_verilog /openLANE_flow/designs/picorv32a/runs/<date>/results/synthesis/picorv32a.synthesis_cts.v

  read_liberty $::env(LIB_SYNTH_COMPLETE)

  link_design picorv32a

  read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

  set_propagated_clock [all_clocks]

  report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

  report_clock_skew -hold

  report_clock_skew -setup
  ```

<img width="848" height="487" alt="Screenshot 2025-08-01 154355" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154355.png?raw=true" />

<img width="848" height="487" alt="Screenshot 2025-08-01 154413" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154413.png?raw=true" />


  Checking clock skew for setup and hold
  
  <img width="848" height="487" alt="Screenshot 2025-08-01 154439" src="https://github.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day-4/Screenshot%202025-08-01%20154439.png?raw=true" />




# DAY 5 
# Final steps for RTL2GDS using tritonRoute and openSTA

## Lab 5 

## Power Distribution Network (PDN)

Power Distribution Network (PDN) is a crucial step in digital IC design. It ensures that power (VDD) and ground (VSS) are distributed uniformly across the entire chip with minimal voltage drop (IR drop) and high reliability. This becomes particularly important after the clock tree synthesis (CTS) stage, as the placement of clock buffers and registers increases the power demand in various regions of the design.

---

### Steps to Implement PDN in OpenLane

- Launch the OpenLane environment inside Docker.
- Prepare the design as usual by creating the configuration and ensuring your custom standard cell LEF (Library Exchange Format) is included correctly.
- Update configuration variables:
  - `SYNTH_STRATEGY` should be set to `DELAY 3` — this strategy optimizes the synthesized logic for timing, favoring delay optimization over area or power.
  - `SYNTH_SIZING` should be set to `1` — enabling basic sizing of gates during synthesis to meet timing constraints.

- Run the following stages in sequence:
  - **Synthesis**: Generates gate-level netlist from RTL using the configured synthesis strategy.
  - **Floorplan**: Defines the chip dimensions, core area, and block placements.
  - **Placement**: Places all the standard cells within the defined core area.
  - **CTS (Clock Tree Synthesis)**: Constructs a balanced clock tree to distribute the clock signal with minimal skew.

---

### Generating the Power Distribution Network

Once CTS is complete, the next step is to generate the PDN grid.

- Use the `gen_pdn` command inside OpenLane to create the power stripes and connections.
- PDN typically utilizes higher metal layers (e.g., met4/met5/met6 in Sky130 PDK) for vertical and horizontal power routing to minimize resistance and avoid congestion with signal nets.
- This network ensures that every cell gets connected to power and ground rails and that the voltage drop across the die remains within acceptable limits.

The PDN generation is essential to make the design physically realizable and manufacturable with stable power delivery.


 <img width="848" height="490" alt="Screenshot 2025-08-01 214116" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214116.png" />


<img width="848" height="490" alt="Screenshot 2025-08-01 214220" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214220.png" />


<img width="848" height="490" alt="Screenshot 2025-08-01 214310" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214310.png" />

<img width="848" height="495" alt="Screenshot 2025-08-01 214351" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214351.png" />

 <img width="848" height="505" alt="Screenshot 2025-08-01 214407" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214407.png" />

<img width="848" height="495" alt="Screenshot 2025-08-01 214436" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214436.png" />


  - In `<date>/tmp/floorplan/` directory we  have the pdn
    to open we use command : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef      def read 14-pdn.def &`

  <img width="848" height="490" alt="Screenshot 2025-08-01 214519" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214519.png" />
  
<img width="848" height="495" alt="Screenshot 2025-08-01 214556" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214556.png" />

- Perfrom detailed routing using TritonRoute

  - run routing using command --> `run_routing`
  
 <img width="848" height="502" alt="Screenshot 2025-08-01 214637" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214637.png" />

<img width="848" height="495" alt="Screenshot 2025-08-01 214700" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214700.png" />

 
    load the def file : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def      read picorv32a.def &`
    
<img width="848" height="495" alt="Screenshot 2025-08-01 214723" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214723.png" />

   <img width="848" height="500" alt="Screenshot 2025-08-01 214745" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214745.png" />


    fast route guide inside `<date>/tmp/routing`

 <img width="848" height="500" alt="Screenshot 2025-08-01 214802" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214802.png" />


- Post-Route parasitic extraction using SPEF extractor
  
  for the extracting, we use [SPEF_EXTRACTOR](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
  
  ```
  cd Desktop/work/tools/SPEF_EXTRACTOR
  
  python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/tmp/merged.lef      /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/results/routing/picorv32a.def
  ```
  
- Post-Route OpenSTA timing analysis

<img width="848" height="505" alt="Screenshot 2025-08-01 214832" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20214832.png" />

  
  

**Final (Picorv32a.Def)**


<img width="848" height="510" alt="Screenshot 2025-08-01 215103" src="https://raw.githubusercontent.com/Dhanushgp-Sahyadri-ECE/Digital-VLSI-SoC-Openlane-sky130A/main/Day-5/Screenshot%202025-08-01%20215103.png" />




**[DEF](https://teamvlsi.com/2020/08/def-file-in-vlsi-design-exchange.html)**

**[GDSII](https://en.wikipedia.org/wiki/GDSII)**


# Acknowledgements

[Kunal Ghosh](https://github.com/kunalg123) – Founder, VLSI System Design Corp.

[Nickson Jose](https://github.com/nickson-jose)– Developer & Contributor, Open Source Physical Design




     




    






  
  
  

  

  

    

 
    
    

    

    
 
    







  
