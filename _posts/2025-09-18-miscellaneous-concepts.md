## Miscellaneous Concepts
The golden rule in Verilog is  
Use nonblocking assignments whenever one process writes to a variable and another process read the same variable and both processed are synchronized to the same event.  
https://verificationacademy.com/forums/t/question-on-fork-join-with-case-statement/49004

How to enable or disable scoreboard from sequence?
https://stackoverflow.com/questions/28058475/disabling-a-scoreboard-from-a-sequence-using-uvm/28082459  
```verilog
//Below code will cause race condition, to avoid race condition use non blocking assignment.
always@(posedge clk)begin
  a=b;
  b=a;
end
always@(posedge clk)begin
  b=a;
end

//Below code avoids race condition, and values will get swapped
always@(posedge clk)begin
  a<=b;
  b<=a;
end
always@(posedge clk)begin
  b<=a;
end
```
# UCIe Protocol  
Let's understand bandwidth, frequency, interface width calculation for UCIe protocol  
UCIe max link speed is 32GT/s and x32 is the max lanes we can have, so let's calculate bandwidth  

BW per lane = (32GT/s) / 8 = 4GB/s  
BW for 32 lanes = 4GB/s * 32 = 128GB/s  

BW = frequency * interface width(bits carrying in one clock)  

We have to make adjustments in frequency or interface width to meet 128GB/s BW.  

so, let's assume interface width = 128bytes  

BW/interface width = frequency  
128/128 = 1 GHz frequency  

Simillarly, if we want 2 GHz frequency then we have to make adjustments in interface width  
128/64 = 2 GHz frequency we will get if we make 64byte interface width  

# 1x3 Router  
interface signals are listed below  
we need 1 agent which is driving packets to router dut and 3 agents to capture these packets  
```verilog
logic clk;
logic reset_n;

logic [7:0] data_in; //data to be routed
logic pkt_valid; // valid packet

logic read_en; //read agent will drive this signal, and it's monitor will capture the data and send it to the scoreboard  
logic [7:0] data_out; // output data
logic vld_out; // output data is valid

logic err; // parity mismatch error
logic busy; //if busy is high then don't drive the next byte 
```
packet structure is as follows    
header   --> includes length[7:2] and addr[1:0] i.e 2bit of address but only 0,1,2 are valid as we have only 3 destinations   length is of 6bits which means 2^6=64bytes of data we can transfer   
payload  parity = parity ^ payload[i]  
parity  --> 8bits parity = parity ^ header  

Testscenarios  
0. check address decoding logic  
1. check error signal by injecting parity error  
2. check payload len, corner scenarios like len=0 , len=1 and len=63, len=64  
3. corrupt parity  
4. send packet when busy is high  
5. multiple pakcets of same len  
6. multiple packets of different len
7. valid check, send packet without valid high


Transaction class
```verilog
rand logic[7:0] header;
rand logic[7:0] payload[];
logic [7:0] parity;

constraints
0. header[1:0] should not get value equal to 3
constraint inval_addr{
  header[1:0] != 3;
}

1. length must be not 0
constraint val_len{
  header[7:2] > 0;
}

2. payload size should be equal to header[7:2]
constraint payload_size{
  payload.size == header[7:2];
}

post_randomize()
parity = parity ^ header;
for(int i=0; i<header[7:2]; i++)begin
parity = parity ^ payload[i];
end


```

Driving logic 
```verilog

task run_phase();
intf.resetn <= 1'b0;
@(posedge clk)
intf.resetn <= 1'b1;

forever begin
seq_item_port.get_next_item(req);
drive();
seq_item_port.item_done();
end
endtask
task drive();
  wait(!intf.busy)
  intf.pkt_valid <= 1;
  intf.data_in <= req.header;
  @(posedge clk)

for(int i=0; i<req.header[7:0](its nothing but length); i++)begin
  wait(~intf.busy)
  intf.data_in <= req.payload[i];
  @(posedge clk)
end

wait(~intf.busy);
intf.pkt_valid <= 0;
intf.data_in <= req.parity;
@(posedge clk)

endtask

```
Driving and monitoring logic for read agent

```verilog
driver

run_phase();
wait(vld_out)
one cycle delay
read_en<=1;
wait(vld_out)
one cycle delay
read_en<=0;

monitor
run_phase();
forever
collect_data();

collect_data();
wait(read_en)
once cycle delay
txn.header=vif.header;
for loop
once cycle delay
txn.payload[i] = vif.payload[i];

once cycle delay
txn.parity = vif.parity;
once cycle delay
ap_r.write(txn); //analysis port broadcasting

```

## APB Protocol Verification
```verilog
task run_phase(uvm_phase phase);
  drive_reset();
  forever begin
    seq_item_port.get_next_item();
    if(tr.pwrite == 1)begin
      drive_write();
    end else begin
      drive_read();
    end
    seq_item_port.item_done();
  end
endtask

task drive_write();
  //setup phase
  intf.psel <= 1;
  intf.pwrite <= 1;
  intf.paddr <= tr.paddr;
  intf.pwdata <= tr.pwadata;
  //access phase
  @(posedge pclk);
  intf.penable <= 1;
  while(intf.pready == 0) @(posedge pclk);
  tr.pslverr <= intf.pslverr;
  intf.penable <= 0;
endtask

task drive_read();
  //setup phase
  intf.psel <= 1;
  intf.pwrite <= 0;
  intf.paddr <= tr.paddr;
  intf.pwdata <= 0;
  //access phase
  @(posedge pclk);
  intf.penable <= 1;
  while(intf.pready == 0) @(posedge pclk);
  tr.prdata <= intf.prdata;
  tr.pslverr <= intf.pslverr;
  intf.penable <= 0;
endtask

task drive_reset();
  intf.psel <= 0;
  intf.penable <= 0;
  intf.pwrite <= 0;
  intf.paddr <= 0;
  intf.pwdata <= 0;
  intf.prdata <= 0;
  intf.p <= 0;
endtask

```
### Monitor 
it captures the dut output only when handshake is completed. i.e if psel && penable && pready are high then only it will capture pwdata, prdata
```verilog
class apb_mon extends uvm_monitor;
uvm_analysis_port#(apb_transaction) m_ap;
function new(string name="", uvm_component parent = null);
  super.new(name, parent);
  m_ap = new("m_ap", this);
endfunction

task run_phase(uvm_phase phase);
  apb_transaction tr;
  forever begin
    tr = apb_transaction::type_id::create("tr"); //each time create fresh object and store the data and send to scb// this avoids cloning of object
    @(posedge pclk)
    if(psel && penable && pready)begin
      tr.paddr = vif.paddr;
      tr.pslverr = vif.pslverr;
      if(tr.pwrite)begin
        tr.pwdata = vif.pwdata;
      end else begin
        tr.prdata = vif.prdata;
      end
    end
  m_ap.write(tr);
  end
endtask
endclass
```
### Scoreboard
declare analysis imp port
declare one memory to store the write txn and to compare when read txn is initiated
implement write method, just store the incoming txn into queue then in run phase take out elements and do compare operations

```verilog
class apb_scoreboard extends uvm_scoreboard;
uvm_analysis_imp#(apb_transaction, apb_scoreboard) s_ap;
apb_transaction exp_queue[$];
bit[31:0] mem[8];
function new();
s_ap = new("s_ap", this);
endfunction

funciton void write(apb_transaction tr);
if(tr.pslverr)begin
`uvm_warning("pslverr has been seen", UVM_NONE)
end else begin
exp_queue.push_back(tr);
end
endfunction

task run_phase(uvm_phase phase);
apb_transaction expdata; //store the incoming pkt from monitor and check the results
forever begin
  wait(exp_queue.size >0);
  expdata = exp_queue.pop_front(); //extract first pkt and check
  if(expdata.pwrite == 1)begin
    mem[expdata.paddr] = expdata.pwdata;
  end else if(expdata.pwrite == 0)begin
    if(expdata.prdata != mem[expdata.paddr])begin
      `uvm_error("read data mismatch", UVM_NONE)
    end else begin
      `uvm_info(get_type_name(), "READ MATCH", UVM_NONE)
    end
  end
end
endtask
endclass
```
