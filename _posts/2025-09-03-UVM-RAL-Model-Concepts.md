## UVM RAL Model Concepts

**Desired value:** the value which you want to write into the hardware register  
**Mirrored value:** the value which is actually present inside the hardware register, this value gets updated after each write/read txn

Example:  
initially both values are 0's  
desired=0, dut=0, mirrored=0  
Now I want to write 2 into dut, first I have to call set method  
set(2)  
desired=2, dut=0, mirrored=0  
update(2) //after calling update, we are writing value 2 into dut register  
desired=2, dut=2, mirrored=2  

To configure the desired value we have two methods.  
set() and randomize()  

**Write vs update method.** 
upate method will first check whether desired and mirrored values are different? if they are different then only it will initiate the write transaction(it will write the desired value which is configured by set/randomize method), otherwise if desired and mirrored values are same then it won't initiate any write txn.  
While write txn simply write the desired value by initiating write txn.(there are no checks)  

**We have three types of predictors.**  
1. Implicit/auto predictor --> defalut_map.set_auto_predict(1)
2. Explicit predictor --> contains adapter and map --> useful if you want to implement functional coverage
uvm_reg_predictor urp;  
urp.map = map;  
urp.adapter = adapter_inst;  
3. Passive predictor  

**predict()** --> updates the desired and mirrored value of a register in RAL model. No txn is performed on hardware registers  
**mirror()** it internally calls predict() --> Read the hardware register and upate/check its existing mirror value.    

Example:  
write(2) // desired=2 mirrored=2 dut=2  
predict(3) // desire=3, mirrored=3, dut=2  
mirror() // dut=2 != mirrored=3 throws error if UVM_CHECK is enable  
---------------------------------------------------------------------------------
Building blocks of RAL model
registers --> build with the help of uvm_reg class, it's job is to give register details with the help of configure method
register block --> built using uvm_reg_block class, it's job is to give address mapping
adapter block --> uvm_reg_adapter, it's job is to reg to bus txn and bus to reg txns
predictore block --> uvm_reg_predictor use to capture response of dut and send to register model
reg sequence --> uvm_sequence, which will have write, read, update etc methods

flow of the txn is given below  
registers --> adapter --> sequencer --> driver --> interface --> DUT register
DUT/register --> interface --> monitor --> predictor(adapter)--> register block

To model registers we have uvm_reg class, to model memories we have uvm_mem class  

## uvm_reg class(to model registers present inside dut)

In this class we will declare the register that we want to verify.  
ex: inside dut we have cntrl register of 32bits then we will declare this register as uvm_reg_field type
and we will configure this register using build in configure method

```verilog
1. declare all registers  
2. create all registers  
3. configure all registers  

Note: this cntrl register have only one field of 32bits, if you have multiple fields then you have to declare those fields and configure each of them.  
ex: [31:0]cntrl register = [31:16]addr, [15:0]data field  

class cntrl_reg extends uvm_reg; //uvm_reg is an object not the component
  rand uvm_reg_field cntrl; //uvm_reg_field is also a class which has a method called configure

  function new(string name="cntrl");
    super.new(name, 32/*size of register i.e cntrl register is 32bits */, UVM_NO_COVERAGE/**/); // It has 3 arguments
  endfunction

//As uvm_reg is object it don't have phases so defining build function manually to create register instance to configure it
function void build();
cntrl = cntrl_reg::type_id::create("cntrl");
//configure method have 8 arguments
cntrl.configure(.parent(this),
		  .size(32), //size of the field
      .lsb_pos(0), //LSB 0 MSB 31
		  .access("RW"),
		  .volatile(0), //1- the value of the register is not predictable because it may change between consecutive accesses(DUT MAY INTERNALLY UPDATE THIS FIELD). Moslty it is zero.
		  .reset(0), //default value when reset is applied
		  .has_reset(1),// does this field support reset? yes most of the cases
		  .is_rand(1), // if access is RW then this field can be random, if access is RO then this is_rand must be zero
		  .individually_accessible(1)); // can be access individually 
endfunction

```
Register declaration is done, now it's time to give address mapping for these registers with the help of uvm_reg_block class

```verilog
class top_reg extends uvm_reg_block;
rand cntrl_reg cntrl_reg_inst;
function new(string name="top_reg");
  super.new(name, UVM_NO_COVERAGE); //it has only two arguments
endfunction

function void build();
cntrl_reg_inst = cntrl_reg::type_id:create("cntrl_reg_inst");
//call build function of uvm_reg class
cntrl_reg_inst.build();
cntrl_reg_inst.configure(); //this configure will configure with respect to register block

//create a map and then add your registers inside that map
defalut_map = create_map("default_map", 0/*base addr*/, 4/*width of bus in bytes*/, UVM_LITTLE_ENDIAN /*reg[31:0]*/) 
default_map.add_reg(cntrl_reg_inst, 'h0/*register offset*/ ,"RW" /*just mention here RW and actual access you mention while configuring fields of register*/);
endfunction
endclass
```
Now adapter component
```verilog

class top_adapter extends uvm_reg_adapter;

function new(string name="top_adapter"); //normal constructor
  super.new(name);
endfunction

function uvm_sequence_item reg2bus(const ref uvm_reg_bus_op rw);
  transaction tr;
  tr=transaction::type_id::create("tr");
  tr.addr = rw.addr; //accordingly we have to make 
endclass

function void bus2reg(uvm_sequence_item bus_item, ref uvm_reg_bus_op rw);
  transaction tr;
  assert($cast(tr, bus_item));
  rw.addr = tr.paddr;
endfunction
endclass
     
```

```verilog
class env extends uvm_env;
  top_reg regmodel;
  top_adapter adapter_inst;
  uvm_reg_predictor#(transaction) predictor_inst;

  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    regmodel = top_reg::type_id::create("regmodel");
    regmodel.build();
    //create adapter, and predictor
  endfunction

  function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    regmodel.default_map.set_sequencer( .sequencer(agent_inst.seqr), .adapter(adapter_inst) );
    regmodel.default_map.set_base_addr(0);
    
    predictor_inst.map       = regmodel.default_map;
    predictor_inst.adapter   = adapter_inst;
  endfunction
endclass
```

```verilog

 class ctrl_wr extends uvm_sequence;
  `uvm_object_utils(ctrl_wr)
  
   top_reg regmodel;
  
  function new (string name = "ctrl_wr"); 
    super.new(name);    
  endfunction
  
  task body;  
    uvm_status_e   status;
    bit [3:0] wdata; 
    for(int i = 0; i < 5 ; i++) begin
       wdata = $urandom();
       regmodel.cntrl_reg_inst.write(status, wdata);
    end
  endtask
endclass
```
