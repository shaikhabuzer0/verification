## SystemVerilog Constraints Questions

Q1. Write a constraint to generate 01010101... pattern.

```verilog
class pattern_gen;

//Class members
rand int a[]; //dynamic array, each element is of 32bits. as our pattern is either 0 or 1 we can declare it as bit also.

constraint a_size{ a.size == 10;} //size of the array will be 10, if you randomize one time, 10 random values will get stored in this array.

//Instead of random 10 numbers we want some pattern as described above, hence writing below constraint.
constraint patt{
  foreach(a[i])
    if(i%2 == 0) //even location fill 0's
      a[i] == 0;
    else
      a[i] == 1; //odd locations fill 1's
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
