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
