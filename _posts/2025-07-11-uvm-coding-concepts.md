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
