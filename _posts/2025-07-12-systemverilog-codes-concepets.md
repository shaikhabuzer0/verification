## Systemverilog Coding Concepts
### Call by value, Call by reference & Call by const reference
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

One more example :)
```verilog
module test;
class packet;
	bit[7:0] data;
endclass

task compute_d(packet p); //Solution: task automatic compute_d(ref packet p);
	p=new();  // this p object is local to this task
	p.data = 10;
	$display("Packet data value = %d", p.data);
endtask
initial begin
	packet p;
	compute_d(p);
	$display("Outside packet data value = %d", p.data); //here you will get null object access Error.
end
endmodule
```
Now in above case, whatever you declare inside task it's scope is limited to task. Thats the reason you will get Error.  Solution to above problem is declare task as automatic and use ref keyword(as shown in commented code).
## const ref
This keyword is for safety purpose, if you want to protect your data from being modified then you can use const ref.

```verilog
module test;
real round_off=0.01; //No one should modify this value 
  real data=5;
  function automatic real get_val(const ref real p, input real value);
	//round_off = 0.02; You will get error, as you can't modify const ref value inside this function
 	return value * p;
endfunction
initial begin
get_val(round_off, data);
  $display("value of round_off=%f, data=%d and computed data=%f", round_off, data, get_val(round_off, data));
end
endmodule
```

## Static Vs Automatic  
By default inside class properties and methods are automatic untill declared static explicitly and inside module properties and methods are static.  
Inside class you can't explicitly mention automatic keyword, it will give compilation error.  
But inside module you can mention static keyword explicitly and you can't declare automatic variable inside module, module is meant to be static.    
```verilog
class base;
function get(); //automatic function
$display("Inside get function");
endfunction
endclass
base b;
b.get();
```
will it print the result? Yes, even without creating object it will print the result as there is no automatic variables accessed inside function.  
```verilog
class base;
function get();//automatic function
int a; //automatic variable
a++;
$display("Inside get function a=%d",a);
endfunction
endclass
base b;
b.get();
b.get();
```
will it print the result? Yes, it will print the results as we are not accessing global automatic variable, int a is local automatic variable to that function.

```verilog
class base;
static int a;
function get();//automatic function
a++;//static variable
$display("Inside get function a=%d",a);
endfunction
endclass
base b;
b.get();
```
will it print the results? Yes, no need to create object as we are accessing static variable inside automatic function.  
```verilog
class base;
static int a;
static function get();//static function
automatic int b; // automatic varialbe, but it's local to method, fine.
a++;//static variable, fine
$display("Inside get function a=%d",a);
endfunction
endclass
base b;
b.get();
```
Getting confused? So just remember the GOLDEN rule.  
If you want to access automatic variables declared outside method, then you must create object of a class, otherwise no need to create the object of class in all other scenarios.

##Fork join
what is the output of below code?
```verilog
initlal begin
for(int i=0; i<3; i++)begin
	fork
		$display("i=%d", i);
	join_none
$display("Outside loop");
end
end
```
How many threads will be scheduled? I'ts 4 threads. because i is static variable.   
first i=0 then condition check(i<3) which is true then goes inside loop and then i++ i.e i=1  
i=1 condition check(i<3) true, i++ i.e i=2  
i=2 condition check(i<3) true, i++ i.e i=3  
i=3 condition check(i<3) false, stop.  
Now you see i already became 3. which will override all other thread values.  

Lets see how static is making a difference...  
first it will schedule the threads and then starts executing  
imagine, delta+0 delta+1 delta+2 delta+3  
            i=0   i=1     i=2     i=3  
the i variable is same so the latest value will override all other values  
  
Now just use automatic variable and store i value in it.  
automatic k=i;  
imagine, delta+0 delta+1 delta+2 delta+3;  
          k=i=0   k=i=1   k=i=2   k=i=3     for each ke there is separate memory so no overriding happens.  
```verilog
initlal begin
for(int i=0; i<3; i++)begin
	automatic k=i;
	fork
		$display("i=%d", k);
	join_none
$display("Outside loop");
end
end
```


