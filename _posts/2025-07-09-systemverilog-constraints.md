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
