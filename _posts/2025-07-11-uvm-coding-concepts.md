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
# Virtual sequence and Virtual sequencer

what is stimulus coordination?  
Suppose I have two drivers inside my environment, one is apb and other is ucie, first I want to configure CSR space of ucie and then drive the ucie flits.  
inside my testcase, first I will start  
apb_seq.start(apb_seqr);  
ucie_seq.start(ucie_seqr);  
here, instead of providing the complete path of sequencer, I am just giving the name of the sequencer, this is possible because the concept of virtual sequence and sequencer.  

```verilog
class apb_txn extends uvm_sequence_item;
addr;
data;
endclass

class ucie_txn extends uvm_sequene_item;
rand typedef enum{68B_flit, 256B_LOPT_FLIT, 256B_STD_HDR_FLIT} flits;
endclass

class vsequencer extends uvm_sequencer;
apb_seqr;
ucie_seqr;
endclass

class env extends uvm_env;
vsequencer vseqr_h;
apb_agent apb_agt;
ucie_agent ucie_agt;
//connect 
vseqr_h.apb_seqr = apb_agt.apb_seqr;
vseqr_h.ucie_seqr = ucie_agt.ucie_seqr;
endclass

class vsequence_base extends uvm_sequence;//by default it will get parameterized with uvm_sequence_item
vsequencer vseqr_h;
// or 
`uvm_declare_p_sequencer(vsequencer) it is same as
vsequencer p_sequencer 
and $cast(p_seqeuncer, m_sequencer)
//

apb_sequencer apb_seqr;
ucie_sequencer ucie_seqr;

apb_seqr = vseqr_h.apb_seqr;
ucie_seqr = vseqr_h.ucie_seqr;

//OR apb_seqr = p_sequencer.apb_seqr;
endclass

class vsequence extends vsequence_base;
apb_sequence apb_seq;
ucie_sequence ucie_seq;

apb_seq.start(apb_seqr);
ucie_seq.start(ucie_seqr); 

endclass  
```

class test;
vsequence vseq;
vseq.start(env.vseqr_h);
endclass
