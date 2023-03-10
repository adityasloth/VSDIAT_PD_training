# VSDIAT_PD_training
VLSI Physical Design Training using OPENLANE. What I leared during this 5-day course is documented below.

# Day1: Open source EDA, openLANE and sky130A PDK

## D1_SK2:
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

## D2_SK3:


