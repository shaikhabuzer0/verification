## Shallow Copy and Deep Copy In System Verilog

Let's take an example of a person and it's passport.

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
