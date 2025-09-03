## UVM RAL Model Concepts

Desired value: the value which you want to write into the hardware register
Mirrored value: the value which is actually present inside the hardware register, this value gets updated after each write/read txn

example:  
initially both values are 0's  
desired=0, dut=0, mirrored=0  
Now I want to write 2 into dut, first I have to call set method  
set(2)  
desired=2, dut=0, mirrored=0  
update(2) //after calling update, we are writing value 2 into dut register  
desired=2, dut=2, mirrored=2  

To configure the desired value we have two methods.  
set() and randomize()  

Write vs update method.  
upate method will first check whether desired and mirrored values are different? if they are different then only it will initiate the write transaction(it will write the desired value which is configured by set/randomize method), otherwise if desired and mirrored values are same then it won't initiate any write txn.  
While write txn simply write the desired value by initiating write txn.(there are no checks)  

We have three types of predictors.  
1. Implicit/auto predictor --> defalut_map.set_auto_predict(1)
2. Explicit predictor --> contains adapter and map --> useful if you want to implement functional coverage
uvm_reg_predictor urp;  
urp.map = map;  
urp.adapter = adapter_inst;  
3. Passive predictor  

predict() --> updates the desired and mirrored value of a register in RAL model. No txn is performed on hardware registers  
mirror() it internally calls predict() --> Read the hardware register and upate/check its existing mirror value.    

example:  
write(2) // desired=2 mirrored=2 dut=2  
predict(3) // desire=3, mirrored=3, dut=2  
mirror() // dut=2 != mirrored=3 throws error if UVM_CHECK is enable  


