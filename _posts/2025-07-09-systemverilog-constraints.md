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
```verilog
module test;
    int result=1;
//pre_randomize is to perform constraint overriding if needed
//post_randomize is for modifying randomization results
//in constraint you can't use begin end block to write multiple lines, you
//have to use curly braces instead of begin end
//$onehot, $countbits, $countones

class cons_problems;
    rand int array[];
    int dummy[$];
    constraint size_c { array.size == 50;}
    //constraint size_d { dummy.size == 50;}
/*
//generate consecutive elements
    constraint values_c{
        array[i]== i;
        }
*/

/*
//pattern 01010101010101
    //constraint value_c {
    //    foreach(array[i])
    //        array[i]==i[0];
    //    }
    constraint value_c {
        foreach(array[i])
            if(i%2==0) //even number
                array[i] == 0;
            else
                array[i] == 1;
        }
*/

/*
//pattern 1234554321
    constraint value_c {
        foreach(array[i])
            if(i < 5)
                array[i] == i+1; // adding 1 because our pattern starts from 1 not from 0
            else 
                array[i] == 10 - i;

        } 
*/

/*
//9 19 29 39 49 59 ...

    constraint value_c{
            foreach(array[i])
                array[i] == i*10+9;
    }
*/

/*
//5 -10 15 -20 25 -30

    constraint value_c{
        foreach(array[i])
            if(i%2==0)
                //array[i] == -5*i; 
                array[i] == (5*i)+5;//adding 1 as it is starting from 5 not from 0
            else
                //array[i] == -5*i;//adding 1 as it is starting from 5 not from 0
                array[i] == -5*(i+1);//adding 1 as it is starting from 5 not from 0
            }
*/

/*
//1122334455...

    constraint value_c{
        foreach(array[i])
            array[i] == ((i+2)/2);
        }
*/

/*
//1.35 and 2.57
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
*/
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
*/

/*
//010203040506...

    constraint value_c{
        foreach(array[i])
            if(i%2==0)
                array[i] == 0;
            else
                array[i] == (i+2)/2;
            }
*/

/*
// 25 27 30 36 40 45
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
*/

/*
// 25 27 30 36 40 45
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
*/

/*
// random even number between 50 100 wihout 

    constraint value_c{
        foreach(array[i])
            array[i] inside {[50:100]};
        }
    constraint value_d{
        foreach(array[i])
            array[i]%2==0;
        }

*/

/*
//in 32bit variable, number of 1's should be 12 non consecutively
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
*/
/*
//factorial numbers
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
*/

/*
//odd numbers at even location and even numbers at odd locations

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
*/

/*
//  randomize only 12th bit out of 32bits rest of the bits should be 0s

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
*/
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
//prime number
rand int num;
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
            $display("The given number=%d is not a prime number", num);
            return 0; //if its not prime number then return 0 value
        end else begin
            //$display(" %d Number is prime", num);
            return num;//if its prime number then return that value
        end
    end

endfunction


function void post_randomize();
        result = prime(num);
        $display("----------------------------------------------------------------------------");
        $display("Prime number is %d", result);
        $display("----------------------------------------------------------------------------");
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
            return 3; //if its not prime number then return 0 value
        end else begin
            //$display(" %d Number is prime", num);
            return num;//if its prime number then return that value
        end
    end
endfunction

/*
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


```
