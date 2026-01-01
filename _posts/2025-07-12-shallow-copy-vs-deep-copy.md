## Shallow Copy and Deep Copy In System Verilog
Why we need shallow copy?  
To solve the problem of handle assignment we do shallow copy.  
ex:
```verilog
person p1, p2;
p1=new();
p1.name = "Jhon";
p2 = p1;
p2.name = "Wick"; // this will override the p1.name to Wick, because p2 and p1 share the same memory space. solution is shallow copy.
i.e
p2 = new p1;// when doing copy make new memory for p2 and then copy p1 into p2.
```
Why we need deep copy?  
Problem with shallow copy is that if there is any nested class i.e one class handle declared inside other class handle then shallow copy will not work for nested class. Deep copy is the solution.  

Let's take an example of a person and it's passport.

- If you shallow copy a person, both people point to the same passport (same object).
- If you deep copy, each person has a separate passport with the same info.

```verilog
module top;

  class Passport;
    string country;
    int number;
  endclass

  class Person;
    string name;
    int age;
    Passport passport; //passport class handle declaration inside person class. it is also called nesting of class.
  endclass

  Person p1, p2;

  initial begin
    // Create and initialize the first person
    p1 = new();
    p1.passport = new();
    p1.name = "Alice";
    p1.age = 30;
    p1.passport.country = "USA";
    p1.passport.number = 123456;

    $display("--- Original Person ---");
    $display("Name: %s, Age: %0d", p1.name, p1.age);
    $display("Passport: %s - %0d", p1.passport.country, p1.passport.number);

    // --- Shallow Copy ---
    p2 = new p1;  // Shallow copy: same passport handle

    p2.name = "Bob";  // Change name
    p2.passport.number = 999999;  // Change shared passport

    $display("\n--- After Shallow Copy and Modifying p2 ---");
    $display("p1: %s, Passport No: %0d", p1.name, p1.passport.number);  // Passport number also changed!
    $display("p2: %s, Passport No: %0d", p2.name, p2.passport.number);

    // --- Deep Copy ---
    p2 = new p1;                // Shallow copy base values again
    p2.passport = new p1.passport;  // Deep copy: new passport

    p2.name = "Charlie";
    p2.passport.number = 888888;

    $display("\n--- After Deep Copy and Modifying p2 ---");
    $display("p1: %s, Passport No: %0d", p1.name, p1.passport.number);  // p1's passport is untouched
    $display("p2: %s, Passport No: %0d", p2.name, p2.passport.number);
  end

endmodule


```
