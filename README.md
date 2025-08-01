
## Physical Design using OpenLANE & Sky130 PDK

## 📘 Project Introduction
This repository provides a comprehensive walkthrough of the physical design flow, leveraging the OpenLane toolchain with the Sky130 open-source Process Design Kit (PDK). Developed as part of the Digital VLSI SoC Design and Planning Workshop by VLSI System Design Corporation, this project delivers hands-on experience in chip design, from initial Register Transfer Level (RTL) code to the final GDSII manufacturing file.

OpenLane is a robust, open-source automation framework that transforms RTL designs into GDSII layouts. It integrates essential tools such as Yosys, OpenROAD, Magic, Netgen, Fault, and OpenPhySyn, ensuring efficient and automated generation of clean, manufacturable chip layouts without the need for manual intervention. The flow is specifically optimized for the Skywater 130nm PDK, making it ideal for developing both hard macros and complete chips.



## 📁 Repository Structure

- [DAY 1](#DAY-1)
  - [Theory](#DAY-1)
    - [Introduction to RISC-V](#Introduction-to-RISC-V)
    - [Simplified RTL to GDSII Flow](#Simplified-RTL-to-GDSII-Flow)
    - [OpenLane Flow](#OpenLane-Flow)
  - [Lab](#Lab-1)
    - [Synthesis](#Lab-1)
    - [Estimation of Flip Flop Ratio](#Estimation-of-Flip-Flop-Ratio)
    - [Slack](#Slack)
      
- [DAY 2](#Day-2)
  - [Theory](#DAY-2)
    - [Floorplan](#Floorplan)
    - [Placement](#Placement)
  - [Lab](#Lab-2)
    - [Floorplan](#Floor-Plan)
    - [Placement](#Place-Ment)
    - [Characterization](#Library-Characterization)
    - [Estimation of area of the die](#Estimation-of-area-of-the-die)
      
- [Day 3](#DAY-3)
  - [Theory](#DAY-3)
    - [Designing a Library Cell](#Designing-a-Library-Cell)
    - [steps for simulation in ngspice](#Follow-these-steps-for-simulation-in-ngspice)
    - [SPICE Switching Threshold and Propagation Delay](#SPICE-Switching-Threshold-and-Propagation-Delay)
    - [16-Mask CMOS Process](#16-Mask-CMOS-Process)
  - [Lab](#Lab-3)
    - [Spice extraction of inverter in magic](#Spice-extraction-of-inverter-in-magic)
    - [Post-Layout Spice simulation (ngspice)](#Post-Layout-Spice-simulation-(ngspice))
    - [Fix Tech File DRC via Magic](#Fix-Tech-File-DRC-via-Magic)
      
- [DAY 4](#DAY-4)
  - [Theory](#DAY-4)
    - [Delay Table](#Delay-Table)
    - [Timing Analysis (using Ideal Clocks)](#Timing-Analysis-(using-Ideal-Clocks))
    - [Clock Tree Synthesis Stage](#Clock-Tree-Synthesis-Stage)
  - [Lab](#Lab-4)
    
- [DAY 5](#DAY-5)
  - [Lab](#Lab-5)
  


# DAY-1
# Inception-of-Open-source-EDA,OpenLane-and-Sky130-PDK

we explore the fundamental physical elements of an integrated circuit (IC) : 

**Package, Chip, Pads, Core, Die, and IPs** : 
- The QFN-48 (Quad Flat No-lead) package is a surface-mounted IC package with 48 pins used to connect the chip to the outside world.

- Inside the package is the die, the actual silicon chip where transistors are fabricated.

- The core is the functional part of the die where the logic (e.g., processor, memory) is implemented.

- Surrounding the core are pads, which act as electrical contact points for interfacing signals from the core to the pins on the package.
  
- The chip as a whole includes the die, package, and connections.

The core of the chip will contain two types of blocks:

**Foundry IP Blocks** (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.

**Macro blocks** (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts.

<img width="848" height="853" alt="182751377-2810d388-21b0-4df1-b1d4-c72176d80d28" src="https://github.com/user-attachments/assets/d6c9f443-dc08-4587-972a-c46c9b927dad" />

## Introduction to RISC-V

- RISC-V stands for Reduced Instruction Set Computing – Version 5 and is designed with simplicity and modularity in mind.

- Unlike proprietary ISAs like ARM or x86, RISC-V is open, meaning anyone can implement or modify it freely.

- The architecture consists of a base integer set and optional extensions (e.g., for multiplication, atomic operations, floating point).

- RISC-V is ideal for education, research, and industrial design due to its transparency and scalability.

## Simplified RTL to GDSII Flow 

*Synthesis*

The Register Transfer Level (RTL) code, typically written in Verilog or VHDL, is translated into a gate-level netlist. This netlist is composed of logic gates and components from a pre-defined standard cell library. These cells have fixed sizes and electrical characteristics, provided by the Process Design Kit (PDK). During synthesis, optimizations such as constant propagation, logic simplification, and technology mapping are performed to ensure an area-efficient and logically equivalent design.

*Floor Planning and Power Planning*

This step involves defining the physical layout of the chip, including the die area, core area, margins, and reserved regions for IP blocks or macros. Floorplanning also includes decisions on I/O pin placement and defining power domains.
In power planning, a Power Distribution Network (PDN) is created to deliver clean power across the chip. The power rails and straps are usually placed on upper metal layers since they are thicker and have lower resistance, which helps reduce IR drop and ensures reliable power delivery.

*Placement*

Placement determines the exact physical location of standard cells within the defined floorplan. This process occurs in two phases:

- Global Placement: Estimates optimal positions to reduce wirelength and congestion, but may not obey all design rules.

- Detailed Placement: Adjusts the global result to ensure legal placement while minimizing additional wirelength or timing penalties.

*Clock Tree Synthesis (CTS)*

A clock tree is built to distribute the clock signal to all sequential elements (flip-flops) in the design. Since all flip-flops must receive the clock signal simultaneously to avoid timing violations like skew and jitter, structures such as H-trees or X-trees are used. CTS also buffers the clock signal and ensures balanced timing paths across different regions.

*Routing*

Routing connects the logically associated pins (nets) across the placed cells using horizontal and vertical metal layers defined in the PDK.

- Global Routing identifies approximate routing paths.

- Detailed Routing determines exact wiring with specific tracks, vias, and metal layers.
 
- The Sky130 PDK provides six metal layers, each with defined widths, spacings, pitches, and via rules to guide the router for legal and             manufacturable connections.

*Verification Before Sign-off*

Before the final design is ready for fabrication, several verification steps are required:

- DRC (Design Rule Check): Ensures the physical layout complies with all foundry-imposed rules like minimum spacing, width, enclosure, etc.

- LVS (Layout Versus Schematic): Compares the layout netlist against the schematic/netlist from the synthesis phase to confirm logical equivalence.

- Timing Analysis: Static Timing Analysis (STA) verifies that all setup and hold time constraints are satisfied across all paths and corners         (process, voltage, temperature).
The final Result is [GDSII file format.](https://anysilicon.com/semipedia/gdsii/)

RTL to GDSII flow : [Openlane](https://efabless.com/openlane)

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

#### Slack 

refers to the difference between the required arrival time and the actual arrival time of a signal. If the signal arrives earlier than required, it's called positive slack, indicating the design meets timing. If it arrives later, it's negative slack, meaning a timing violation has occurred. Positive slack is desirable, while negative slack must be fixed to ensure reliable circuit operation.

In Order to fix negetive slack we change the clock period to ```55.00``` in ```sky130_xxxxx_config.tcl``` 

```before```

<img width="848" height="255" alt="Screenshot 2025-07-24 123323" src="https://github.com/user-attachments/assets/9fe5c38e-a80a-4875-9cfc-4d0b81cf11c3" />


```After```

<img width="848" height="255" alt="Screenshot 2025-07-24 131231" src="https://github.com/user-attachments/assets/ae72bbe7-8a79-40b1-80b4-6af999df93fb" />

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
 run_synthesis
    
    ```

    <img width="848" height="921" alt="Screenshot 2025-07-28 223452" src="https://github.com/user-attachments/assets/354f77d9-484b-4fc9-82db-f98ba83e714f" />

    <img width="848" height="905" alt="Screenshot 2025-07-28 225050" src="https://github.com/user-attachments/assets/8fd2aa54-a14f-4a08-9c63-2e0da79a209a" />

    values area has increased and worst negative slack has become 0

- run floorplan

  <img width="1847" height="1200" alt="Screenshot 2025-07-28 225444" src="https://github.com/user-attachments/assets/9c38515d-dcd4-43cb-93d8-db4e18e8d09e" />

  we get error

  <img width="1847" height="1200" alt="Screenshot 2025-07-28 225444" src="https://github.com/user-attachments/assets/2e5e7292-a69f-4646-9e8a-3f35f27d626a" />

  run these commands instead :

  ```
  init_floorplan
  place_io
  tap_decap_or
  ```

- run placement and we obtain `DEF` file
  opening the file using magic by command --> `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read        ../../tmp/merged.lef def read picorv32a.placement.def &`

  <img width="1832" height="245" alt="Screenshot 2025-07-30 124909" src="https://github.com/user-attachments/assets/cd4713c1-b98b-4fcd-b906-30b1b344b2a8" />

  <img width="1438" height="740" alt="Screenshot 2025-07-30 125132" src="https://github.com/user-attachments/assets/db8a2104-c693-4c8c-ae57-b77b073c9139" />

  <img width="1441" height="738" alt="Screenshot 2025-07-29 010515" src="https://github.com/user-attachments/assets/3d0ac85a-1153-45a3-aff5-97221c0888b9" />

  <img width="1505" height="782" alt="Screenshot 2025-07-29 010553" src="https://github.com/user-attachments/assets/9461a51c-4038-4e8f-ad4a-088adf6a8967" />

- Post-Synthesis timing analysis

  Since we are having improved timing of 0 wn. so what we will do is we will do timing analysis on Synthesis that has lot of violations
  - we gradually improve and reduce our slack by replacing cells with better one and to deliver improved delay.
  This is **timing ECO fixes**

  First we do is running the synthesis --> include newly added lef to openlane flow --> set ::env(SYNTH_SIZING) 1 --> set                ::env(SYNTH_MAX_FANOUT) 4 --> run_synthesis

  <img width="1848" height="902" alt="Screenshot 2025-07-31 191742" src="https://github.com/user-attachments/assets/3d42043d-4758-4577-ae4b-c14846e44257" />
  

  Later in `cd Desktop/work/tools/openlane_working_dir/openlane` we write command `sta pre_sta.conf`

  *sta pre_sta.conf*
  <img width="1838" height="901" alt="Screenshot 2025-07-31 211105" src="https://github.com/user-attachments/assets/ee021675-c15d-46ad-b587-43b0a22ccf3f" />

  *my_base.sdc*
  <img width="1832" height="891" alt="Screenshot 2025-07-31 211155" src="https://github.com/user-attachments/assets/2c6e2506-2d8f-4049-ad3a-d95f4fafeec8" />

  <img width="1433" height="756" alt="Screenshot 2025-07-31 191954" src="https://github.com/user-attachments/assets/0893743c-bb30-450a-9620-e1d8b62fa90b" />

  <img width="1817" height="908" alt="Screenshot 2025-07-31 192011" src="https://github.com/user-attachments/assets/b17625a1-252e-4413-befd-2b22a3379632" />

  Replacing some cells to reduce slack

  <img width="1855" height="437" alt="Screenshot 2025-07-31 202908" src="https://github.com/user-attachments/assets/73c52399-6aae-4cce-bd09-1e93e5eadfda" />

  report_net -connections _11672_ [we see the driver pins] 
  <img width="1857" height="910" alt="Screenshot 2025-07-31 203237" src="https://github.com/user-attachments/assets/5f8240b5-57a3-4f99-84f6-3ab736b991f4" />

  replace_cell _14510_ sky130_fd_sc_hd__or3_4 [we replace some cells with better suited ones for driving 4 fanouts]
  <img width="1847" height="921" alt="Screenshot 2025-07-31 203516" src="https://github.com/user-attachments/assets/affa5a5a-3594-4d79-9f75-0cf3bf84f6e7" />

  To check the report and see the changes in slack we use command : report_checks -fields {net cap slew input_pins} -digits 4

  Slack reduced

  <img width="1841" height="905" alt="Screenshot 2025-07-31 203603" src="https://github.com/user-attachments/assets/38e44c1a-927a-47ec-8eec-e10d63772ddc" />

  report_checks -from _29052_ -to _30440_ -through _14510_
  <img width="1842" height="912" alt="Screenshot 2025-07-31 204342" src="https://github.com/user-attachments/assets/747b907d-20f5-44fa-9665-31cadfd64629" />

  <img width="1852" height="916" alt="Screenshot 2025-07-31 204533" src="https://github.com/user-attachments/assets/5e3f3cc8-b68e-4948-909c-57ace47adec3" />

  report_net -connections _11668_
  
  replace_cell _14506_ sky130_fd_sc_hd__or4_4

  <img width="1855" height="645" alt="Screenshot 2025-07-31 205421" src="https://github.com/user-attachments/assets/bb689f1d-315f-476a-8b97-14916f3b0c5e" />

  Slack Reduced

  <img width="1853" height="880" alt="Screenshot 2025-07-31 205436" src="https://github.com/user-attachments/assets/6610dbe3-72fb-4f44-ac34-efcfef1c67b7" />

  Intially : -23.90
  
  now : -22.9860

  <img width="397" height="667" alt="Screenshot 2025-07-31 212541" src="https://github.com/user-attachments/assets/e52a36e4-8a81-4ed7-bdac-31ecddd4e35e" />

  - We have reduced around 0.914 ns of violation
 
  - iterative process

- Run CTS
  
  - Here we proceed with earlier 0 violation design. we want to proceed with the clean design to further stage.
  - run synthesis,floorplan,placement and then run cts (`run_cts`)

  <img width="1297" height="457" alt="Screenshot 2025-07-29 181220" src="https://github.com/user-attachments/assets/1cb63874-68e5-4fba-a84f-a27dcec9eb0b" />

  <img width="1847" height="272" alt="Screenshot 2025-07-29 181247" src="https://github.com/user-attachments/assets/7573ef66-934f-45e7-8193-366e2c699ff4" />
  

- Post-CTS Timing analysis : OpenROAD

  command : `openroad`

  <img width="1257" height="480" alt="Screenshot 2025-07-29 181627" src="https://github.com/user-attachments/assets/4123c22c-b05e-457c-a5f0-804acaad7fc8" />

  <img width="1841" height="620" alt="Screenshot 2025-07-29 181808" src="https://github.com/user-attachments/assets/7d40a9ad-701b-4e1f-8449-5c060cdf403c" />

  <img width="1847" height="908" alt="Screenshot 2025-07-29 181832" src="https://github.com/user-attachments/assets/2f8ed163-c532-491e-906b-cad0c448fc34" />

  <img width="1830" height="878" alt="Screenshot 2025-07-29 181916" src="https://github.com/user-attachments/assets/1e754cbe-4c97-479c-bcce-e9beb3ce3154" />

  removing 'sky130_fd_sc_hd__clkbuf_1 and running CTS again 

  <img width="1852" height="907" alt="Screenshot 2025-07-29 182340" src="https://github.com/user-attachments/assets/e4d7ed6d-9bc7-4d7a-975e-4a574397192c" />

  <img width="1518" height="655" alt="Screenshot 2025-07-30 222049" src="https://github.com/user-attachments/assets/3a085a95-c2b1-493c-8b05-0ca7bbe19744" />
  
  echo $::env(CTS_CLK_BUFFER_LIST) [Checking current value of CTS_CLK_BUFFER_LIST]

  <img width="1518" height="655" alt="Screenshot 2025-07-30 222049" src="https://github.com/user-attachments/assets/3a085a95-c2b1-493c-8b05-0ca7bbe19744" />

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
  
  <img width="1848" height="916" alt="Screenshot 2025-07-29 183123" src="https://github.com/user-attachments/assets/d270f744-9aec-439a-b537-3e5fb5a04a9a" />

  <img width="1337" height="740" alt="Screenshot 2025-07-30 220638" src="https://github.com/user-attachments/assets/f89677be-3220-45dc-8868-1bde680a25e1" />

  Checking clock skew for setup and hold
  <img width="1852" height="945" alt="Screenshot 2025-07-29 183410" src="https://github.com/user-attachments/assets/5b3b741f-fc9e-40d4-9720-3ff3ae5cf279" />



# DAY 5 
# Final steps for RTL2GDS using tritonRoute and openSTA

## Lab 5 

- Performing (PDN) Power Distribution Network

  - In docker open openlane andd perform prepare the design and include newly added `lef` to openlane flow.
  - set SYNTH_STRATEGY --> DELAY 3
  - set SYNTH_SIZING --> 1
  - run synthesis, floorplan, placement and finally cts
  - after cts is done we do power distribution network PDN
  - gen_pdn

  <img width="1845" height="898" alt="Screenshot 2025-07-29 183947" src="https://github.com/user-attachments/assets/0623ac22-6ec7-4ea5-9456-c67c1de65eab" />

  <img width="1852" height="906" alt="Screenshot 2025-07-29 184133" src="https://github.com/user-attachments/assets/944c3ba9-5daf-4bd0-8a20-96b47da0d1c6" />

  <img width="1810" height="887" alt="Screenshot 2025-07-29 184343" src="https://github.com/user-attachments/assets/23dc0a50-d4e4-4b89-8e1e-8fab538c8f30" />

  <img width="1847" height="902" alt="Screenshot 2025-07-29 184540" src="https://github.com/user-attachments/assets/f948ace1-3fe0-488a-9a35-a88aca1b8b36" />

  <img width="1856" height="898" alt="Screenshot 2025-07-29 184602" src="https://github.com/user-attachments/assets/4d1d219b-4dc2-4011-9633-dfe76a9c24be" />

  - In `<date>/tmp/floorplan/` directory we  have the pdn
    to open we use command : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef      def read 14-pdn.def &`

  <img width="1606" height="818" alt="Screenshot 2025-07-29 184757" src="https://github.com/user-attachments/assets/d80cc966-ecb3-4adc-a923-1d4fe65a3cc7" />

  <img width="1502" height="838" alt="Screenshot 2025-07-29 184832" src="https://github.com/user-attachments/assets/8979c7cf-74b0-4249-b8ba-4a8595d78207" />

  <img width="1506" height="837" alt="Screenshot 2025-07-29 184913" src="https://github.com/user-attachments/assets/9ecab539-9937-4f37-92ee-a3ead193d6ac" />

- Perfrom detailed routing using TritonRoute

  - run routing using command --> `run_routing`
 
    <img width="1852" height="903" alt="Screenshot 2025-07-29 185022" src="https://github.com/user-attachments/assets/1c33b709-93b7-4a08-81d0-c7b58e537e81" />

    <img width="1853" height="505" alt="Screenshot 2025-07-30 175231" src="https://github.com/user-attachments/assets/1a0038ba-6100-4473-b3db-b90d596c6e27" />

    <img width="1852" height="918" alt="routing" src="https://github.com/user-attachments/assets/26d7286a-5e80-4040-8960-0c9006bc4a71" />
  - go to `<date>/results/routing/` directory
 
    load the def file : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def      read picorv32a.def &`

    <img width="1841" height="912" alt="Screenshot 2025-07-29 190229" src="https://github.com/user-attachments/assets/2e960fa2-3caf-4d39-99e6-bfb2747c5a95" />

    <img width="1748" height="890" alt="Screenshot 2025-07-29 190359" src="https://github.com/user-attachments/assets/d3cc2791-c395-4fb3-bd54-f2905c7f5c99" />

    <img width="1742" height="895" alt="Screenshot 2025-07-29 190416" src="https://github.com/user-attachments/assets/c8b8a5fd-7a7c-4a43-bdc7-a9378b4e07b9" />

    <img width="1688" height="880" alt="Screenshot 2025-07-29 190436" src="https://github.com/user-attachments/assets/db974f00-d023-45df-acfe-24e865549b76" />

    fast route guide inside `<date>/tmp/routing`

    <img width="1853" height="915" alt="Screenshot 2025-07-29 194603" src="https://github.com/user-attachments/assets/c2643529-ff21-4734-a34a-dbd7a7ec5990" />

- Post-Route parasitic extraction using SPEF extractor
  
  for the extracting, we use [SPEF_EXTRACTOR](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
  
  ```
  cd Desktop/work/tools/SPEF_EXTRACTOR
  
  python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/tmp/merged.lef      /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/results/routing/picorv32a.def
  ```
  
- Post-Route OpenSTA timing analysis

  <img width="1857" height="936" alt="Screenshot 2025-07-29 195516" src="https://github.com/user-attachments/assets/5d7777a5-7e44-4c99-a3c7-7ad856b9cfe1" />
  
  

**Picorv32a.def.png**


<img width="751" height="817" alt="Screenshot 2025-07-30 133612" src="https://github.com/user-attachments/assets/09ab689e-e161-4e35-a7cf-3023380250a4" />


**Picorv32a.gds.png**


<img width="1055" height="857" alt="Screenshot 2025-07-29 224044" src="https://github.com/user-attachments/assets/ace98a9a-4742-4409-8a97-660841e75a81" />



**GDS**

Stands for Graphic Design System. This is the file that is sent to the foundry and is called "tape-out".

In openLane use the command run_magic

The GDSII file is generated in the `results/magic` directory


**[DEF](https://teamvlsi.com/2020/08/def-file-in-vlsi-design-exchange.html)**

**[GDSII](https://en.wikipedia.org/wiki/GDSII)**


# Acknowledgements

[Kunal Ghosh](https://github.com/kunalg123) – Founder, VLSI System Design Corp.

[Nickson Jose](https://github.com/nickson-jose)– Developer & Contributor, Open Source Physical Design




     




    






  
  
  

  

  

    

 
    
    

    

    
 
    







  
