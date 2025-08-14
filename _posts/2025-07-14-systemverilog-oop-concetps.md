## Systemverilog OOPs Concepts
# Class (Encapsulates the data)
It is a user defined data type. It wraps the data in single unit.  
Everything inside the class is dynamic, to make it static use static keyword while declaring properties/methods. 
Dynamic --> User have to create memory for it.   
Static --> Compiler will create memory for it.   
```verilog
// Code your testbench here
// or browse Examples
module top;
class packet;
	int data; //it is a public member, visible to all
	local int addr; //now this variable is not publically visible as it is declared as local. Even this class handle cannot acces this variable.

//Class constructor, even if you don't declare it compiler will add class constructor. It is used to create the object of class. 
	function new(int addr=12, data);
		this.addr = addr;// this keyword is nothing but predefined handle of the class. To access the class members within the class we use this keyword  
		this.data = data;//this.data = data is nothing but p.data = data
	endfunction
  
  function void print();
    $display("addr = %d\n data = %d\n", addr, data);
  endfunction
endclass
  
packet p; //handle/pointer/instance of a class packet. It stores the address of object.
  
initial begin
p=new(,15); //object creation. i.e memory is created and its address is stored inside p, and all the members are initialized to their defalut value
//addr is initialized to 12, and data is initialized to 15.
p.data = 10;
//p.addr = 12;//not allowed as addr is local to class packet. it can't be accessed outside of class.
p.print();
end
endmodule
```
[class edaplayground link](https://www.edaplayground.com/x/hBJU)  

Note: without creating object of class still we can access its methods. but there are conditions as follows
1. Methods should not be virtual
2. Methods should not contain class members
Note2: static members/methods of class can be accessed without creating the object of class with the help of scope resolution operator. i.e ::

# Inheritance 
Suppose you sold VIP to a customer, now customer came back to you and asked to add new features to VIP. So simply by extending existing class 
we can add new features.
Points to remember:
  - Child handle can access parent's class properties and methods
  - Parent handle can't access child's class properties.(Methods can be access with the help of polymorphism)
  - Parent handle can point to child object i.e p=c check who is handle and who is object here..
  - Child cannot point to parent object i.e c=p (Possible using $cast)
