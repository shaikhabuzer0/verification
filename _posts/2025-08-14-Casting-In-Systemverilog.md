# Casting in Systemverilog
Casting means converting from one format to other format.
There are two types of casting. 
1. Static Casting --> happens during compile time  
    * Implicit Casting
    * Explicit Casting(tick operator " ' ")
Applications of static casting.  
It is used to just convert one data type to another data type.  
```verilog
int var;
var = 34.87; // Implicit casting
var = int'(34.87); //explicit casting
```

Static casting does not check out of bound value. It wont throw error if you assign out of bound value.  
```verilog
module test;
enum {RED, GREEN, BLUE} colour_e; //holds value 0,1,2 only
initial begin
//Static casting
colour_e = 3;// out of bound value it won't throw error
colour_e = int'(3);// out of bound value it won't throw error

//Dynamic casting
$cast(colour_e, 3);// it will throw error as 3 is out of bound value

end
endmodule
```

2. Dynamic Casting($cast task/function)  --> happens during runtime  
   upcasting   
   downcasting

The following is example of upcasting(parent is upgrading here)   
parent class   
&#8595;    
child class   
Here parent is pointing to child object. This is done by simulator itself.
i.e p = c;  

The following is example of downcasting(child is downgrading here)     
parent class   
&#8593;    
child class   
i.e c = p;  
Here child is pointing to parent. We need to do this using $cast 

Following is the example of $cast 
```verilog
class transaction;
int addr;
endclass
transaction t1, t2;
t2.copy(t1); //from t2 handle we are calling copy method and copying t1 into t2

function void do_copy(uvm_object rhs); //rhs -> t1 i.e rhs is pointing to t1 i.e parent is pointing to child
//this.addr = rhs.addr; but parent cannot access child properties hence use $cast
transaction rhs_;
$cast(rhs_, rhs); //rhs_ -> rhs -> t1
this.addr = rhs_.addr; //i.e t2.addr = rhs_->t1.addr
endfunction

```
Complete working code:
```verilog
module test;
import uvm_pkg::*;
`include "uvm_macros.svh"
class txn extends uvm_object;
	`uvm_object_utils(txn);
rand int addr;
virtual function void do_copy(uvm_object rhs); //rhs -> t1; 
//	this.addr = rhs.addr;// we can't access child properties from parent handle	
	txn rhs_;
	super.do_copy(rhs);
	$cast(rhs_, rhs);//rhs_ -> rhs - > t1 i.e rhs_ -> t1
	this.addr = rhs_.addr; //this -> t2
	//t2.addr = rhs_->t1.addr;
	//this keyword refers to the handle through which the copy method is called
endfunction
virtual function bit do_compare(uvm_object rhs, uvm_comparer comparer);
	txn rhs_;
	$cast(rhs_, rhs);//rhs_ -> rhs - > t1 i.e rhs_ -> t1
	super.do_compare(rhs, comparer);

return (this.addr == rhs_.addr);
endfunction
virtual function void do_print(uvm_printer printer);
	super.do_print(printer);
printer.print_int("addr", addr, $bits(addr) ,UVM_HEX);
endfunction
endclass

txn t1, t2;

initial begin
t1 = txn::type_id::create("t1");
t2 = txn::type_id::create("t2");
t1.randomize();
t2.copy(t1); //copy t1 into t2
t1.print();
t2.print();
if(t2.compare(t1))begin
	$display("t1 and t2 are same");
end
end
endmodule

```
Are you bored of doing static casting? there is a better way to do it with the help of unions..  
```verilog
Q. Convert bit[7:0] u_addr to byte s_addr
typedef union packed{
	bit[7:0] u_addr;
	byte s_addr;
} u_to_s_conv;
u_to_s_conv usc;
usc.u_addr = 8'hFF; //255 unsigned
$display("signed value is %h", usc.s_addr);// -1

in above case 255 got converted to signed number.

if you received 32bits of number and you want to extract address fields(4bits) from the received packet then unions are useful

typedef struct packet{
	bit[3:0] addr; //MSB
	bit[27:0] data;//LSB
} header_s;

typedef union packed{
	header_s h;
	bit[31:0] packet; //Note inside union all members size must be same, packet and h size must be same.
}
packet_u pkt_u;
pkt_u.packet = dut.output; //received 32bits of packet

if(pkt_u.h.addr == 4'd6)begin //alternate way is if(packet[31:28] == 4'd6) which is not recommended
	if(pkt_u.h.data== 28'dFFFF_FFF)begin //alternate way is if(packet[27:0] == 4'd6)
		$display("Received correct data");
	end
end

```
