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
