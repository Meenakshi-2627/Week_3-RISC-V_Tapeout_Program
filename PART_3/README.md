# 🧠 Week 3 – OpenSTA Timing Analysis and Automation (Innovative Workflow)

## 🔍 Overview
This week’s focus is on performing **Static Timing Analysis (STA)** using **OpenSTA** with Docker, Liberty files, and gate-level netlists — combined with **our innovation** of automating min/max delay analysis across different PVT corners for the **VSDBabySoC** project.


---

## ⚙️ Step 1: Git Clone and Setup

```bash
git clone https://github.com/parallaxsw/OpenSTA.git
cd OpenSTA/
```

### 🐳 Install Docker
```bash
sudo apt update
sudo apt install docker
```

**Note:** Docker group requires root or user with docker group access.

### Build the Docker Image
```bash
sudo docker build --file Dockerfile.ubuntu22.04 --tag opensta .
```  
![docker_build](./Images/docker_build.png)

**Error Fix Tip:**  
If you face this error:
```
permission denied while trying to connect to the Docker daemon socket
```
Use:
```bash
sudo usermod -aG docker $USER
sudo systemctl restart docker
```

### Run the OpenSTA Shell
```bash
sudo docker run -i -v $HOME:/data opensta
```  
![docker_run](./Images/docker_run.png)

---

## 🧩 Example 1 – Basic Timing Analysis

**Script**
```tcl
read_liberty /OpenSTA/examples/nangate45_slow.lib.gz
read_verilog /OpenSTA/examples/example1.v
link_design top
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}
report_checks
```

**Sample Report**
The delays through the cells can be compared by openSTA shell's report_timing
```
Startpoint: r2 (FF)
Endpoint: r3 (FF)
Slack (MET): 9.43 ns
```
![report_check1](./Images/report_check1.png)

---

## ⚡ Synthesis and Netlist Verification (Yosys Flow)

```bash
gunzip -k nangate45_slow.lib.gz
gunzip -k nangate45_fast.lib.gz
```

**In yosys:**
```bash
read_liberty -lib nangate45_slow.lib
read_verilog example1.v
synth -top top
show
```  
![example1_netlist](./Images/example1_netlist.png)

**Output:**
```
=== top ===
Cells:
  AND2_X1 : 1
  BUF_X1  : 1
  DFF_X1  : 3
```

---

## 🧮 SPEF-Based Analysis

This analysis is done for the parasitic delay due to capacitor , resistor ,etc..

```bash
sudo docker run -i -v $HOME:/data opensta
```

**In Yosys**
```tcl
read_liberty /OpenSTA/examples/nangate45_slow.lib.gz
read_verilog /OpenSTA/examples/example1.v
link_design top
read_spef /OpenSTA/examples/example1.dspef
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}
report_checks
```

![report_check2](./Images/report_check2.png) 

Result shows parasitic delay effects, lowering slack due to net capacitance.

---

## 📊 Detailed Reports

### 1️⃣ Capacitance Report
```tcl
report_checks -digits 4 -fields capacitance 
```
![report_check3](./Images/report_check3.png)


### 2️⃣ Slew & Fanout Report
```tcl
report_checks -digits 4 -fields [list capacitance slew input_pins fanout]
```   
![report_check4](./Images/report_check4.png)

### 3️⃣ Power Report
```tcl
report_power
```  
![report_check5](./Images/report_check5.png)

### 4️⃣ Pulse Width Report
```tcl
report_pulse_width_checks
```  
![report_check6](./Images/report_check6.png)

### 5️⃣ Units Check
```tcl
report_units
```  
![report_check7](./Images/report_check7.png)


---

## 💡 Automated Min/Max Delay Calculation

We automated OpenSTA’s flow via a TCL script to simplify multi-lib and multi-path delay verification.

**min_max_delays.tcl**
```tcl
read_liberty -max /data/OpenSTA/OpenSTA/examples/nangate45_slow.lib.gz
read_liberty -min /data/OpenSTA/OpenSTA/examples/nangate45_fast.lib.gz
read_verilog /data/OpenSTA/OpenSTA/examples/example1.v
link_design top
create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}
report_checks -path_delay min_max
```

**Run Command**
```bash
sudo docker run -it -v $HOME:/data opensta /data/OpenSTA/OpenSTA/examples/min_max_delays.tcl
```

---

## 🧱 VSDBabySoC Custom STA

### Directory Setup
```bash
mkdir -p examples/timing_libs/
mkdir -p examples/BabySoC/
```

**Copy:**
```
sky130_fd_sc_hd__tt_025C_1v80.lib
avsdpll.lib
avsddac.lib
vsdbabysoc.synth.v
vsdbabysoc_synthesis.sdc
```

### File Paths
```
/data/OpenSTA/OpenSTA/examples/timing_libs/
/data/OpenSTA/OpenSTA/examples/BabySoC/
```

**vsdbabysoc_min_max_delays.tcl**
```tcl
read_liberty -min /data/OpenSTA/OpenSTA/examples/timing_libs/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -max /data/OpenSTA/OpenSTA/examples/timing_libs/sky130_fd_sc_hd__tt_025C_1v80.lib

read_liberty -min /data/OpenSTA/OpenSTA/examples/timing_libs/avsdpll.lib
read_liberty -max /data/OpenSTA/OpenSTA/examples/timing_libs/avsdpll.lib

read_liberty -min /data/OpenSTA/OpenSTA/examples/timing_libs/avsddac.lib
read_liberty -max /data/OpenSTA/OpenSTA/examples/timing_libs/avsddac.lib

read_verilog /data/OpenSTA/OpenSTA/examples/BabySoC/vsdbabysoc.synth.v
link_design vsdbabysoc
read_sdc /data/OpenSTA/OpenSTA/examples/BabySoC/vsdbabysoc_synthesis.sdc

report_checks -path_delay min_max
```

**Run Command**
```bash
sudo docker run -it -v /home/meena:/data opensta /data/OpenSTA/OpenSTA/examples/BabySoC/vsdbabysoc_min_max_delays.tcl
```

---

## ⚠️ Common Errors & Fixes

### ❌ Liberty Syntax Error
Error:
```
/avsdpll.lib line 54, syntax error
```
Fix:
Replace from line 54 with:
```liberty
/* pin (GND#2) {
   direction : input;
   max_transition : 2.5;
   capacitance : 0.001;
} */
pin (VDD) {
   direction : input;
   max_transition : 2.5;
   capacitance : 0.001;
}
```

---

## 🌡️ Advanced: STA Across PVT Corners

**sta_across_pvt.tcl**
```tcl
set list_of_lib_files {
    "sky130_fd_sc_hd__tt_025C_1v80.lib"
    "sky130_fd_sc_hd__ss_100C_1v60.lib"
    "sky130_fd_sc_hd__ff_n40C_1v95.lib"
}

foreach lib $list_of_lib_files {
    read_liberty $lib
}

read_verilog /data/OpenSTA/OpenSTA/examples/BabySoC/vsdbabysoc.synth.v
link_design vsdbabysoc
read_sdc /data/OpenSTA/OpenSTA/examples/BabySoC/vsdbabysoc_synthesis.sdc
report_checks -path_delay min_max
```

---

## 📦 External Resource for library files 

**Liberty Library Download:**
[https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing](https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd/tree/master/timing)

```bash
git clone https://github.com/efabless/skywater-pdk-libs-sky130_fd_sc_hd.git
cd skywater-pdk-libs-sky130_fd_sc_hd/timing
```  
![git_clone3](./Images/git_clone3.png)   

## min/max delay lib text files after .tcl

![listed_files](./Images/listed_files.png)

## ⏱️ Timing Graphs  

**After generating min/max delay lib files , generate the timing graphs using the following commands**
```
report_checks -path_delay max -n_paths 5 -format dot > sta_tns.dot #Inside the OpenSTA shell, after linking and clock/delay setup
dot -Tpng sta_tns.dot -o sta_tns.png  #Then render it with Graphviz
```

<details> <summary>▶ STA – TNS (Total Negative Slack)</summary>

![sta_tns](./Images/sta_tns.png)


</details> <details> <summary>▶ STA – WNS (Worst Negative Slack)</summary>

![sta_wns](./Images/sta_wns.png)


</details> <details> <summary>▶ STA – Worst Min Slack</summary>

![sta_worst_min_slack](./Images/sta_worst_min_slack.png)


</details> <details> <summary>▶ STA – Worst Max Slack</summary>

![sta_worst_max_slack](./Images/sta_worst_max_slack.png)


</details>
