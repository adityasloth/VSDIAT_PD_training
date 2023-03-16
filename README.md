# VSDIAT_PD_training
VLSI Physical Design Training using OPENLANE. What I leared during this 5-day course is documented below.

# Open source EDA, openLANE and sky130A PDK

## About Openlane and the various EDA tools under its wing:
1. Learnt about the different tools available under OPENLANE and the role each of them plays in the design process starting from RTL all the way upto GDS II.  
From the below image, we can recount the different tools and their roles in detail below:
![image](https://user-images.githubusercontent.com/125293220/218512333-b4b4044a-9258-4f9d-80a0-0745b5a30230.png)

i) RTL Synthesis -  
a) Yosys takes care of the conversion of behavioral netlist to gate level format.  
b) Abc then annotates all gate cells in the netlist with the cell attributes such as delay using the Standard Cell Library.  
![image](https://user-images.githubusercontent.com/125293220/224394420-77ba2c83-f587-482f-90dd-ec2c39fb908a.png)

ii) STA - OpenSTA checks timing violations.  
iii) DFT - ATPG generation and testing of scan chains done using Fault. Optional process.  
iv) OpenROAD - Takes care of all steps from Floorplanning till Global Routing. To be studied in detail in further labs.  
v) IOPlacer - Placing the macro input and output ports.  
vi) RePlace - Performing global placement.  
vii) OpenDP - Perfroming detailed placement to legalize the globally placed components.
![image](https://user-images.githubusercontent.com/125293220/224396593-7aecd51d-2d9f-4c47-b964-4afbf912d837.png)
  
viii) TritonCTS - Synthesizes clock tree.  
ix) FastRoute - Performing global routing to generate a guide file for the detailed router.  
x) TritonRoute - Performing detailed routing.  
xi) Magic - Streaming out the final GDSII layout file from the routed def.  

## Labs

### Steps to invoke openlane:

- Before invoking openlane, we need to execute docker.  
> docker
  
- After this, we can get into the openlane interactive shell using the below command:  
> ./flow.tcl -interactive
- Then we need to load openlane package:  
> package require openlane 0.9

![image](https://user-images.githubusercontent.com/125293220/225643109-f738146f-f5df-4775-a6d2-3e270212820f.png)

- Now we load the design. The design we would be working on is picorv32a.  
> prep -design picorv32a

- This creates a directory under designs/picorv32a/runs/ based on the date on which it was run - the tag for each specific run.

### Synthesis:  

- This step involves taking the input netlist and converting it from behavioral to gate format (Yosis) and then annotating the standard cell library to all gates (Abc).
- To run it we simply execute:
> run_synthesis

in the openlane shell
- Once the run completes, we would be able to see:
![image](https://user-images.githubusercontent.com/125293220/225650560-21afe181-1894-4731-9d58-c9df662bce21.png)

- The output for this step ie. the synthesized netlist, would be saved in the following directory  
> /picorv32a/runs/<tag/results/synthesis/picorv32a.synthesis.v

### Floorplanning:

- In this step, we decide the die area and plan the amount of space that would be taken up by the various gates in our synthesized netlist. This requires a general idea of where the gates should be placed to ensure an optimal performance as well as to avoid any violations in the later stages 
- This step requires understanding of the below concepts:  
a) Core and Die: The part of the chip where the fundamental logic is placed is called the core. The die is a small semiconductor area on which the entire chip is to be fabricated. The die contains the core within it.  
b) Utilization Factor: The ratio of the area of the core used by the cells to the total core area is called the Uilization factor. It is usualy kept in the range of 0.5-0.7. A good utilization factor ensures efficient placement and routing.  
c) Aspect Ratio : The height of the core area divided by the width of the core area.  
d) Preplaced Cells : Some cells like Memory Cells, Clock-gating Cells etc. are desiged only once. These are then used repeatedly in the design. These IPs are placed before placement and routing and are thus called preplaced cells. These cells, once placed, can never be removed.  
e) Decoupling Caps : Capacitors placed close to preplaced cells to ensure smooth transfer of logic between them. These capacitors will charge up to the power supply voltage over time and behave like the power supply which faces drop due to the interconnect wires.  

## Labs (Continued...)

### Floorplan
- To run floorplan, we simply run the below command:
> run_floorplan
 
in the openlane console  

- With the successful run, a def is generated in the results directory. Let us review this def using magic.
- To invoke magic, we need to specify the lef file, def file as well as the technology file. Magic can be invoked using the below command:
> magic -T \<path to tech file\> lef read \<path to lef file\> def read \<path to def file\>
- This opens the magic gui with the input info loaded. It looks as follows:
![image](https://user-images.githubusercontent.com/125293220/225667980-1b002f70-e26f-443c-93ea-0fe8b7a25646.png)

- In the console window, we can execute various magic specific commands. For e.g. we can select any object by hovering our mouse over it and pressing the 'S' key. Once done, we can use the 'what' command to know about the selected object:
![image](https://user-images.githubusercontent.com/125293220/225668670-36cec6c8-056c-4d4c-b0eb-7c8230481352.png)

- We can zoom into different regions using the Z key. We can see the pad cells placed based on the config settings we have specified: 
![image](https://user-images.githubusercontent.com/125293220/225669117-83685c61-cfe5-4477-9d03-588a230f58e7.png)

- We can also observe that all our core cells are sitting in the corner. This is because we are yet to run the placement step:
![image](https://user-images.githubusercontent.com/125293220/225669468-939c6216-6484-4df0-9c35-a762d8bd273f.png)

### Placement
- To run placement, we use the below command in shell:
> run_placement

- This is done in two steps:  
a) Global placement: It ensures that the cells are optimally placed in roughly the correct locations so that minimum wire lengths would be needed to finally establish connections at the routing stage. Cells may overlap in this stage.  
b) Detailed placement: Places the cells properly in well defined rows and columns. Cells will not overlap at the stage.  

- Running placement:
![image](https://user-images.githubusercontent.com/125293220/225674352-9400ae28-91a8-4791-982e-c501d575a22b.png)

- Once run is successful we get a new def with the core cells placed. Let us review this def using magic:
![image](https://user-images.githubusercontent.com/125293220/225674609-c07446d8-6a54-4c57-aa3c-09a8d0deb513.png)


## Custom inverter design using Magic layout and ngspice characterization

### CMOS inverter
CMOS cells have five modes of operation:  

- NMOS Cutoff PMOS Linear
- NMOS Saturation PMOS Linear
- NMOS Saturation PMOS Saturation
- NMOS Linear PMOS Saturation
- NMOS Linear PMOS Cutoff
Thershold voltage is the voltage at which Vin = Vout. Threshold voltage is a function of the W/L ratio of a device, therefore varying the W/L ratio will vary the output waveform of CMOS devices and inturn the transfer charecteristic of the device as well. A perfectly symmetrical device will have a switching threshold such that Vin = Vout = VDD/2 which is achieved when (W/L)p is approximatly 2.5 times (W/L)n.  

### LAB
- Clone the repo: https://github.com/nickson-jose/vsdstdcelldesign.git, and use the mag file to see the layout of the inv cell.
> magic -T sky130A.tech sky130_inv.mag
- 

## D2_SK3:


