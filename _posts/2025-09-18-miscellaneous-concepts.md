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

//Below code avoids race condition, and values will get swapped
always@(posedge clk)begin
  a<=b;
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
