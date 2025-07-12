## Systemverilog Coding Concepts
### Call/Pass by reference
We will talk about ref keyword in systemverilog.  
There are two rules to use ref keyword.  
  R1- task/function must be automatic  
  R2- When you are calling task/function you have to pass variables as arguments not the values.  
    ex: example(a) is correct  
    example(5) is not correct  
Do you know the concept of soft link in linux? if not then first you should learn that concept(takes 5min) and come back here. 

In system verilog, ref keyword does similar to soft links, i.e it points to the actual value, its like reference to the actual value.  
Here is one simple example:  
```verilog
module test;
bit clk=0;
always #5 clk = ~clk;

task automatic example(input bit clk);
  forever begin
  @(posedge clk);
  $display("Posedge of clock %b", clk);  //What do you think, how many times this display will get printed? on every clock edge will it get printed?
                                        //The answer is no, because clk variable which is inside task is not linked with the actual clock which is generated.
                                        //To print this display at every posedge of clock you need to link the clk variable of task with actual clk variable
                                        //task example(ref input bit clk); give a try on edaplayground you will get more clarity
end
endtask
initial begin
  example(.clk(clk)); // 
end
```
Let's understand it with different way. 
int data=5; // that means at address 0x001 number 5 is stored.

giving reference means pointing to the address of that variable.  
task exmpale(ref input bit data);// this data variable address is also 0x001. which means if the data variable declared above got changed   then this data variable present inside task will get updated automatically.  

Another example.  
```verilog
module test;

int x;

function automatic void pbr(ref int x);
	x=x*2;
	$display("Inside pbr function x=%0d",x); //x=8
endfunction

initial
begin
x=4;
pbr(x);
  $display("Outside the pbr function x=%0d",x); //x=8

end
endmodule
```
If you observe, everywhere the value of x is same because all the variables have the same address.
