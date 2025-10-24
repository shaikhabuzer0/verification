## AXI Protocol Concepts  
It's a point to point specification.  
5 Independant channels  
AW  Addr Write  --->     
W  Write data  --->   
B  Buffered  <---   
AR  Addr Read  --->   
R   Read data  <---  

- Each channel is unidirectional, hence separate write response i.e B channel is needed.  
![AXI Flow Diagram](/asset/axi_arch.jpg)
### Features  
- Separate Write and Read channels maximize the bandwidth, both can work in parallel
- Multiple outstanding addresses i.e manager can issue transactions without waiting for earlier txns to complete. It enables parallel processing of txns hence improved performance.
- No timing relations between address and data operations i.e manager can issue AW(write address) txn but there is no restriction that when manager will provde W(write data) txn
- Unalingned data transfer
- Out of order txns completion i.e slave received multiple request with ID's. Slave can serve(give response back to master) any request out of order with ID's tagged
- Burst txns i.e only single address is required i.e start address, then slave will calculate the next address based on burst
  #### Handshake
  - Valid goes from source to destination
  - Valid is sticky, must remain asswerted until destination accepts information
  #### Transfer vs Transaction  
  Transfer: Single exchange of information with one valid and ready handshake
  Transaction: Write transaction, Read transaction
  #### Valid ready assertions
  Ready can be asserted at any time, before valid, after valid or at same time as valid

  #### Write transaction   
  - The master drives the wlast signal high to indicate final data\
      - slave either can monitor wlast or it can count all the transfered data using SIZE and LENGTH\
      - Once all the data is received then slave give a single BRESP for all the burst\
      - If there is any error in received data, slave has to wait for entire transfer to complete then inform master that error has occured\
  #### Read transaction  
  - For read transaction there is RRESP for every transfer in transaction because in write transaction slave has to send BRESP as separate transfer on separate channel i.e response channel. But in case of read transaction we don't have separate channel for response, instead slave uses the same channel i.e RDATA to send the response  
  - If an error occured, slave will respond with error message but still slave has to wait for burst to complete i.e NO EARLY BURST TERMINATION  
  #### Active transaction / Outstanding transaction  
  - READ: AR has been transfered but last read data has not been transfered i.e it is still pending(outstanding/active)  
  - WRITE: AW has been transfered but response i.e BRESP has not yet been transfered(outstanding/active)  
    NOTE:  
    - In read transaction address must come before data.  
    - In write transaction data can come after address, leading write data is also allowed

| **AXI3**        | **AXI4**                                                                 |
|------------------|--------------------------------------------------------------------------|
| AxLEN\[3:0\] (2^4 = 16bytes)    | AxLEN\[7:0\] (2^8 = 256bytes)                                                 |
| AxLOCK\[1:0\]    | AxLOCK i.e there is no lock support, only exclusive access support is given                                                                  |
| WID\[x:0\]       | AWQOS\[3:0\], AWREGION\[3:0\], AWUSER\[x:0\], WUSER\[x:0\], BUSER\[x:0\] |

### Protection & Privilege Level Support AxPROT\[2:0\]
0bit- Is it a data transfer or instruction transfer?(instruction or data). If the transaction is mixed of instruction and data then it is treated as data only.
1bit- Is it accessing secured memory region or non-secured memory region?(secure/non-secure)  
2bit- Is this a normal transaction or VIP transaction? (privilege/non-privilege)  

### Cache  
- How to handle the data? should it be stored in cache? or should it be stored inside buffer?
- When AXI master try to read the data from cache and the data is not present there then it is called read cache miss, similarly for write cache miss. Now the memory controller will decide whether should I bring back the data to cache or not? thats where we read allocate and write allocate comes in picture.
- Read Allocate  
Meaning: If you read some data and it’s not in cache, you can allocate (create space and store it in cache).  
Why useful: Speeds up future reads, since the data will already be in cache next time.  
If disabled: Every read miss goes directly to memory, cache never stores it.

- Write Allocate  
Meaning: If you write to an address not in cache, should we allocate cache space for it?  
Why useful: If you’ll read that data later, keeping it in cache saves time.  
If disabled: The write goes straight to memory and doesn’t fill the cache.

Read Allocate → "If I fetch from memory, should I also keep it in cache?"  
Write Allocate → "If I write something new, should I also keep it in cache?"  

### Response signaling   
00-Okay Normal access success OR Exclusive access failure  
01-Ex-Okay  
10-SLVERR  
  - when there is unsupported transfer size is attempted  
  - when you write on read only space  
11-DECERR generated by interconnect  
  - When the requrested address is does not exist i.e it tells the given address does not belong to any of the slaves available  

NOTE: We don't have read storbe signal, we have only write strobe signal.  
      WSTRB can change between transfers in transaction  

### Atomic Access AxLOCK (Avoids memory overwrite problem) 
#### Locked Access 
locks the channel, remains locked until unlocked transfer is generated.  
When its lock transaction then interconnect must ensure only that master can access the targeted slave region until unlocked from same master completed  
#### Exclusive access(sharing a slave memory space)    
Multiple masters accessing single slave but different locations  
Helpful in read modify write operations.  
Example.  location 0x04 value 1234
  Suppose master M1 performs an ex-read on address location 0x04, at some later point time, M1 attempts to complete the exclusive operation by performing ex-write to the same address location.  
  The exclusive write of the M1 is signalled as successfull if no other master i.e M2 has written to that location between ex-read and ex-write of M1. i.e M1 will get response as ex-okay(successful exclusive access)
  The exclusive write of the M1 is signalled as failed if another master i.e M2 has written to that location between ex-read and ex-write of M1, in this case M1 will get Okay response(error).  

Scenario 1:  
  M1-> ex-read @0x04  
  M1-> ex-write @0x04 -> response ex-okay(successfull)  
  
Scenario 2:  
  M1-> ex-read @0x04  
  M2-> ex-write @0x04 -> response ex-okay  
  M1-> ex-write @0x04 -> response okay(error) and whatever M1 is trying to write won't get upated in memory  

### QoS AxQOS (Quality of service)(prioritize transactions)  
  x0 -> lowest priority  
  xf -> highest priority  
Sometimes CPU require memory access that are more important thatn GPU or any other system

### AXI channel dependancies
- wlast must complete before bvalid is asserted
  - Master must send all data before a write response can by seen by master
  - Note: in AXI3 address does not have to be seen before write response is sent i.e response can come first then address we can send, In AXI4 all data and address must be transfer before master can see a write response
- Rvalid cannot be asserted until ARADDR has been transfer(slave cannot transfer any rdata without seeing the address first(read xtn)
- wvalid can assert before AWVALID i.e master can write data to slave before sending the address to slave

### Transfer behaviour and Transaction ordering
#### Out of order(transfer ID)
AWID WID(not in AXI4) BID ARID RID
Rules:  
  - All transfer must have ID
  - All transfers in xtn must have same ID
  - master can support multiple ID's for multple threads( i.e multiple transaction with different IDs)
#### Ordering rule for write xtn(interleaving is not supported on WDATA)
- wdata must follow the same order as the addr transfer on AW channel
- i.e if master issues address A then B, so data must start with A0A1AL before B0B1BL
- Note: In AXI4 we don't have WID to keep track of xtn hanece interleaving for write xtn is not supported
- A0B0B1A1ALBL not supported in AXI4(write data interleaving)

- Transaction with **different** ID's can complete in any order i.e out of order completion i.e response can come in any order
- i.e write data order (A0A1AL, id0) (B0B1BL, id1)
- write response order (B, id1) (A, id0) i.e first response is recived of B xtn then A xtn
- Transaction with **same** IDs must complete in same order
- i.e write data order (A0A1AL, id0) (B0B1BL, id1) (C0CL, id0)
- completion order (B, id1) (A, id0) (C, id0) here A and C are having same IDs and same completion order

#### Ordering rules for read xtn(interleaving is supported on rdata)
- rdata for **different ID's** on R channel has no ordering restrictions i.e slave can send it in any order
- ex:
AWID 0 1  
AR   A B  
RID  1  
R    BL  
In above example txn B is serviced first before A, even though the address for txn A is received first  
ex:  
ARID 0 1  
AR   A B  
RID  1 0 0 1 1 0 0  
R    B A A B
---------------------------
(B0, ID1) (A0A0, ID0) (B1BL, ID1) (A2AL, ID0)  
For txn with **same** IDs, rdata on R channel must be returned in the order that they were requested  

### Unaligned transfer (start address)
Note: There won't be any data loss in unaligned txn   
Unaligned address only affects the first transfer in txn, all other transfers are aligned  
ex:  
AWADDR = 1   
AWSIZE = 2 == 4BYTES  
AWLENG = 5  
AWBURST == INCR  
AWSTROBE = 20'hF_FFFE    

TOTAL DATA TRANSFER = SIZE * LEN = 4BYTES * 5 = 20BYTES out of which first byte is invalid represented by strobe  


### 4KB boundary  
For each slave 4kb of address space is allotted.  
0 to 2095KB is allocated to slave1    
4096 to 8191 is second slave address space  

So the idea is, whenever AXI is doing a write txn or read txn it should be within single slave address space, it should not cross specific slave address space  
i.e address offset + total transfer should not cross 4kb boundary  

address offset = AWADDR % 4096  
total transfer = 2^(AWSIZE) * (AWLEN + 1) 

#### Constraint for 4kb boundary  
address offset + total transfer <= 4kb  
awaddr % 4096 + (2^size * (awlen + 1)) <= 4096  

### End address calculation for burst 
end address = AWADDR + 2^(AWSIZE) * (AWLEN - 1)
Example:  
AWADDR = 100  
AWSIZE = 0  
AWLEN = 15  
end address = AWADDR + (2^AWSIZE * (AWLEN -1))  
            = 100 + 1 * 16 -1  
            = 115  
            

### Wrap calculation  
wrap boundary = int(start addr/total transfer) * total transfer  
### End address calculation for wrap  
end address = wrap boundary + total transfer  
Example:  
AWADDR = 4  
AWSIZE = 2  
AWLEN = 3  
AWBURST = WRAP  
end address = wrap boundary + total transfer  
wrap boundary = int(start addr/total transfer) * total transfer  
              = (4/16) * 16  
              = 0 * 16
              = 0
end address   = 0 + 16
              = 16  
transfer starts from address 4 till 16 then roll back to 4th address i.e wrap  
### strobe calculation
no. of strobe bits = no. of data bits / 8  
no. of 1's = 2^size  
ex: 2^2 = 4bytes = strobe needs 4bits to represent 4bytes, each bit represents 1byte  
Constraint for strobe  
if(awsize == 0) $counones(wstrobe) == 1  
if(awsize == 1) $counones(wstrobe) == 2  
if(awsize == 2) $counones(wstrobe) == 4  



Interleaving concept is related with burst wdata and rdata  
Out of order(outstanding) txn concept is related with bresp and rresp  
