# OpenFasoc   
Fully-Autonomous SoC Synthesis using Customizable Cell-Based Synthesizable Analog Circuits.
The SoC synthesis tool realizes analog circuits, including PLLs, power management, ADCs, and sensor interfaces by recasting them as structures composed largely of digital components while maintaining analog performance. 
OpenFASOC is focused on open-source automate analog generation from user specification to GDSII with fully open-sourced tools. This project is led by a team of researchers at the University of Michigan is inspired from FASoC whcih sits on proprietary tools.
See more about FaSoC at https://fasoc.engin.umich.edu/    also at https://openfasoc.readthedocs.io/en/latest/index.html   

# Installation   

## Openfasoc   
The command used to install OpenFASOC are    
```
git clone https://github.com/idea-fasoc/openfasoc

cd openfasoc

pip install -r requirements.txt
```   
## Prerequisites   

 ### 1.OpenROAD   
The commands to install OpenROAD are,
```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git

cd OpenROAD

./etc/DependencyInstaller.sh

./etc/DependencyInstaller.sh -run

./etc/DependencyInstaller.sh -dev

mkdir build

cd build

sudo cmake ..

sudo make
```    
 ### 2. Open_PDKs   
 
 we need sky130A files   
 
 download from http://opencircuitdesign.com/open_pdks/archive/open_pdks-1.0.354.tgz    
 
 or git clone https://github.com/RTimothyEdwards/open_pdks    
 
 or git clone git://opencircuitdesign.com/open_pdks   
 ```   
 ./configure --enable-sky130-pdk
sudo make
sudo make install   
```   
you may need some prerequisites for this clone them from opencircuitdesign.com/open_pdks     

 ### 3. Klayout
 we need laytest version.
Downlaod the latest version of the Klayout from [here](https://www.klayout.de/build.html). Install the following dependencies: qt5-default, qttools5-dev, libqt5xmlpatterns5-dev, qtmultimedia5-dev, libqt5multimediawidgets5 and libqt5svg5-dev.   
```
sudo apt-get install -y libqt5widgets5

sudo dpkg -i klayout_0.27.11-1_amd64.deb

````    
### 4. Netgen  
download http://opencircuitdesign.com/netgen/archive/netgen-1.5.241.tgz   
commands for installation   
```   
./configure
make
make install   
```   
### 5. magic   
download from http://opencircuitdesign.com/magic/archive/magic-8.3.335.tgz 
```  
./configure
make
make install   
```   
### 6. yosys   
The software used to run gate level synthesis. Yosys is a framework for Verilog RTL synthesis.
To install yosys, install the prerequisites using the following command 
 ```
 sudo apt-get install build-essential clang bison flex \
	libreadline-dev gawk tcl-dev libffi-dev git \
	graphviz xdot pkg-config python3 libboost-system-dev \
	libboost-python-dev libboost-filesystem-dev zlib1g-dev
```
To install latest Version of Yosys, 
```
git clone https://github.com/YosysHQ/yosys.git

make

sudo make install 

make test

```   
## Run Openfasoc Flow   

before that-

1.  cd into the directory where the OpenFASoC repository was cloned;

    Now edit the openfasoc/common/platform_config.json file, replacing the open_pdks value with the path to the sky130A/ directory from open_pdks;
    
2. copy  /home/sritam/OpenFasoc/openfasoc/OpenROAD/build/src/openroad  /home/sritam/OpenFasoc/openfasoc/generators/temp-sense-gen/tools/install/OpenROAD/bin/openroad

3. edit /OpenFasoc/openfasoc/generators/temp-sense-gen/flow/Makefile at line 657 python to python3.9 or python3.8 which is present at your computer/bin/ 

4. edit /OpenFasoc/openfasoc/generators/temp-sense-gen/flow/Makefile , insert a line at 167 export OPENROAD_EXE = /home/sritam/OpenFasoc/openfasoc/generators/temp-sense-gen/tools/install/OpenROAD/bin/openroad


After that   

run flow-  

<p align="center">   
  <img src="images/run_start.png">
</p><br>    

The generator references the model files in an iterative process until either meeting specifications or failing.     
the tool is trying to minimize the error iteratively, by varying the number of inverters and headers for the given temperature range.   

<p align="center">   
  <img src="images/run_flow2.png">
</p><br>    

after the flow completed you get    
<p align="center">   
  <img src="images/run_flow3.png">
</p><br>    

<p align="center">   
  <img src="images/folder_6.png">
</p><br>    

klayout view of 6_final.gds   

<p align="center">   
  <img src="images/Screenshot from 2022-11-21 17-43-00.png">
</p><br>    



## Temperature Sensor Generator   

Circuit   
-------
This generator creates a compact mixed-signal temperature sensor based on the topology from this [paper](https://ieeexplore.ieee.org/document/9816083).   

It consists of a ring oscillator whose frequency is controlled by the voltage drop over a MOSFET operating in subthreshold regime, where its dependency on temperature is exponential.   

![tempsense_ckt](https://user-images.githubusercontent.com/110079631/199317479-67f157c5-6934-470b-8552-5451b1361b9c.png)    

### OpenFASOC Flow
<p align="center">
  <img src="/images/of3.png">
</p><br>   


The steps from the RTL-to-GDS flow look like this, usual in a digital flow:   
<p align="center">   
  <img src="images/tempsense_digflow_diagram.png">
</p><br>    

<p align="center">   
  <img src="images/tempsense_flow_diagram.png">
</p><br>    


```   
temp-sense-gen
├── blocks
└── flow
    └── design
        ├── sky130hd
        │   └── tempsense
        │       ├── config.mk             <--
        │       └── constraint.sdc
        └── src
            └── tempsense
                ├── counter.v             <--
                ├── TEMP_ANALOG_hv.nl.v   <--
                ├── TEMP_ANALOG_lv.nl.v   <--
                ├── TEMP_AUTO_def.v       <--
                └── tempsenseInst_error.v <--
```    
The default circuit’s physical design generation can be divided into three parts:

 1. Verilog generation     

 2. RTL-to-GDS flow (OpenROAD)     

 3. Post-layout verification (DRC and LVS)     

### Verilog generation   

Running make sky130hd_temp (temp for “temperature sensor”) executes the temp-sense-gen.py script from temp-sense-gen/tools/. This file takes the input specifications from test.json and outputs Verilog files containing the description of the circuit.

The generator starts from a Verilog template of the temperature sensor circuit, located in temp-sense-gen/src/. The .v template files have lines marked with @@, which are replaced according to the specifications.  

templetes -

1.TEMP_ANALOG_hv.v      
2.TEMP_ANALOG_lv.v     
3.counter_generic.v     

using this generated files-     

1.TEMP_ANALOG_hv.nl.v    
2.TEMP_ANALOG_lv.nl.v    
3.counter.v    

<p align="center">   
  <img src="images/hv,lv,counter_gen.png">
</p><br>   


 #### what are hv lv file do-   
 
   they are part of temperature sensor circuits.   
   
   #### HV FILE   

![image](https://user-images.githubusercontent.com/110079729/199910810-4962f9ed-95e8-4857-ae3f-8acf9d9fe634.png)

User Specs    

Temperature sensing range: -20⁰C – 125⁰C    

Frequency range of operation: 100Hz – 10MHz    

* Inputs

1. CLK_REF: System clock taken from input.

2. RESET_COUNTERn: Input signal to reset the module initial state.

3. SEL_CONV_TIME: Four bit input used to select how many times the 1 bit of the output DOUT is fractionated (0-16).

* Outputs

1. DOUT: The output voltage whose frequency is dependent on temperature.

2. DONE: Validity signal for DOUT

Theses are the verilog template files which are used for the creation of netlist verilog files.    

We use an all-digital temperature sensor architecture, that relies on a new subthreshold oscillator (achieved using the auxiliary cell “Header Cell“) for realizing synthesizable thermal sensors. We choose frequency as the temperature dependent variable. So, we use a ring oscillators that is based on inverters only and stacked native IO devices for better line sensitivity.    
Since the subthreshold current has an exponential dependency on the temperature, the frequency generated from the subthreshold ring oscillator is also dependent on temperature. Using this principle, we can sense the temperature by comparing the clock generated from a reference oscillator and the clock frequency from our proposed frequency generator.

## Acknowledgement
  
  * Kunal Ghosh, Director, VSD Corp. Pvt. Ltd.
  * Madhav Rao, Professor, IIIT-Bangalore.
  * Nanditha Rao, Professor, IIIT-Bangalore.
  

## Contact Information

  * Sriman Sritam Birtia, Post Graduate student, IIIT Bangalore, Sriman.Birtia@iiitb.ac.in
  * Kunal Ghosh, Director, VSD Corp. Pvt. Ltd. kunalghosh@gmail.com
  * Madhav Rao, Professor, IIIT-Bangalore. mr@iiitb.ac.in
  * Nanditha Rao, Professor, IIIT-Bangalore. nanditha.rao@iiitb.ac.in
   




