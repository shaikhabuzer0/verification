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
# Callback
Polymorphism's advanced version is callback.
Without changing the code, changing the behaviour of the code.  
Callback involves virtual method, inheritance and handle assignment.  
```verilog
module test;
typedef enum{GOOD, BAD1, BAD2} pkt_type;
class driver;
pkt_type pkt;
task send_pkt();
std::randomize(pkg) with {pkt == GOOD;};
modify_pkt(); // will get called only if inject_error = 1 otherwise dummy method will get called.  
endtask

//dummy virtual method
virtual task modify_pkt():
endtask

endclass

class err_driver extends driver;
task modify_pkt();
std::randomize(pkt) with {pkt inside {BAD1, BAD2};};
endtask
endclass

class env;
	driver d;
	err_driver ed;
function new();
	d=new();
	ed=new();
endfunction

task execute();
	if(inject_error)begin
		d = ed;
	end
	d.send_pkt();
	$display("Sending pkt = %s", d.pkt.name());
endclass
initial begin
	env e;
	e = new();
	e.execute(); // good pkt
	e.inject_error = 1;
	e.execute(); // bad pkt
end
endmodule
```

### Frequency check assertion
Given clk_period = 20
```verilog
property fcheck(int clk_period);
time prev_t;
@(edge) //checking only half period
(1, prev_t = $time) |=> ( (clk_period/2) == $time - prev_time); 
endproperty
```
OR
```verilog
property fcheck(int clk_period);  
time prev_t;  
@(posedge clk) //checking only half period  
(1, prev_t = $time) |=> ( (clk_period) == $time - prev_time);  
endproperty  
```
Along with tolerance of +-5% on time period  
```verilog
20*5/100 = 21
i.e clk_period *(100 + tolerance)/100

property fcheck(int clk_period, int tolerance);  
time prev_t;
time measured;
@(posedge clk) //checking only half period  
(1, prev_t = $time) |=> ((measured = $time - prev_time, 
						  measured >= clk_period * (100 - tolerance)/100
						  &&
						  measured <= clk_period * (100 + tolerance)/100 	
						);  
endproperty

why we added 100 to tolerance?
clk_periode = 20
tolerance = 5
5/100 = 0.05 indicates its just 5% but we want 1.05 incremented version so that we can directly multiply this number with clk_period  
(100 + 5)/100 = 105/100 = 1.05  
```
## Basics

left shift operation(multiplication).  
```verilog
logic [3:0] data;  
bit in;  
data << 1;// left shift by 1 bit and append zeros on LSB  
data << 1 | in; // left shift by 1 bit and append incoming new bits  
{data[2:0], in}; // does exactly as above i.e it removes the MSB and appends new bits to LSB  
```
READ MODIFY WRITE
```verilog
logic[7:0]register=8'hFF; // want to make it 8'hAF  
logic[7:0] temp;  

temp = register;  
temp = {4'hA, 4'b0} // result is A0  
		|  
		(8'hAF & 8'h0F); // result is 0F  
register = temp; // AF  
```
