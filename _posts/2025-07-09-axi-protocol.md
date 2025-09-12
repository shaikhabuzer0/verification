## AXI Protocol Concepts  
It's a point to point specification.  
5 Independant channels  
AW  Addr Write  --->     
W  Write data  --->   
B  Buffered  <---   
AR  Addr Read  --->   
R   Read data  <---  

- Each channel is unidirectional, hence separate write response i.e B channel is needed.  

### Features  
- Separate Write and Read channels maximize the bandwidth, both can work in parallel
- Multiple outstanding addresses i.e manager can issue transactions without waiting for earlier txns to complete. It enables parallel processing of txns hence improved performance.
- No timing relations between address and data operations i.e manager can issue AW(write address) txn but there is no restriction that when manager will provde W(write data) txn
- Unalingned data transfer
- Out of order txns completion i.e slave received multiple request with ID's. Slave can serve(give response back to master) any request out of order with ID's tagged
- Burst txns i.e only single address is required i.e start address, then slave will calculate the next address based on burst
  ### Handshake
  - Valid goes from source to destination
  - Valid is sticky, must remain asswerted until destination accepts information
  ### Transfer vs Transaction  
  Transfer: Single exchange of information with one valid and ready handshake
  Transaction: Write transaction, Read transaction
  ### Valid ready assertions
  Ready can be asserted at any time, before valid, after valid or at same time as valid

  ### Write transaction   
  - The master drives the wlast signal high to indicate final data\
      - slave either can monitor wlast or it can count all the transfered data using SIZE and LENGTH\
      - Once all the data is received then slave give a single BRESP for all the burst\
      - If there is any error in received data, slave has to wait for entire transfer to complete then inform master that error has occured\
  ### Read transaction  
  - For read transaction there is RRESP for every transfer in transaction because in write transaction slave has to send BRESP as separate transfer on separate channel i.e response channel. But in case of read transaction we don't have separate channel for response, instead slave uses the same channel i.e RDATA to send the response  
  - If an error occured, slave will respond with error message but still slave has to wait for burst to complete i.e NO EARLY BURST TERMINATION  
  ### Active transaction / Outstanding transaction  
  - READ: AR has been transfered but last read data has not been transfered i.e it is still pending(outstanding/active)  
  - WRITE: AW has been transfered but response i.e BRESP has not yet been transfered(outstanding/active)  
    NOTE:\
    - In read transaction address must come before data.  
    - In write transaction data can come after address, leading write data is also allowed

| **AXI3**        | **AXI4**                                                                 |
|------------------|--------------------------------------------------------------------------|
| AxLEN\[3:0\] (2^4 = 16bytes)    | AxLEN\[7:0\] (2^8 = 256bytes)                                                 |
| AxLOCK\[1:0\]    | AxLOCK i.e there is no lock support, only exclusive access support is given                                                                  |
| WID\[x:0\]       | AWQOS\[3:0\], AWREGION\[3:0\], AWUSER\[x:0\], WUSER\[x:0\], BUSER\[x:0\] |

### Protection & Privilege Level Support AxPROT\[2:0\](for Processors)
0bit- Is it a data transfer or instruction transfer?(instruction or data). If the transaction is mixed of instruction and data then it is treated as data only.
1bit- Is it accessing secured memory region or non-secured memory region?(secure/non-secure)  
2bit- Is this a normal transaction or VIP transaction? (privilege/non-privilege)  

### Cache  


