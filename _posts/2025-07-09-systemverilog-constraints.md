## SystemVerilog Constraints Questions

Q1. Write a constraint to generate 01010101... pattern.

```verilog
class pattern_gen;

//Class members
rand int a[]; //dynamic array, each element is of 32bits. as our pattern is either 0 or 1 we can declare it as bit also.

constraint a_size{ a.size == 10;} //size of the array will be 10, if you randomize one time, 10 random values will get stored in this array.

//Instead of random 10 numbers we want some pattern as described above, hence writing below constraint.
 //METHOD-1
constraint patt{
  foreach(a[i])
    if(i%2 == 0) //even location fill 0's
      a[i] == 0;
    else
      a[i] == 1; //odd locations fill 1's
  }

//METHOD-2 
constraint patt1{
  foreach(a[i])
    a[i] == i[0]; // i variable starts from 0 to 9 as we have restricted size to 10. so taking the last bit of i variable.
}
//var,decimal,binary
//i=0=00[0]
//i=1=00[1]
//i=2=01[0]
//i=3=01[1]
// and so on...

function void post_randomize();
  $display("Randomized data is %p", a);
endfunction
endclass

module top;

pattern_gen p; //class handle declaration

initial begin
p=new(); //class object creation

if(!p.randomize())
  $error("Randomization Failed");

end
endmodule

```
Q2- Generate pattern 1234554321

Here we can declare one array of size 10.
```verilog
class pattern_gen;
 rand int a[];
constraint a_size{ a.size == 10;}
constraint patt{
 foreach(a[i])
 if(i<5) 
  a[i] == i+1; //till i becomes 4 store i+1 value
 else
  a[i] == 10-i; //10-5=5, 10-6=4, 10-7=3, 10-8=2, 10-9=1
}
function void post_randomize();
  $display("Randomized data is %p", a);
endfunction
endclass
module top;

pattern_gen p; //class handle declaration

initial begin
p=new(); //class object creation

if(!p.randomize())
  $error("Randomization Failed");

end
endmodule
```
Q3- Generate pattern 1122334455667788

METHOD-1
If you divide integer by integer you will get integer value. i.e 2/2=1 and 3/2=1, 4/2=2 and 5/2=2
so the formula is (i+2)/2

METHOD-2
Consider the below 3bit binary numbers, if we ignore the LSB bit then we will get the desired pattern.  
00 0  
00 1  
01 0  
01 1  
10 0  
10 1  
11 0  
11 1  

```verilog
class pattern_gen;
 rand int a[];
constraint a_size{ a.size == 16;}

//METHOD-1
constraint patt{
 foreach(a[i])
  a[i] == (i+2)/2; 
}
function void post_randomize();
  $display("Randomized data is %p", a);
endfunction

//METOHD-2 (hack, in post randomized we have trimed two indices values, and started printing from index 2) 
constraint patt{
 foreach(a[i])
   if(i>1)
     a[i] == i[4:1]; 
}
function void post_randomize();
 foreach(a[i])begin
if(i>1)
  $display("Randomized data is %d", a[i]);
end
endfunction
endclass
module top;

pattern_gen p; //class handle declaration

initial begin
p=new(); //class object creation

if(!p.randomize())
  $error("Randomization Failed");

end
endmodule
```
Q4- Reverse the bits of an array
METHOD-1
We can do it using foreach loop.
METHOD-2
We can do this by using stream operator i.e {<<{variable}}
```verilog
module top;
bit[7:0] variable=8'b1100_1001;
bit[7:0] reversed; //expected output is 1001_0011
initial begin
for(int i=0; i<8; i++)begin
 reversed[i] = variable[7-i];
end

//METHOD-2
reversed = {<<{variable}}; //that's it.
end
endmodule
```
NOTE:
//pre_randomize is to perform constraint overriding if needed  
//post_randomize is for modifying randomization results  
//in constraint you can't use begin end block to write multiple lines, you  
//have to use curly braces instead of begin end  
//$onehot, $countbits, $countones  

Q5. Generate consecutive numbers
```verilog
module test;
    int result=1;
class cons_problems;
    rand int array[];
    int dummy[$];
    constraint size_c { array.size == 50;}
    //constraint size_d { dummy.size == 50;}

    constraint values_c{
        array[i]== i;
        }
```
Q6. Generate pattern 1234554321

```verilog
    constraint value_c {
        foreach(array[i])
            if(i < 5)
                array[i] == i+1; // adding 1 because our pattern starts from 1 not from 0
            else 
                array[i] == 10 - i;

        }
```

Q7. Generate 9 19 29 39 49 59 ...

 ```verilog
    constraint value_c{
            foreach(array[i])
                array[i] == i*10+9;
    }
```
Q8. 5 -10 15 -20 25 -30

```verilog
    constraint value_c{
        foreach(array[i])
            if(i%2==0)
                //array[i] == -5*i; 
                array[i] == (5*i)+5;//adding 1 as it is starting from 5 not from 0
            else
                //array[i] == -5*i;//adding 1 as it is starting from 5 not from 0
                array[i] == -5*(i+1);//adding 1 as it is starting from 5 not from 0
            }
```

Q9. 1122334455...

```verilog
    constraint value_c{
        foreach(array[i])
            array[i] == ((i+2)/2);
        }
```

Q10. 1.35 and 2.57
```verilog
    real array_r[10];
    constraint value_c{
        foreach(array[i])
            array[i] inside {[135:257]};
        }

        function void post_randomize();
            foreach(array[i])begin
                array_r[i] = (array[i]/100.0);
            end
            $display("----------------------------------------------------------------------------");
            $display("Value after randomization is %p", array_r);
            $display("----------------------------------------------------------------------------");
        endfunction

/* //for below code you have to randomize class multiple times to get the  results
rand int a;
real b;

    constraint value_c{
        a inside {[135:257]};
        }
    function void post_randomize();
        b = a/100.0;
        $display("----------------------------------------------------------------------------");
        $display("Value after randomization is %f", b);
        $display("----------------------------------------------------------------------------");
    endfunction
```

Q11. 010203040506...

    constraint value_c{
        foreach(array[i])
            if(i%2==0)
                array[i] == 0;
            else
                array[i] == (i+2)/2;
            }
```

Q12. 25 27 30 36 40 45

```verilog
// to generate exact sequence you can use array and post randomize function
 int result[6];
 int counter=0;

    constraint value_c{
        foreach(array[i])
            array[i] == i;
        }

function void post_randomize();
    foreach(array[i])begin
        if(i > 24 && i < 46 && i!=35)begin
            if(i%5==0 || i%9==0)begin
                result[counter] = i;
                counter++;
            end

        end
    end
    $display("----------------------------------------------------------------------------");
    $display("Value after randomization is %p", result);
    $display("----------------------------------------------------------------------------");
endfunction
```

Q13. 25 27 30 36 40 45

```verilog
// this problem can be solved without using array, you have to randomize the class multiple times
// there is no gurantee that it will follow the exact sequence

rand int data;
    constraint value_c{
        data > 24 && data < 46;
        }
    constraint value_d{
        data%5 == 0 || data%9 == 0;
        data!=35;
        }
    function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("Value after randomization is %p", data);
        $display("----------------------------------------------------------------------------");
    endfunction
```

Q14. random even number between 50 100 wihout 

```verilog
    constraint value_c{
        foreach(array[i])
            array[i] inside {[50:100]};
        }
    constraint value_d{
        foreach(array[i])
            array[i]%2==0;
        }
```

Q15. in 32bit variable, number of 1's should be 12 non consecutively

```verilog
//print vriable in binary format
   rand bit[31:0] a;
    constraint value_c{
        $countones(a) == 12;
        }
    constraint value_n{
        foreach(a[i])
            if(i>0 && a[i])
                a[i] != a[i-1];
            }
                
    function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("Value after randomization is %b", a);
        $display("----------------------------------------------------------------------------");
    endfunction
```

Q. there is 1000bit variable, number of 7 consecutive 1's only and remaining all zeros  

```verilog

module test;
class constraints;
rand bit[999:0] data;
rand int start_index;
constraint seven_ones_c{
 $countones(data)  == 7;
}
/*
constraint ones_c{
 foreach(data[i]){
 if(i>0 && data[i] ==1)
  data[data[i]+:7] == 7'b1111111;
}
;}
/*
constraint index_c{
 start_index inside {[0:999-7]};
}
constraint final_c{
 data[start_index+:7] == 7'b1111111;
}
function void post_randomize();
  $display("data=%b", data);
endfunction
endclass
constraints c;
initial begin
  c=new();
  c.randomize();
end
endmodule

```
Q16. factorial numbers

```verilog
    //constraint size_c{
    //    array.size == 5;
    //    }
    constraint value_c{
        foreach(array[i])
            array[i] == factorial((i+1)*2);
        }
    function int fact(int num);
        result=1;
        if(num == 0)begin
            return 1;// 0! is 1
        end
        for(int i=1; i<=num; i++)begin
            result = result * i;
        end
        fact = result;
        $display("number=%d, fact=%d", num, fact);
    endfunction
    //using recursion function
    function int factorial(int num);
        if(num==0)begin
            return 1;
        end else begin
            factorial = num * factorial(num -1);
        end
    endfunction
```

Q17. odd numbers at even location and even numbers at odd locations

```verilog
    //constraint values_c{
    //    foreach(array[i])
    //        if(i%2==0)
    //            array[i] == i+1;
    //        else
    //            array[i] == i+1;
    //}
    constraint values_c{
        foreach(array[i])
            if(i%2==0)
                array[i]%2==1;
            else
                array[i]%2==0;
            }
```

Q18. randomize only 12th bit out of 32bits rest of the bits should be 0s

```verilog
rand bit[31:0] variable;
    constraint values_c{
        foreach(variable[i])
            if(i==12)
                variable[i] inside {0,1};
            else
                variable[i]==0;
        }
    function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("Value after randomization is %b", variable);
        $display("----------------------------------------------------------------------------");
    endfunction
```
// Q14 visit again

/*
//bit[7:0] array1 [10] with unique values and multiple of 3

rand bit[7:0] array1[10];
constraint values_c{
    unique {array1};
    }
constraint values_d{
    foreach(array1[i])
        array1[i] % 3 == 0;
    }
function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("Value after randomization is %p", array1);
        $display("----------------------------------------------------------------------------");
endfunction
*/

/*
//generate unique numbers without using unique keyword
//one way is to use shuffle in post_randomize function

//    constraint values_c{
//        foreach(array[i])
//            array[i] == i;
//        }
//        OR
//
    constraint values_e{
        foreach(array[i])
            array[i] inside {[1:100]};
        }

    constraint values_d{
        foreach(array[i])
            foreach(array[j])
                if(i!=j)
                    array[i]!=array[j];
                }
*/

/*
//prime number checking
rand int num;
int flag=0;
constraint value_c{
    num inside {[1:100]};
    }
function int prime(int num);
    if(num == 0 || num == 1)begin
        $display("0 and 1 are not prime numbers");
        return 0;
    end
    for(int i=2; i<num; i++)begin
        if(num%i==0)begin
            $display("The given number=%0d is not a prime number", num);
            flag = 1;
            return 0; //if its not prime number then return 0 value
        end
    end
  if(flag == 0)begin
   $display("----------------------------------------------------------------------------");
   $display("The given number=%0d is prime number", num);
   $display("----------------------------------------------------------------------------");
  end
endfunction


function void post_randomize();
        result = prime(num);
endfunction
*/

function int prime(int num);
    if(num == 0 || num == 1)begin
        $display("0 and 1 are not prime numbers");
        return 3;
    end
    for(int i=2; i<num; i++)begin
        if(num%i==0)begin
            //$display("The given number=%d is not a prime number", num);
            flag = 1;
            return 3; //if its not prime number then return 0 value
        end
  if(flag == 0)begin
   $display("----------------------------------------------------------------------------");
   $display("The given number=%0d is prime number", num);
   $display("----------------------------------------------------------------------------");
   return num;
  end
    end
endfunction

/*
//Prime numbers generation
    constraint valued_cx{
        dummy.size == 50;
        }

    constraint value_c{
        foreach(array[i])
            array[i] inside {[1:100]};
        }

    constraint valued_d{
        foreach(array[i])
            dummy[i] == prime(array[i]);
        }
    function void post_randomize();
        //foreach(array[i])
        //    array[i] = prime(array[i]);
        $display("----------------------------------------------------------------------------");
        $display("Prime number is %p", array);
        $display("----------------------------------------------------------------------------");
    endfunction
*/

/*

// 0-31 bits shoule be 1, 32-61 bits should be 0s

rand bit[0:61] addr;
//constraint values_c{
//    addr[0:31] == 'hFFFF_FFFF;
//    addr[32:61] == 'h0;
//    }
//-------- OR--------
constraint values_c{
    foreach(addr[i])
        if(i<32)
            addr[i] == 1;
        else
            addr[i] == 0;
        }

function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("Final value is %b", addr);
        $display("----------------------------------------------------------------------------");
endfunction

*/

/*
//generate consecutive elements
    constraint values_c{
        array[i]== i;
        }
*/


/*

// generate 10 unique numbers between 99 to 100 
    rand int foo[10];
    real store[10]; //point to be noted, its real data type
    constraint values_u{
        unique {foo};
        }
    constraint values_c{
        foreach(foo[i])
            foo[i] inside {[990:1000]};
        }

function void post_randomize();
    foreach(foo[i])
        store[i] = foo[i]/10.0;
        $display("----------------------------------------------------------------------------");
        $display("Final value is %p", store);
        $display("----------------------------------------------------------------------------");
endfunction
*/

    constraint asscending_c;
    constraint range_c;
    constraint data_c;

/*
// extern constraint
    rand int da[];
function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("Final value is %p", da);
        $display("----------------------------------------------------------------------------");
endfunction
*/

/*
//even numbers in range of 10 to 30 using fixed, dynamic, queue arrays

rand int fa[10];  //fixed
rand int da[];  //dynamic
rand int qa[$];  //queue // solution is eaxactly same as da


constraint fa_c{
    foreach(fa[i])
        fa[i] inside {[10:30]};
    }
constraint e_c{
    foreach(fa[i])
        fa[i]%2==0;
    }
constraint da_c{
    da.size == 10;
    }
constraint dav_c{
    foreach(da[i])
        da[i]%2==0;
    }

function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("Fixed array is %p", fa);
        $display("dynamic array is %p", da);
        $display("----------------------------------------------------------------------------");
endfunction
*/


/*
//write constraint so that if we randomize it for 10 times then we should get
//sequence as 1010101010 you should not use arrays

rand bit seq;
bit a=0;

constraint values_c{
    seq!=a;
    }
function void post_randomize();// $write won't add new line character so 10 10 will appear as 1010
        $write("Final value is %b, %b", seq, a);
endfunction
*/

/*
//write solve before constraint

rand bit a; //can take values 0 or 1
rand bit[2:0] b; // can take values 0, 1, 2, 3

constraint solv_a{
    (a==0) -> (b==1); //iiff a==0 then b must be 1, if a is not 0 then be can be 0,1,2,3
    solve a before b;
    }
function void post_randomize();
        $display("----------------------------------------------------------------------------");
        $display("value of a=%d, and b=%d", a, b);
        $display("----------------------------------------------------------------------------");
endfunction

*/
rand int da[];

/*
//2d dynamic array consecutive numbers
    rand int dynamic_a[][];
    constraint size_d{
        dynamic_a.size == 3; // allocating size for the 1st dimension
        foreach(dynamic_a[i])
            dynamic_a[i].size == 4;// allocating size for second dimension for each element
        }
    constraint value_c{
        foreach(dynamic_a[i,j])
            dynamic_a[i][j] == i+j;
        }
    function void post_randomize();
        $display("----------------------------------------------------------------------------");
        foreach(dynamic_a[i,j])begin
            $display("da[%d][%d] = %d",i,j, dynamic_a[i][j]);
        end
        $display("----------------------------------------------------------------------------");
    endfunction
*/


/*
//Pallindrome number check
   function int palli(int num);
       int temp, append, re;
       temp=num;
       for(int i=0; i<3; i++)begin
           re = temp%10;  //extract
           append = append * 10 + re;
           $display("re=%d, append=%d, temp=%d", re, append, temp);
           temp=temp/10;
       end
    if(num == append)begin
        $display("%d is paalindrome number", num);
        return num;
    end
   endfunction

constraint values_c{
    foreach(array[i])
        array[i] inside {121, 111, 122, 222, 333, 110, 012};
    }

function void post_randomize();
    int value, count;
    foreach(array[i])begin
        value = palli(array[i]);
        if(value)begin
            dummy[count] = value; 
            count++;
    end
    end
    $display("----------------------------------------------------------------------------");
    $display("value of array = %p", dummy);
    $display("----------------------------------------------------------------------------");
endfunction
*/


/*
//check whether random generated number is armstrong or not?
rand int arms;
constraint values_c{
    arms inside {[100:999]};
    }

function void post_randomize();
    int store;
    store = arm(arms);
    if(store == arms)begin
        $display("%d number is armstrong number", store);
    end else begin
        $display("%d is not armstrong",store);
    end
endfunction
function int arm(int num);
    int extract, temp, append;
    temp = num;
    for(int i=0; i<3; i++)begin
        extract = temp%10;
        append = extract**3 + append;
        $display("extract=%d, temp=%d, append=%d", extract, temp, append);
        temp = temp/10;
    end
    if(num == append)begin
        $display("%d number is armstrong number", num);
        return num;
    end else begin
        $display("%d is not armstrong",num);
        return num;
    end
endfunction
*/

/*
//fibonaci series is a series, so we have to make use of iterator variable

    constraint values_c{
        foreach(array[i])
            if(i==0)
                array[i] == 0;
            else if(i==1)
                array[i] == 1;
            else
                array[i] == array[i-1] + array[i-2];
            }
function void post_randomize();
    $display("----------------------------------------------------------------------------");
    $display("value of array = %p", array);
    $display("----------------------------------------------------------------------------");
endfunction
*/

/*
//two queues should not have same elements
rand int queue1[$];
rand int queue2[$];
constraint qsize_c{
    queue1.size == 10;
    queue2.size == 10;
    }
constraint qvalue_c{
    foreach(queue1[i]) {
        queue1[i] inside {[10:40]};
        queue2[i] inside {[10:40]};
        }
        }
 constraint not_c{
    foreach(queue1[i])
        !(queue1[i] inside {queue2}); // to be noted, each element of q1 is checked inside entire q2
        }
function void post_randomize();
    $display("----------------------------------------------------------------------------");
    $display("value of queue1 = %p", queue1);
    $display("value of queue2 = %p", queue2);
    $display("----------------------------------------------------------------------------");
endfunction
*/
/*
//distribution constraint 70% 30%

rand bit[7:0] a;

constraint values_c{
    a dist {[1:100]:=70, {[101:130]:= 30};
    }
*/

/*
//16bit variable such that no consecutive 1 

rand bit[15:0] vari;

constraint values_c{
    foreach(vari[i])
        if(i>0 && vari[i])
            vari[i] != vari[i-1];
    }
*/

/*
//write constraint such that when rand bit[3:0] a; is randomized then value of
//a should not be same as previous 5 values of a.

rand bit[3:0]a;
int q[$];
constraint values_c{
    !(a inside {q});
    }
function void post_randomize();
    q.push_back(a);
if( q.size == 6)begin
    q.pop_front();
end
$display("value is %p", q);
endfunction
*/


/*
//randc behaviour without using randc

rand bit[2:0] vec;
int q[$];
constraint values_c{
    !(vec inside {q});
    }
function void post_randomize();
    q.push_back(vec);
    if(q.size == 2**$size(vec))begin
        $display("deleting the que as it got full");
        q = {};// empty queue
    end
    $display(" values of q=%p",q);
endfunction
*/

/*
//write constraint such that payload size should be in between 11 and 22, each
//value of payload should be greate than previous value by 2.

    rand bit[7:0] payload[];
    constraint size_aa{
        payload.size inside {[11:22]};
        }
    constraint valued_aa{
        foreach(payload[i])
            if(i>0)
                payload[i] == payload[i-1]+2;
        }
    function void post_randomize();
    $display("----------------------------------------------------------------------------");
    $display("value of data = %p", payload);
    $display("----------------------------------------------------------------------------");
    endfunction
*/

/*
//sort elements of dynamic array
rand bit[3:0] sort[];
constraint sixe_c{
    sort.size == 5;
    }

function void post_randomize();
    $display("value of before sorting sort is %p", sort);
    for(int i=0; i<sort.size()-1; i++)begin
        for(int j=0; j<sort.size()-1-i; j++)begin
            if(sort[j] > sort[j+1])begin
                sort[j] = sort[j] + sort[j+1];
                sort[j+1] = sort[j] - sort[j+1];
                sort[j] = sort[j] - sort[j+1];
            end
        end
    end
    $display("value of sorted sort is %p", sort);
endfunction
*/

rand bit[3:0] td[4][4];
/*
// 44. constraint for {1000, 0100, 0010, 0001} diagonal matrix
constraint matrix_c{
    foreach(td[i,j])
        if(i == j)
            td[i][j] == 1;
        else
            td[i][j] == 0;
        }
*/

// 45. constraint for {1111, 1110, 1100, 1000} matrix
/*
constraint matrix_d{
    foreach(td[i,j])
        if(i==i && j<((4)-i))
            td[i][j] == 1;
        else 
            td[i][j] == 0;
        }

function void post_randomize();
    $display("----------------------------------------------------------------------------");
    $display("value of td = %p", td);
    $display("----------------------------------------------------------------------------");
endfunction
*/


endclass



//------------------------------------------------------------------------------------------
constraint cons_problems::range_c{
    da.size inside {[5:10]};
    }

//constraint cons_problems::asscending_c{
//    foreach(da[i])
//        if(i>0)
//            da[i] == da[i-1]+1;
//    }
//-------- OR--------
constraint cons_problems::asscending_c{
    foreach(da[i])
        if(i>0)
        da[i] > da[i-1];
    }
constraint cons_problems::data_c{
    foreach(da[i])
        da[i] inside {[1:30]};
    }

cons_problems c_h;

initial begin
    c_h=new();
    //c_h.arm(432);
    //repeat(20)begin
    if(!c_h.randomize())begin
        $error("Randomization failed");
    end
    //end
end
endmodule
module test;
class test;
rand bit data[4][4];

constraint test_c{
    foreach(data[i])
        foreach(data[i][j])
            data[i][j] == (!i[0]) ^ j[i[31:1]];
        }
function void post_randomize();
$display("value is %p", data);
endfunction
//constraint values_c{
//    foreach(data[i])
//        display(i);
//        }
//        function int display(int i);
//            fork
//                $display("data is %d", data[i]);
//            join_none
//                return 1;
//        endfunction
endclass

test t;
initial begin
t=new;
t.randomize();
end
endmodule

```
1. Write constraint to generate 01010101 pattern, 11110000  
x. Write constraint for power of 9 power of 3  
x. write constraint such that for two 4bit variables a and b, lsb of a and lsb of b should not be equal.  
x. constraint for 2 3 4 5 6 7 8 9 10 11 12 13 14 15  
x. generate 888887777766666555554444433333222221111100000  
x. mobile number first 4 digits must be 8919  
23. Write a constraint so that if we randomize a single bit variable for 10 times values should generate be like 101010101010.  
35. Write a constraint on a 16 bit rand vector to generate alternate pairs of 0's and 1's.  //Ans:Ex:0011.. or 1100..  
2. 1234554321, 123404321, 1010110101, 11101110, 000111222333, 123123123, 11001100110011001100, 00110011001100110011, 1,22,3,33,5,44,7,55, 1,11,3,22,5,33,7,44,9,55  
1,22,3,44,5,66,7,88,7,66,5,44,3,22,1  
01002000300004  
x. write a constraint for two random variables such that one variable does not match the other and five bits are toggled  
x. write a constraint for generating a gray code sequcne of 5bits  
x. write a constraint such that all elements in an array are powers of 2 and sorted in descending order  
  
3. 9 19 29 39 49 ..    and 9 99 999 9999 99999  
4. 5 -10 15 -20 25 -30...  
5. 1122334455....  
49. 9966631002  
6. generate random number between 1.35 to 2.57  
7. 0102030405  
8. 25 27 30 36 40 45 without using set membership  
9. generate random even number between 50 to 100  
10. for 32bit rand variable it should have 12 number of 1's non consecutively  
11. factorial of first 5 even numbers and odd numbers  
12. Write constraint such that even location should containt odd numbers and odd locations should contain even numbers  
13. for 32bit variable randomize only 12th bit  
14. Write a constraint on two dimensional array for generating even numbers in the first 4 locations and odd numbers in the next 4 locations.Also the even number should be in multiple of 4 and odd number should be multiple of 3.  
15. Constraint to generate bit[7:0] array1[10] with unique values and also multiple of 3  
16. Write a constraint to generate unique numbers in an array without using unique keyword.  
17. Write a constraint to generate prime numbers between the range of 1 to 100  
18. Write a constraint to generate a variable with 0-31 bits should be 1, 32-61 bits should be 0.  
19. Write a constraint to generate consecutive elements using Fixed size array and also write the constraint to generate non consecutive elements?  
20. write a constraint to randmoly genrate 10 unquie numbers between 99 to 100.  
21. Write a constraint sunch that array size between 5 to 10 values of the array are in asscending order.  //In this program Iam going to use extern constraint.  
22. Write a constraint to generate even numbers should in the range of 10 to 30 using Fixed size array, Dynamic array & Queue.  
24. Write a constraint to demonstrate solve before constraint.  
25. Write a code to generate unique elements in an array without using unique keyword and constraints  
  
26. Write a constraint for 2D dynamic array to print consecutive elements.  
27. Constraint to check whether the randomized number is palindrome or not  
28. Write a constraint to display the fibonacci sequence. and reverse fibonacci series  
29. Write a code to check whether the randomized number is an armstrong number or not.  
30. Write a constraint so that the elements in two queues are different.  
31. Constraint for 0-100 range is 70% and 101-255 range is 30%  
32. Write a constraint for 16-bit variable such that no two consecutive ones should be generated.  
33. Write a constraint to generate 32 bit number with 1 bit high using $onehot().  
34. Write a constraint to randomly generate unique prime numbers in an array between 1 and 200. The generated prime numbers should have 7 in it.  
36. Write a constraint such that sum of any three consecutive elements in an array should be an even number.  
37. Write a constraint for two random variables such that one variable should not match with the other & the total number of bits toggled in one variable should be 5 w.r.t the other.  
38. Write a constraint such that when rand bit[3:0] a is randomized, the value of "a" should not be same as 5 previous occurrences of the value of "a".  
39. Write a code to have the cyclic randomization behaviour without using the randc keyword.  
40. Write constraint for payload such that the size should be randomly generated in between 11 and 22 and each value of the payload should be grater than previous value by 2.  
41. Write a constraint to print unique elements in a 2D array without using unique keyword.  
42. Sorting the elements in the dynamic array using constraints. // not a constraint question  
43. Write a constraint to generate 1221122112211.....  
write constraint to generate pattern 021346578  
  
----------------------2D----------------------------  
44. constraint for {1000, 0100, 0010, 0001} diagonal matrix when i and j are equal make it 1 else 0  
45. constraint for {1111, 1110, 1100, 1000} matrix  
46. {1010, 0101, 1100, 0011}  
47. 0001, 0010, 0100, 1000  
48. 1234, 2341, 3421, 4123  
50. {{1,0,2,0,},{0,3,0,4},{5,0,6,0},{0,7,0,8}}  
51. constraint for 5*5 matrix such that last coloumn is sum of previous coloumns   
  
------------------array manipulator--------------  
  
