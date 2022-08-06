# SKY130_PD_WORKSHOP
 Advanced Physical Design using openlane/sky130
# AGENDA
  Day1 - Inception of open-source EDA, OpenLane and Sky130 PDK
  
         - How to talk to computers
         - SOC design and OpenLane
         - Starting RISC-V SOC Reference design
         - Get familiar to open-source EDA tools
  Day2 - Undestand importance of good floorplan vs Bad floorplan and introduction to library cells
  
         - Chip floorplanning considerations
         - Library Binding and Placement
         - Cell design and characterization flows
         - General timing characterization parameters
  Day 3 - Design and characterize one library cell using Magic Layout tool and ngspice
  
         - Labs for CMOS inverter ngspice simulations
         - Inception of Layout – CMOS fabrication process
         - Sky130 Tech File Labs
Day 4 - Pre-layout timing analysis and importance of good clock tree

         - Timing modelling using delay tables
         - Timing analysis with ideal clocks using openSTA
         - Clock tree synthesis TritonCTS and signal integrity
         - Timing analysis with real clocks using openSTA
Day 5 - Final steps for RTL2GDS

         - Routing and design rule check (DRC)
         - PNR interactive flow tutorial
# Day1 - Inception of open-source EDA, OpenLane and Sky130 PDK
# How to talk to computers
1 below is a block diagram of a complete SOC chip which is QFN-48 (quad flat no leads package) having the RISCV SOC and foundary IP's

         - during floorplan foundary IP's are mainly placed and rest other logic is placed by the PNR tool
![image](https://user-images.githubusercontent.com/110658068/183241039-41952a8e-0354-4ac5-82ab-b15678694380.png)

2 Below image shows the RTL to GDS mapping for the RISCV SOC

![image](https://user-images.githubusercontent.com/110658068/183241238-3fba36b2-a67c-4b2f-a6e9-f7d3e7cce943.png)
# RTL2GDS flow steps

![image](https://user-images.githubusercontent.com/110658068/183242250-787dec77-1b0b-4f8f-b4cc-506d013b965e.png)

    -Synthesis :- In logic synthesis, a high-level description of the design (RTL Code) is converted into an optimized gate-level representation of a given standard cell library and certain design constraints.
    
    -Floorplanning :- determines the shapes and arrangement of sub-circuits or modules, as well as the locations of external ports and IP or macro-blocks.
   
    -Power and ground routing (power planning) :- often intrinsic to floorplanning, distributes power (VDD) and ground (GND) nets throughout the chip.
    
    -Placement :-finds the spatial locations of all cells within each block and place the all cells in the design.
    
    -Clock Tree Synthesis(CTS) :- determines the buffering, gating (e.g., for power management) and routing of the clock signal to meet prescribed skew and delay requirements.
    
    -Global routing :- allocates routing resources that are used for connections; example resources include routing tracks in the channel and in the switch box.
    
    -Detailed routing :- assigns routes to specific metal layers and routing tracks within the global routing resources.
    
    -sign-off :- If there are some timing violations in post route design, we have a further stage called ECO (Engineering Change Order) where we can fix the timing violations. Apart from timing violation, there may be issues like IR Drop, DRC Violations all these are fixed in this stage and a final layout file   free from all the violation is streamed out in GDSII format. This process is known as tapeout in ASIC flow.
# whole OpenLane ASIC flow 

![image](https://user-images.githubusercontent.com/110658068/183242851-fa624d02-8d4a-41ee-8ce4-ebf264d4df9d.png)

# Getting started with OpenLane flow and setup to run the whole flow 

1) initial Window once the LINUX terminal is opened

![image](https://user-images.githubusercontent.com/110658068/183243055-7d6e73e9-dc99-42b8-b2df-679e842dab75.png)

2) All the PDK's data stored in the directory ~/Desktop/work/tools/openlane_working_dir/pdks
        - open_pdks
        - sky130A
        - skywater-pdk
3) All the runs need to be fired in the directory ~/Desktop/work/tools/openlane_working_dir/openlane

![image](https://user-images.githubusercontent.com/110658068/183243261-e497d5d7-23a1-47ff-a33d-8dd6d0960b0d.png)

4) steps to evoke the tool
     1) go to the directory ~/Desktop/work/tools/openlane_working_dir/openlane
     2) type command docker
     3) type ./flow.tcl -interactive
     4) type package require opnelane 0.9 (to import all the packages require to run the flow)

![image](https://user-images.githubusercontent.com/110658068/183243553-6a86610e-4232-4047-b354-88ac5cf31561.png)

5) the design on which we will work is picorv32a and all the settings or configurations file related to the design stored in
      1) ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a
      2) there are two configuration files present under the above directory config.tcl and sky130A_sky130_fd_sc_hd_config.tcl which 
      have all the settings required to create the desired design
      
      ![image](https://user-images.githubusercontent.com/110658068/183243740-d19d593d-e49b-4d3d-807b-8cb8518ae96c.png)

6) Preparing the system to run all the steps from synthesis to routing by using
     1) prep -design picorv32a -tag <tag_name>
     2) this step will fetch all the files required to run the whole RTL2GDS flow
     
     ![image](https://user-images.githubusercontent.com/110658068/183243941-9d950ffa-343f-4198-a30b-71dbbfca6b64.png)
  
 7) After successful preparation we can go for first step in the design which is synthesis
     1) type command run_syntheis
     
     ![image](https://user-images.githubusercontent.com/110658068/183244358-898a36b3-78aa-44e3-9336-e7cc8cafaded.png)

     2) during the sythesis process tool will create the gate level netlist which we can further use in other steps of design
     3) netlist is present in the directory 
        ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/run_08_06_2022/results/synthesis/picorv32a.synthesis.v
     4) reports for the synthesis present in the directory 
        ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/run_08_06_2022/reports/synthesis/2-opensta.timing.rpt
        
        ![image](https://user-images.githubusercontent.com/110658068/183244297-331211fc-5b97-495a-837f-85a31f93cfc6.png)

8) flop ratio can be calculated after the synthesis by (total number of flops)/(total number of cells)
 
# Day2 - Undestand importance of good floorplan vs Bad floorplan and introduction to library cells

mostly focusing on the floorplan constraints, concept of decoupling cells, power planning and finally performing the floorplan and reviewing the floorplan in magic 

1)For defining width and height of CORE and DIE

      a.	Dimensions of standard cells is crucial part, and the number of standard cells present in the design 
      
      b.	So, these values can be pre-estimated based on the amount of logic present in the design 
      
      c.	Two terminologies are used for dimension of the design utilization factor and aspect ratio
      
      d.	Utilization_factor = (Area occupied by netlist)/(Total area of the core)
      
      e.	Aspect_ratio = (height of die/width of die)

![image](https://user-images.githubusercontent.com/110658068/183250804-a6f7969e-452b-4bc9-8137-720f276757f9.png)

2)	Preplaced cells: - these are the cells which are manually placed by the designer during the floor planning, after the placement of these cells these cells are not touched in later stage of the design

![image](https://user-images.githubusercontent.com/110658068/183250851-46518b44-432f-4394-b7bf-61041de08402.png)

3)	Need for decoupling capacitance:- A decoupling capacitor is a capacitor, which is used decouple the critical cells from main power supply, in order to protect the cells from the disturbance occurring in the power distribution lines and source

![image](https://user-images.githubusercontent.com/110658068/183250867-60326015-41fe-43e2-8d3d-9ade3cb2b623.png)

4)	Power planning: - Power planning means to provide power to every macros, standard cells, and all other cells are present in the design

![image](https://user-images.githubusercontent.com/110658068/183250881-b753cfdb-38a1-4fc5-b2ca-94f5a20aedb9.png)

# Running the actual floorplan in the tool

1)	By using below command floorplan can be created
       a.	Type run_floorplan
       
       ![image](https://user-images.githubusercontent.com/110658068/183250930-09eb6db8-1675-4519-afa8-1edec527051e.png)

2)	Floorplan will be created after this step and can be seen in magic using command

![image](https://user-images.githubusercontent.com/110658068/183250969-fa0fb131-547d-42d2-8b1c-d63d8522f6ad.png)

![image](https://user-images.githubusercontent.com/110658068/183250984-3ed05f96-9e3e-4b2b-afea-18a83881d853.png)

![image](https://user-images.githubusercontent.com/110658068/183250998-c913d22b-20d2-4d1c-915f-1f83dcf43aa4.png)

# Running the actual placement of the design

1)	By using the below command final placement of the standard cells done in the design

![image](https://user-images.githubusercontent.com/110658068/183251031-3bee8099-c83d-467f-9e44-4f0ede211d3c.png)

2)	Placement is done after this step and placed design can be view by magic

![image](https://user-images.githubusercontent.com/110658068/183251046-00bd4972-7bf3-46ba-89d2-7f4890f1194a.png)



