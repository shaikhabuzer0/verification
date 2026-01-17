## AXI Protocol

- [AXI Channels](#axi-channels)
  - [Features](#features)
  - [Handshake](#handshake)
  - [Transfer vs Transaction](#transfer-vs-transaction)
  - [Write Transaction](#write-transaction)
  - [Read Transaction](#read-transaction)
  - [Active / Outstanding Transaction](#active-transaction--outstanding-transaction)

- [AXI3 vs AXI4](#axi3--axi4)

- [Protection & Privilege (AxPROT)](#protection--privilege-level-support-axprot)

- [Cache](#cache)
  - [Read Allocate](#read-allocate)
  - [Write Allocate](#write-allocate)

- [Atomic Access (AxLOCK)](#atomic-access-axlock-avoids-memory-overwrite-problem)
  - [Locked Access](#locked-access)
  - [Exclusive Access](#exclusive-access)

- [4KB Boundary](#4kb-boundary)
  - [Constraint](#constraint-for-4kb-boundary)
- [End address calculation](#End-address-calculation)

## AXI Channels  
It's a point to point specification.  
5 Independant channels  
AW  Addr Write  --->     
W  Write data  --->   
B  Buffered  <---   
AR  Addr Read  --->   
R   Read data  <---  

- Each channel is unidirectional, hence separate write response i.e B channel is needed.  
![AXI Flow Diagram](https://shaikhabuzer0.github.io/verification/asset/axi_arch.jpg)

<p align="center">
  <img src="{{ site.baseurl }}/asset/axi_arch.jpg" width="600" alt="AXI Flow Diagram">
</p>

### Features  
- Separate Write and Read channels maximize the bandwidth, both can work in parallel  
- Multiple outstanding addresses i.e manager can issue transactions without waiting for earlier txns to complete. It enables parallel processing of txns hence improved performance  
- No timing relations between address and data operations i.e manager can issue AW(write address) txn but there is no restriction that when manager will provde W(write data) txn  
- Unalingned data transfer  
- Out of order txns completion i.e slave received multiple request with ID's. Slave can serve(give response back to master) any request out of order with ID's tagged  
- Burst txns i.e only single address is required i.e start address, then slave will calculate the next address based on burst  

#### Handshake  
  - Valid goes from source to destination  
  - Valid is sticky, must remain asswerted until destination accepts information

#### Transfer vs Transaction  
  - Transfer: Single exchange of information with one valid and ready handshake  
  - Transaction: Write transaction, Read transaction

#### Valid ready assertions  
  - Ready can be asserted at any time, before valid, after valid or at same time as valid  

#### Write transaction   
  - The master drives the wlast signal high to indicate final data  
  - slave either can monitor wlast or it can count all the transfered data using SIZE and LENGTH  
  - Once all the data is received then slave give a single BRESP for all the burst  
  - If there is any error in received data, slave has to wait for entire transfer to complete then inform master that error has occured   
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
0bit- Is it a data transfer or instruction transfer?**(instruction or data)**. If the transaction is mixed of instruction and data then it is treated as data only.  
1bit- Is it accessing secured memory region or non-secured memory region?**(secure/non-secure)**   
2bit- Is this a normal transaction or VIP transaction? **(privilege/non-privilege)**  

### Cache  
- How to handle the data? should it be stored in cache? or should it be stored inside buffer?
- When AXI master try to read the data from cache and the data is not present there then it is called read cache miss, similarly for write cache miss. Now the memory controller will decide whether should I bring back the data to cache or not? thats where we read allocate and write allocate comes in picture.
#### Read Allocate  
Meaning: If you read some data and it’s not in cache, you can allocate (create space and store it in cache).  
Why useful: Speeds up future reads, since the data will already be in cache next time.  
If disabled: Every read miss goes directly to memory, cache never stores it.

#### Write Allocate  
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
#### Out of order(transfer ID) only for completion transactions i.e BID and RID   
AWID WID(not in AXI4) BID ARID RID
Rules:  
  - All transfer must have ID
  - All transfers in xtn must have same ID
  - master can support multiple ID's for multple threads( i.e multiple transaction with different IDs)
#### interleaving vs Out of order?
Interleaving is nothing but mixing of beats of burst. If you mix the beats of burst you should be able to identify as well that which beat belongs where, this identification we can do if we have IDs.  

Interleaving is possible when you are doing AWID, WID, ARID transactions, all these transactions are initiated by master.  
Out of order is not related with beats, it's related with complete transaction response. If master has initiated multiple transaction without waiting for any of the response then slave can respond back to these transactions in any order, this is called out of order completion.  

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
AWLEN = 5  
AWBURST == INCR  
AWSTROBE = 20'hF_FFFE    

TOTAL DATA TRANSFER = SIZE * LEN = 4BYTES * 5 = 20BYTES out of which first byte is invalid represented by strobe  


### 4KB boundary  
For each slave 4kb of address space is allotted.  
0 to 4095KB --> slave1    
4096 to 8191 --> slave2    

So the idea is, whenever AXI is doing a write txn or read txn it should be within single slave address space, it should not cross specific slave address space  
i.e address offset + total transfer should not cross 4kb boundary  

address offset = AWADDR % 4096  
total transfer = 2^(AWSIZE) * (AWLEN + 1) 

Why AWADDR % 4096, why modulo operator is used?  
Let's do calculation without modulo operator  
AWADDR = 7045
AWSIZE = 2 -> 2^2=4bytes
AWLEN = 8
total transfer = 4 * 8 = 32bytes
4kb boundary = 7054 + 32bytes <= 4096  //crossing the boundary but actually this is wrong.  
Here if you see, we directly took the address without converting into 0 to 4096 range  
To convert the given address in 4kb address space we have to take modulo of the address  
Let's do it again  
7054 % 4096 = 2958  
4kb boundary = 2958 + 32bytes <= 4kb // this equation is not crossing 4kb boundary  

#### Confusion on AWLEN calculation  
AWLEN = no. of beats - 1  
no. of beats = AWLEN + 1 // we are interested in no. of beats to calculate total transfer  

Now let's take an example  
AWADDR = 1000  
AWSIZE = 2 -> 4bytes of one beat  
AWLEN = 3  
no. of beats = 3 + 1 = 4  

total transfer = 4 * 4 = 16bytes  
address 1000 -> 4bytes beat 0  
address 1004 -> 4bytes beat 1  
address 1008 -> 4bytes beat 2  
address 1012 -> 4bytes beat 3  

end address = AWADDR + 2^SIZE * (AWLEN)  
end address = 1000 + 4 * 3 = 1012  

#### Constraint for 4kb boundary  
address offset + total transfer <= 4kb  
awaddr % 4096 + (2^AWSIZE * (AWLEN + 1)) <= 4096  

### End address calculation  
end address = AWADDR + 2^(AWSIZE) * (AWLEN)  
Example:  
AWADDR = 100  
AWSIZE = 0  
AWLEN = 15  
end address = AWADDR + (2^AWSIZE * (AWLEN))  
            = 100 + 1 * 15 
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

## Driver code



## pieplined driver concept 

## Functional coverage  
1. What are the methods to sample the values for coverage
2. Why we need conditional coverage
3. what is option.per_instance and option.auto_bin_max
4. What is implicit, explicit, default, vector, scalar bins
5. What is procedural code and declarative code
6. How duplicate values are handled in case of vector bins
7. How to filter values of variable, example how do you create bins to cover only odd numbers, or even numbers etc?
8. Why do we need ignore bins, what is it's use case
9. What is illegal bins
10. How do you handle presence of X and Z values, explain with example
11. What is the purpose of weights of coverpoint
12. Explain cross coverage with different examples
13. How to filterout specific combinations from cross coverage?
14. What is transitions bins, and its representation
15. Write coverpoints for full adder
16. Cover all values of a random variable between 0 and 100 rand bit[6:0] data;
17. Cover even and odd multiples of 10 within 0 to 100 rand bit[6:0] data;
18. Cover all combinations of signal data and addr which is 1bit, i.e rand bit addr, data;
19. Cover binary vector values with an even number of 1's, rand bit[3:0] data;
20. Cover different length of burst transactions of AXI, rand bit[7:0] awlen; rand bit[1:0]awburst;

## Assertion  
1. What are the regions where assertions gets executed
2. Drawback of assertions
3. What is the need of default clocking?
4. Where to declare clocking? inside sequence or inside property?
5. 
6. Difference between SIA, DIA, Concurrent Assertions, Why concurrent assertions are preferred over other methods?
7. What are the layers in concurrent assertions? Explain in detail.
8. What is single thread and multiple threads in assertion? which blocks to use for this behaviour?
9. list down the edge sample functions and explain them
10. $rose for single bit and multi bit variable
11. Explain $past with all arguments
12. WAST if a asserts, b must assert in next clock cycle
13. WAST each request must be followed by ack
14. WAST if rst deassert, CE must assert in same clock tick
15. WAST wr_req must be followed by rd_req
16. WAST current value of addr must be one greater than previous value if start asserts.
17. WAST if rst deassert, dout mst be zero
18. If load_in deassert, dout must be equal to load value
19. If rst deassert, output of the shift register must be shifted to left by one in the next clock tick
20. if rst deassert, current value and past value of the signal(a) differ only in signle bit
21. In DFF, output must remain constant if CE is low
22. In TFF if CE assert, output must toggle
23. What is the difference between following two expressions?  
req |=> ack      vs     $rose(req) |=> $rose(ack)
$rose is edge-sensitive: It returns true only if the signal transitioned from 0, X, or Z to 1 between the **previous clock** sample and the **current clock sample**.

Level Signal (signal) is level-sensitive: It returns true if the signal is 1 in the current clock sample, **regardless of its value in the previous clock cycle**
**Ex:** if rst goes high then data must be 0  
rst |-> (data==0) //correct solution, till the time rst is held high, data must be zero, at every posedge it will create a different threads to check the data whether it is zero or not, suppose if rst is high for 10 cycles then 10 threads will get created to check data==0 or not in all 10 thread  
$rose(rst) |-> (data==0) //whenever rst goes high then same cycle data must be zero, then in next cycle data can be anything which is not correct behaviour. here thread will get created only when rst goes from low to high. if rst is going low to high only once then only one thread will get created.  
**Ex:** $rose is usefull if you want to check for 10 request whether you got 10 gnt or not?  
$rose(req) |->##1 $rose(gnt) //correct solution for 10 req it will check 10 gnt  
req |->##1 gnt // wrong solution. if req stays high for 10 cycles then also it will create 10 threads which leads to wrong behaviour  

**Note: Delay operator with infinite range is weak i.e it won't throw error if it fails, use strong keyword**  
example: req |->##[1:$] ack; // when req is high then from next cycle to end of simulation ack should be high anytime in above example if ack is not becoming high for all time then we will not get error, to get error use strong keyword  
req |->strong(##[1:$]ack);  

25. List down all repetition operators
26. When wr is asserted then after 2 clock tick rd should go high for consecutive 2 tick24.
27. Explain detail below assertion, what is importance of tail expression min and max values  
$rose(rd) |-> rd[*2:5]##1 !rd   
28. Are non consecutive operator weak in nature?
29. Explain a[=1:4] ##1 b;
30. if a deassert then it should deassert twice followed by a assert
31. a deassert twice NC then in next cycle a must asssert
32. wr_req must be followed by rd_req, if rd_req do not assert before timeout then system should reset
33. wr must be followd by rd in next clock tick
34. if a assert, b must assert in 5 clock ticks
35. if rst deassert then CE must assert within 1 to 3 clock ticks
36. if req assert and ack not received in three clock tick then req must reassert
37. if a assert, a must remain high for 3 clock
38. system operation must start with rst asserted for 3 cons clock
39. CE must assert somewhere during simulation if reset deassert
40. txn start with CE become high and end with CE becomes low each txn must contain atleast once read and write req
41. if CE assert somewhere after rst deassert then we must receive atleast one write request
42. a must assert twice during simulation
43. if a become high somewhere then b mus become high in the immediate next clock tick
44. if req is received and all the data is sent to slave indicated by done signal then ready must be high in the next clock
45. if b deassert then b must remain false for 3 cons ticks
46. if a assert then a should be followed by c within 1 to 4 clock
47. if a become true in current clock then c must become true within 1 to 4 clock followed by b becoming true in next clock
48. if CE assert then it must deassert within 5 to 10 clock cycles
49. if CE assert it must remain stable for 7 cons cycle
50. if rst deassert and CE assert then read must stay high for 2 clock when user receive rd_req
51. if rd assert then addr must remain stable for 3 cons clock 
52. Write assertion such that req must be followed by ack in next clock cycle
53. Write assertion such that read and write request must not occur at same time

