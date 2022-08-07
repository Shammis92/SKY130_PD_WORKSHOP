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


# Day3 - Design Library cell using Magic layout and ngspice characterization

1)	Creation of SPICE deck for CMOS inverter
        
        a.	Component connectivity
        
        b.	Component values
        
        c.	Identify nodes
        
        d.	Name nodes

![image](https://user-images.githubusercontent.com/110658068/183262345-407f5126-999d-4f03-9784-22dabf7a501f.png)

2)	Command to download the data for already created inverter

![image](https://user-images.githubusercontent.com/110658068/183262372-4771ac42-c3c1-4f9a-b15e-5f7f27d22aeb.png)

3)	Opening the inverter in the magic 
            
            a.	magic -T sky130A.tech sky130_inv.mag &

![image](https://user-images.githubusercontent.com/110658068/183262398-811665f7-6f36-4693-92fb-bd2743ed2982.png)

4)	extracting the inverter from magic
          
          a.	extract all
          
          b.	ext2spice cthresh 0 rthresh 0
          
          c.	ext2spice

![image](https://user-images.githubusercontent.com/110658068/183262424-786711ab-1ae4-45b8-9e81-224bf30e39af.png)

          d.	generated files are sky130_inv.ext and sky130_inv.spice

![image](https://user-images.githubusercontent.com/110658068/183262450-17ea43bd-921d-40af-977d-e6601354e3c5.png)

5)	Corrections and addition in the .spice file
           
           a.	Scale value changed from 1000u to 0.01u as per the magic data

![image](https://user-images.githubusercontent.com/110658068/183262476-19c6fba7-8494-4071-ba42-11b53628fb6f.png)

           b.	Include NMOS and PMOS libs
          
           c.	Creating supply voltages
           
           d.	Adding the type of analysis
           
           e.	Change the name of PMOS and NMOS devices as per the .lib files

![image](https://user-images.githubusercontent.com/110658068/183262501-1cbd1159-5380-46c2-8ffb-8b56d32ce97b.png)

6)	Running the simulation with the help of ngspice 
           
           a.	ngspice sky130_inv.spice

![image](https://user-images.githubusercontent.com/110658068/183262538-ccce3ffb-df64-4e9b-b097-39779e95053a.png)

           b.	plot the result  -plot y vs time a

![image](https://user-images.githubusercontent.com/110658068/183262560-18812fd0-79c5-413c-ad6d-49e304b5352c.png)

# Day4 - Pre-layout timing analysis and importance of good clock tree

1)	Extract LEF file from full design of inverter
 
              a.	Track information present at ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info

![image](https://user-images.githubusercontent.com/110658068/183288238-70b51289-2517-4aa0-9054-b2ca1f55858a.png)


             b.	Converting grid definition according to track definition
             
                   i.	grid help (tkcon window)
                   
                   ii.	grid 0.46um 0.34um 0.23um 0.17um (tkcon window) 


![image](https://user-images.githubusercontent.com/110658068/183288255-23896af2-a8f5-43f0-bb80-84fb304a784b.png)

            c.	width of standard cell should be odd multiple of x-pitch of connecting layer

![image](https://user-images.githubusercontent.com/110658068/183288275-73f7ccb6-47d6-4711-bb97-0a47d70b2a35.png)

            d.	height of standard cell should be even multiple of y-pitch of connecting layer
            
![image](https://user-images.githubusercontent.com/110658068/183288291-380e9f8d-3da8-4997-8be2-c666bbc3b900.png)

            e.	create port definition
            f.	set port class and port use attribute
            g.	write out the lef from magic using  lef write

![image](https://user-images.githubusercontent.com/110658068/183288312-12865a0e-f649-42e0-a5dc-9d3b7913e916.png)

            h.	content of lef file of inverter

![image](https://user-images.githubusercontent.com/110658068/183288335-6a47af39-451e-4af2-a509-e2857ff729ab.png)

2)	Include our inverter to the openlane flow

            a.	Copy the lef file created for inverter to ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

            b.	Copy all .lib file from ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/ to ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

            c.	Modify config.tcl at ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/config.tcl
            
![image](https://user-images.githubusercontent.com/110658068/183288358-fa5a8614-ccd0-41f2-9d0d-cae6eca89730.png)

            d.	Opening the openlane and adding the newly created lef file
            
![image](https://user-images.githubusercontent.com/110658068/183288374-a5c2e48b-c394-42ac-8cf9-b5523a4b0d8c.png)

            e.	run_synthesis right after the addition of newly created inverter cell and checking whether it is included during the synthesis or not

![image](https://user-images.githubusercontent.com/110658068/183288389-0fd0f1d7-4357-443d-9a2e-5e966bdd5d8f.png)

            f.	after this run there are timing violations present which needs to be fixed 
            
            i.	changed below parameters on the fly to reduce the timing violations
            
 ![image](https://user-images.githubusercontent.com/110658068/183288445-c2c12ea2-baab-42f4-83c2-c2d47f6dc969.png)
           
            g.	run_floorplan failed due to some file issue
            
 ![image](https://user-images.githubusercontent.com/110658068/183288462-ad899d5d-f3c2-4dc4-86cf-b13d5f296361.png)
 
            h.	ran below commands manually to create the design
                   
                   i.	init_floorplan

                  ii.	place_io
                 
                 iii.	global_placement_or
                  
                  iv.	detailed_placement
                   
                   v.	tap_decap_or
                  
                  vi.	detailed_placement
                 
                 vii.	gen_pdn
                
                viii.	run_routing
                
             i.	locating the inverter cell in the design with the help of magic               

![image](https://user-images.githubusercontent.com/110658068/183288511-f6a2fb48-1c47-42b6-9594-95eabc2719c2.png)

![image](https://user-images.githubusercontent.com/110658068/183288519-33915800-8f94-4597-bdb9-212cfb66eab9.png)

3)	running CTS with the default settings(tool – TritonCTS)

             a.	run_cts

![image](https://user-images.githubusercontent.com/110658068/183288538-97aaf1fa-589c-47d7-956d-7ab97c88e871.png)

# Day5 - inludes the final routing stage

1) routing the of design is done with the help of command run_routing

![image](https://user-images.githubusercontent.com/110658068/183288647-41b3181b-d923-4e36-96d6-a179e56c5109.png)

2)	final routes design in magic

![image](https://user-images.githubusercontent.com/110658068/183288839-3bcdf0ad-8e23-4cc1-bfa7-54f8d89ce749.png)

![image](https://user-images.githubusercontent.com/110658068/183288846-f60ccf1b-1eb9-4bc2-91f5-9c7a8eacd402.png)


           
       
