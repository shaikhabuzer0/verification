## UVM Coding Concepts

# Hello world code in UVM
```verilog
module top;
  import uvm_pkg::*;
  `include "uvm_macros.svh" // add only if you are using Questasim simulator

  initial begin
    `uvm_info("top", "Hello World!", UVM_NONE)
  end
endmodule
```
If you are using synopsys VCS tool, use the following command.
vcs -sverilog -full64 -ntb_opts uvm -R top.sv  

Commands for vcs  
vcs -> compilation and elaboration  
./simv -> execute the simulation  

Commands for Questasim  
vlog -> compilation  
vsim -> for simulation
run -all -> once the simulation starts then it will tell how long simulation should run i.e in this case it is till $finish   
run 100ns -> it will execute the simulation till 100ns duration  

vsim view -vsim.wlf -> to open waveform  
add wave * -> to add all the signals to waveform window   
restart -f -> it will restart the simulation  

Makefile for Questasim simulator  
```make
INCDIR = +incdir+./
COVOPT = -coveropt 3 +cover=bcft
VSIMOPT = -vopt -voptargs=+acc
VSIMBATCH = -c -do "log -r /* ; run -all;"
work = work
files = fifo.sv
TOP = top #top module name

comp:
        vlib $(work) #library creation
        vmap work $(work) #library mapping
        vlog -work $(work) $(INC) $(files) #compilation

sim: comp
        vsim $(VSIMOPT) $(VSIMCOV) $(VSIMBATCH) -wlf wave.wlf -l run.log -sv_seed random work.$(TOP)

run: sim
dump:
        vsim -gui -view wave.wlf
```
