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

```verilog
logic clk;
logic reset_n;

logic [7:0] data_in; //data to be routed
logic pkt_valid; // valid packet

logic read_en;
logic [7:0] data_out; // output data
logic vld_out; // output data is valid

logic err; // parity mismatch error
logic busy; //if busy is high then don't drive the next byte 
```
packet structure is as follows    
header   --> includes length[7:2] and addr[1:0] i.e 2bit of address but only 0,1,2 are valid as we have only 3 destinations   length is of 6bits which means 2^6=64bytes of data we can transfer   
payload  parity = parity ^ payload[i]  
parity  --> 8bits parity = parity ^ header  

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
