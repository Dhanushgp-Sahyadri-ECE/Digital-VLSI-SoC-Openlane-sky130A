Physical Design using OpenLANE & Sky130 PDK
📘 Project Introduction
This repository showcases the complete physical design flow using the OpenLANE toolchain with the Sky130 open-source PDK. Executed as part of Digital VLSI SoC Design and Planning Workshop by VLSI System Design Corporation, the project offers a hands-on experience in chip design, starting from RTL to the final GDSII.

OpenLANE is a powerful, open-source RTL-to-GDSII automation framework that integrates tools like Yosys, OpenROAD, Magic, Netgen, Fault, OpenPhySyn, and more — all working cohesively to produce clean, manufacturable layouts without manual intervention. The flow is optimized specifically for the Skywater 130nm PDK, enabling the development of hard macros and full chips efficiently.

📁 Repository Structure
DAY 1

Theory
Introduction to RISC-V
Simplified RTL to GDSII Flow
OpenLane Flow
Lab
Synthesis
Estimation of Flip Flop Ratio
Slack
DAY 2

Theory
Floorplan
Placement
Lab
Floorplan
Placement
Characterization
Estimation of area of the die
Day 3

Theory
Designing a Library Cell
steps for simulation in ngspice
SPICE Switching Threshold and Propagation Delay
16-Mask CMOS Process
Lab
Spice extraction of inverter in magic
Post-Layout Spice simulation (ngspice)
Slew rate and Propagation delay
Fix Tech File DRC via Magic
DAY 4

Theory
Delay Table
Timing Analysis (using Ideal Clocks)
Clock Tree Synthesis Stage
Lab
DAY 5

Lab
DAY-1
Inception-of-Open-source-EDA,OpenLane-and-Sky130-PDK
we explore the fundamental physical elements of an integrated circuit (IC) :

Package, Chip, Pads, Core, Die, and IPs :

The QFN-48 (Quad Flat No-lead) package is a surface-mounted IC package with 48 pins used to connect the chip to the outside world.

Inside the package is the die, the actual silicon chip where transistors are fabricated.

The core is the functional part of the die where the logic (e.g., processor, memory) is implemented.

Surrounding the core are pads, which act as electrical contact points for interfacing signals from the core to the pins on the package.

The chip as a whole includes the die, package, and connections.

The core of the chip will contain two types of blocks:

Foundry IP Blocks (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.

Macro blocks (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts.

182751377-2810d388-21b0-4df1-b1d4-c72176d80d28
Introduction to RISC-V
RISC-V stands for Reduced Instruction Set Computing – Version 5 and is designed with simplicity and modularity in mind.

Unlike proprietary ISAs like ARM or x86, RISC-V is open, meaning anyone can implement or modify it freely.

The architecture consists of a base integer set and optional extensions (e.g., for multiplication, atomic operations, floating point).

RISC-V is ideal for education, research, and industrial design due to its transparency and scalability.

Simplified RTL to GDSII Flow
Synthesis

The Register Transfer Level (RTL) code, typically written in Verilog or VHDL, is translated into a gate-level netlist. This netlist is composed of logic gates and components from a pre-defined standard cell library. These cells have fixed sizes and electrical characteristics, provided by the Process Design Kit (PDK). During synthesis, optimizations such as constant propagation, logic simplification, and technology mapping are performed to ensure an area-efficient and logically equivalent design.

Floor Planning and Power Planning

This step involves defining the physical layout of the chip, including the die area, core area, margins, and reserved regions for IP blocks or macros. Floorplanning also includes decisions on I/O pin placement and defining power domains. In power planning, a Power Distribution Network (PDN) is created to deliver clean power across the chip. The power rails and straps are usually placed on upper metal layers since they are thicker and have lower resistance, which helps reduce IR drop and ensures reliable power delivery.

Placement

Placement determines the exact physical location of standard cells within the defined floorplan. This process occurs in two phases:

Global Placement: Estimates optimal positions to reduce wirelength and congestion, but may not obey all design rules.

Detailed Placement: Adjusts the global result to ensure legal placement while minimizing additional wirelength or timing penalties.

Clock Tree Synthesis (CTS)

A clock tree is built to distribute the clock signal to all sequential elements (flip-flops) in the design. Since all flip-flops must receive the clock signal simultaneously to avoid timing violations like skew and jitter, structures such as H-trees or X-trees are used. CTS also buffers the clock signal and ensures balanced timing paths across different regions.

Routing

Routing connects the logically associated pins (nets) across the placed cells using horizontal and vertical metal layers defined in the PDK.

Global Routing identifies approximate routing paths.

Detailed Routing determines exact wiring with specific tracks, vias, and metal layers.

The Sky130 PDK provides six metal layers, each with defined widths, spacings, pitches, and via rules to guide the router for legal and manufacturable connections.

Verification Before Sign-off

Before the final design is ready for fabrication, several verification steps are required:

DRC (Design Rule Check): Ensures the physical layout complies with all foundry-imposed rules like minimum spacing, width, enclosure, etc.

LVS (Layout Versus Schematic): Compares the layout netlist against the schematic/netlist from the synthesis phase to confirm logical equivalence.

Timing Analysis: Static Timing Analysis (STA) verifies that all setup and hold time constraints are satisfied across all paths and corners (process, voltage, temperature). The final Result is GDSII file format.

RTL to GDSII flow : Openlane

OpenLane Flow
182759711-6b9352ec-7652-4589-af31-53a409eb2830
OpenLane is an automated RTL to GDSII flow that utilizes various components such as OpenROAD, Yosys, Magic, Netgen, and custom scripts for design exploration and optimization.

The flow performs all ASIC implementation steps from RTL down to GDSII. OpenLane Interactive Mode Commands include a variety of functions such as setting the current netlist, running logic verification, setting the current def file, preparing lef files, preparing a liberty file, generating an exclude list file, sourcing configurations, specifying a path to save config_in.tcl, preparing a run, specifying a design folder, overwriting an existing run, specifying a path to save the run, specifying a name for a specific run, creating a tcl configuration file for a design, setting the verilog source code file, specifying the design's configuration file, setting a verbose output level, generating a padframe, saving views of a given run, changing save paths for various file types, labeling pins of a macro def, generating a verilog netlist from a def file, creating an obstruction, setting tracks on a layer, extracting core dimensions, running SPEF extraction and Static Timing Analysis, running antenna checks, saving environment variables, running OpenSTA timing analysis, checking for unmapped cells, checking for assign statements, and checking if the LEF was properly read.

Antenna Rules Violation : long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to either use bridging or antenna diode insertion to leak away the charges.
Inside a specific design folder contains a config.tcl which overrides the default settings on OpenLANE.
