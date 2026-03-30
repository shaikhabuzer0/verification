# Systemverilog Assertions

```verilog
##0 OR |->               same cycle 
##1 OR |->##1 OR |=>     next cycle
##2 OR |->##2 OR |=>##1  after two cycles 

Timing window
a |-> ##[1:3]b; is same as below
a |-> ##1b or ##2b or ##3b

Repetition operator
$rose(a) |-> a[*3]  is same as $rose(a) |-> ##1a ##1a ##1a
i.e once a is asserted then it must be high for consecutive 3 cycles

a[*2:3] i.e a should be high for 2 to 3 cycles consecutively, which means a can be high for 2 cycles consecutively or a can be high for 3 cycles consecutively, which is same as below expression
(##1a ##1a) or (##1a ##1a ##1a) i.e a is high for consecutively 2 cycles or consecutively 3 cycles

a[*0:2] // a need not be high for even one cycle also or a should be high for 1 cycle or a should be high for 2 cycles consecutively
example: a ##1 b[*0:2] ##1 c same as below expression
a ##1 c or a ##1b ##1c or a ##1b ##1b ##1c
in above expression a##1c means b need to be high even for one cycle also OR
a ##1b ##1c b should be high for 1 cycle OR
a ##1b ##1b ##1c b should be high for consecutive 2 cycles

goto[->] vs non consecutive[=] operator
b[=2] ##1c // c can be high after 1 cycle or 2 cycle or at anytime after 1 cycle
b[->2] ##1c // c must be high exactly after 1 cycle

Note:
for delay operator variable is mentioned after the operator i.e $rose(en) |->##[2:4] data;
in question it will be mentioned within these many cycles, or in between 4 to 5 cycles then use delay operator 
for repetition operator variable is mentioned before the operator i.e data[*2:4] or data[=4] or data[->2:4]
in question it will tell like for 2 cycles it should be high for n or m cycles it should be high then use repetition operator 

a[*2] // a should go high for 2 consecutive cycles, what if it goes high for consecutive 3 cycles? still it will pass! because a becoming high for consecutive 2 cycles condition is already matched!
a[*2:3] // a should be high for consecutive 2 cycles or 3 cycles, what if it is high for 4 cycles? still it will pass!
To make it fail or restrict it to exact number of cycles you should add tail expression!
a[*2] ##1 !a // a should go high for 2 consecutive cycles after that it should be low in next cycle or else assertion will fail.

```
1. What are the regions where assertions gets executed
2. Drawback of assertions
3. What is the need of default clocking?
4. Where to declare clocking? inside sequence or inside property?
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
```verilog
req |=> ack      vs     $rose(req) |=> $rose(ack)
```
$rose is edge-sensitive: It returns true only if the signal transitioned from 0, X, or Z to 1 between the **previous clock** sample and the **current clock sample**.

Level Signal (signal) is level-sensitive: It returns true if the signal is 1 in the current clock sample, **regardless of its value in the previous clock cycle**
```verilog
Ex: if rst goes high then data must be 0  
rst |-> (data==0) //correct solution, till the time rst is held high, data must be zero, at every posedge it will create a different threads to check the data whether it is zero or not, suppose if rst is high for 10 cycles then 10 threads will get created to check data==0 or not in all 10 thread  
$rose(rst) |-> (data==0) //whenever rst goes high then same cycle data must be zero, then in next cycle data can be anything which is not correct behaviour. here thread will get created only when rst goes from low to high. if rst is going low to high only once then only one thread will get created.  

Ex: $rose is usefull if you want to check for 10 request whether you got 10 gnt or not?  
$rose(req) |->##1 $rose(gnt) //correct solution for 10 req it will check 10 gnt  
req |->##1 gnt // wrong solution. if req stays high for 10 cycles then also it will create 10 threads which leads to wrong behaviour  
```
**Note: Delay operator with infinite range is weak i.e it won't throw error if it fails, use strong keyword**  
```verilog
example: req |->##[1:$] ack; // when req is high then from next cycle to end of simulation ack should be high anytime in above example if ack is not becoming high for all time then we will not get error, to get error use strong keyword  
req |->strong(##[1:$]ack);  
```

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
54. if a is asserted then b can be high anytime after 3 clock cycles
55. if a is asserted then from next cycle b can be high anytime, once b asserted c can be asserted anytime
56. if signal start is asserted then from next cycle signal a stays high for 3 consecutive cycles and after one cycle signal stop should be high

