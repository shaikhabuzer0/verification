## Systemverilog Coding Concepts

We will talk about ref keyword in systemverilog.
Do you know the concept of soft link in linux? if not then first you should learn that concept(takes 5min) and come back here. 

In system verilog, ref keyword does similar to soft links, i.e it points to the actual value, its like reference to the actual value.
Here is one simple example:
```verilog
module test;
bit clk=0;
always #5 clk = ~clk;

task example(input bit clk);
  @(posedge clk);
  $display("Posedge of clock %b", clk); //What do you think, how many times this display will get printed? on every clock edge will it get printed?
                                        //The answer is no, because clk variable which is inside task is not linked with the actual clock which is generated.
                                        //To print this display at every posedge of clock you need to link the clk variable of task with actual clk variable
                                        //task example(ref input bit clk); give a try on edaplayground you will get more clarity
endtask
initial begin
  example(.clk(clk)); // 
end
```
