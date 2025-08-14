# Casting in Systemverilog
Casting means converting from one format to other format.
There are two types of casting. 
1. Static Casting --> happends during compile time  
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

Best examaple of using $cast
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
