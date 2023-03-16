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

# Floorplanning: Good vs Bad floorplanning and intro to library cells

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

### Cell characterization flow:

GUNA software is used for cell characterization. An example buffer cell is characterized in the following steps:
- Device model (mos devices and their electrical values)
- Read the extracted spice netlist(inverters called and used to make buffer)
- Recognize or define the behavior of the buffer
- Read subckt of inverter
- Attach/read in necessary power supplies
- Apply stimulus at input
- Provide necessary output load caps
- Give the required simulation (.tran)
Finally, we feed all above steps from in GUNA software as config file.


### LAB
- Clone the repo: https://github.com/nickson-jose/vsdstdcelldesign.git, and use the mag file to see the layout of the inv cell.
> magic -T sky130A.tech sky130_inv.mag
- The layout looks like this:
![image](https://user-images.githubusercontent.com/125293220/225680284-805c7b73-c256-46bc-a672-ae148e8964c5.png)

- Now we extract the spice netlist from the mag layout.
> extract all
> ext2spice cthresh 0 rthresh 0
> ext2spice

- The first command creates a .ext file which is then converted to a spice netlist using the below two commands. 
- We then edit this file to ensure that the proper library is annotated, the sources are applied and the type of simulation is specified:
![image](https://user-images.githubusercontent.com/125293220/225682705-337aee07-9e3a-4f66-9eff-1cb34bbec422.png)

- Now we can run the simulation using below command:
> ngspice sky130_vsdinv.spice
- This opens the ngspice shell as well.
- We can plot the output/input etc by plotting the voltage at various nodes:
![image](https://user-images.githubusercontent.com/125293220/225684068-5a5ed4a3-6a6f-48fa-b1cc-a89c0ab70255.png)

- Using these waveforms, we can find out the various parameters such as:
a) Rise transition time:
![image](https://user-images.githubusercontent.com/125293220/225684267-8d600380-c9fa-4047-97ce-b916da4e32a0.png)

b) Fall delay:
![image](https://user-images.githubusercontent.com/125293220/225684352-bdbe8591-af76-452d-b8f1-c596d037c2fc.png)

c) By clicking on the point where we want to measure the values, we can print the values at specific points. These get printed in the ngspice shell:
![image](https://user-images.githubusercontent.com/125293220/225684698-54d23adc-be5c-4cfc-91ca-0c6e097b53e3.png)



## Pre-layout timing analysis and CTS

### Magic to std cell lef generation:
PnR is possible just by giving information about the pin placement and metal information, there is no need of providing any information about the logic. This is done by the LEF file (Library Exchange Format) to perform interconnect routing in conjunction to routing guides generated from the PnR flow. This is how the companies do not disclose the logic information to the foundry.  

Before generating the LEF file for our standard cell design we need to ensure that the design we have made is satisfing the foundry requirment i.e. track details. This we can confirm by making a grid in magic with the proper details of the tracks from track.info file as shown below.  

- The track.info file looks like below:
![image](https://user-images.githubusercontent.com/125293220/225688309-45bb0799-b3ea-4300-8b46-71ddcc3b3726.png)

- The format for it is as follows:
> \<layer-name\> \<X/Y direction\> \<track-offset\> \<track-pitch\>

- Following rules should be taken care of when creating a std cell:  
1) The input/output pins should lie on the intersection of the horizontal and vertical track. (so that the route can reach them from the y as well as x direction)  
2) Width of the std cell should be an odd multiple of the track pitch and height should be odd multiple of track's vertical pitch.

- After this we set the direction (power, ground or signal) for all the pins usting the port command. 
![image](https://user-images.githubusercontent.com/125293220/225693866-cab26dab-f9bd-4216-9fe8-a1d035f90418.png)

- Once done, we can write out the lef file for this cell using the command:
> lef write sky130_vsdinv.lef

### Including this cell in our previous design
Now we need to include this in our previously placed design. So we go back to the synthesis stage.

- Before synthesis, we need to add the new lef file in the config.tcl by adding the below line:
```
set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```
- We also have to set the library paths to include standard cell information
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
```
After modifying the config file run synthesis, and just before 'run_synthesis' step, we add the following commands in the console.
```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```
- Now we run synthesis again:
![image](https://user-images.githubusercontent.com/125293220/225696423-9995894e-eb76-429b-8fcd-46d424fb1295.png)

We observe that the slack values are negative. Negative slack means that there are violations.

### OpenSTA run to check violations

OpenSTA can be invoked as a standalone application outside the openlane shell to get additional info on the violations and get an idea on how to fix them:  
The setup for this is done using the following steps:  
- Create the my_base.sdc file from refering to https://github.com/nickson-jose/vsdstdcelldesign/tree/master/extras
- Match the clock period with the clock period set in openlane_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib (highest priority config).
- Create pre_sta.conf file that contains paths to the verilog file, constraints, clcok period and other required parameters.
![image](https://user-images.githubusercontent.com/125293220/225697721-39d03945-3683-42b4-9d5a-36e466b7661c.png)

Now we can run OpenSTA using below command:
> sta pre_sta.conf

- This shows the detailed report along with the slack values:
![image](https://user-images.githubusercontent.com/125293220/225698863-256b20b6-67b6-4fc8-b64f-c2de49c3a197.png)

- We can try to reduce the slack values by changing the fanout and/or the synthesis startegy to be more concerned with DELAY instead of area. This can be done by playing with the two keywords:  
a) SYNTH_STRATEGY, &
b) SYNTH_MAX_FANOUT  

- Trying with SYNTH_STRATEGY as "DELAY 3" and SYNTH_MAX_FANOUT as 4.
  - Synthesis:
![image](https://user-images.githubusercontent.com/125293220/225703422-d0ac5768-683f-4ff1-847c-2ea293380222.png)

  - OpenSTA:
![image](https://user-images.githubusercontent.com/125293220/225703484-5b208f77-7c73-44da-b36f-2e2db25b2343.png)

We can see slack values are reduced.

- We can also think of upsizing buffers/other std cells to increase their drive strength thereby reducing slew values. This can be done by refering to the slack report generated in openSTA. This would however, increase the design area so it is a trade-off.

### Clock Tree Synthesis

This step ensures that the clocks are routed to all sequential cells such that there is equal distribution of the delay i.e. the clock skew is minimal.

Process to run cts is simple. Just run below command in openlane shell:
> run_cts

![image](https://user-images.githubusercontent.com/125293220/225707471-4db3a41f-2bba-450e-ab7a-d0e1a16a7d3d.png)

### Post CTS STA analysis

Following steps need to be run in openlane to check STA after CTS step. These involve invoking openROAD internally within the openLANE gui.
> openroad  
> read_lef /openLANE_flow/designs/picorv32a/runs/16-03_10-19/tmp/merged.lef  
> read_def /openLANE_flow/designs/picorv32a/runs/16-03_10-19/results/cts/picorv32a.cts.def  
> write_db pico_cts.db  
> read_db pico_cts.db  
> read_verilog /openLANE_flow/designs/picorv32a/runs/16-03_10-19/results/synthesis/picorv32a.synthesis_cts.v  
> read_liberty -max $::env(LIB_SLOWEST)  
> read_liberty -min $::env(LIB_FASTEST)  
> read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc  
> set_propagated_clock [all_clocks]  
> report_checks -path_delay min_max -format full_clock_expanded -digits 4  

### Power Delivery Network Generation

Now that we are done we CTS, we start the routing stage by generating out power delivery network as per the pitch and width values from the track.info file.  
To run it simply run below command in openlane shell:
> gen_pdf

