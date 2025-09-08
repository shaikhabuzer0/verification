## AXI Protocol Concepts  
It's a point to point specification.  
5 Independant channels  
AW  Addr Write  --->     
W  Write data  --->   
B  Buffered  <---   
AR  Addr Read  --->   
R   Read data  <---  

- Each channel is unidirectional, hence separate write response i.e B channel is needed.  

Features  
- Separate Write and Read channels maximize the bandwidth, both can work in parallel
- Multiple outstanding addresses i.e manager can issue transactions without waiting for earlier txns to complete. It enables parallel processing of txns hence improved performance.
- No timing relations between address and data operations i.e manager can issue AW(write address) txn but there is no restriction that when manager will provde W(write data) txn
- Unalingned data transfer
- Out of order txns completion i.e slave received multiple request with ID's. Slave can serve(give response back to master) any request out of order with ID's tagged
- Burst txns i.e only single address is required i.e start address, then slave will calculate the next address based on burst
- 
