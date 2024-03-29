# Getting Started with Base Classes: UVM_OBJECT
![image](https://github.com/coolnikitav/coding-lessons/assets/30304422/14ff39ce-9e73-4002-b706-62ed6e3feff7)

Static components - components that remain live for the entire duration of the simulation: driver, monitor, scoreboard

On the other hand, there will be a unique transaction for each new piece of data - dynamic component.

In uvm, static components are built using UVM_COMPONENT (uvm_driver for driver) and dynamic ones are built using UVM_OBJECT (ex: uvm_sequence_item for transaction)

uvm_component itself is derived from uvm_object, so it has all of its properties. uvm_component has a uvm_tree and phases, unlike uvm_object. 

*Base Class*
- uvm_object
  - uvm_transaction
  - uvm_sequence_item
  - uvm_sequence
- uvm_component
  - uvm_driver
  - uvm_sequencer
  - uvm_monitor
  - uvm_agent
  - uvm_scoreboard
  - uvm_env
  - uvm_test
 
Core Methods (Field Macros of data members)
- Print
- Record
- Copy
- Compare
- Create
- Clone
- Pack + unpack

User defined do_methods
- do_print
- do_record
- do_copy
- do_compare
- do_pack
- do_unpack

When you specify implementation for any core method then Field Macros are not required.

```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;  // object class is used to implement all dynamic components of testbench
  `uvm_object_utils(obj);  // macro to register class to factory derived from UVM_OBJECT
  
  function new(string path = "OBJ");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    `uvm_info("TB_TOP", $sformatf("value of a: %0d", o.a), UVM_NONE);
  end
  
endmodule
```

```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;  // object class is used to implement all dynamic components of testbench
  
  function new(string path = "OBJ");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  
  `uvm_object_utils_begin(obj)
  `uvm_field_int(a, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    o.print();
  end
  
endmodule
```
```
# KERNEL: ---------------------------
# KERNEL: Name  Type      Size  Value
# KERNEL: ---------------------------
# KERNEL: obj   obj       -     @335 
# KERNEL:   a   integral  4     'h6  
# KERNEL: ---------------------------
```

Field macros are not as run-time efficient nor as flexible as direct implementations of the do_* methods

Examples:
- Automation: print
- do_ method: do_print
- User method: display

```
o.print(uvm_default_tree_printer);

# KERNEL: obj: (obj@335) {
# KERNEL:   a: 'h6 
# KERNEL: }
```
```
o.print(uvm_default_line_printer);

# KERNEL: obj: (obj@335) { a: 'h6  }
```
```
`uvm_field_int(a, UVM_DEFAULT | UVM_BIN)

# KERNEL: obj: (obj@335) { a: 'b110  } 
```

If you don't register the variable to a factory, its name and value will not be printed:
```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;  // object class is used to implement all dynamic components of testbench
  
  `uvm_object_utils(obj)
  
  function new(string path = "OBJ");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  
  /*
  `uvm_object_utils_begin(obj)
  	`uvm_field_int(a, UVM_DEFAULT | UVM_BIN)
  `uvm_object_utils_end
  */
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    o.print(uvm_default_table_printer);
  end
  
endmodule

# KERNEL: -----------------------
# KERNEL: Name  Type  Size  Value
# KERNEL: -----------------------
# KERNEL: obj   obj   -     @335 
# KERNEL: -----------------------
```

Variable a has a UVM_NOPRINT flag
```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;  // object class is used to implement all dynamic components of testbench
    
  function new(string path = "OBJ");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  rand bit [7:0] b;
  
  `uvm_object_utils_begin(obj)
  	`uvm_field_int(a, UVM_NOPRINT | UVM_BIN)
  	`uvm_field_int(b, UVM_DEFAULT | UVM_BIN)
  `uvm_object_utils_end
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    o.print(uvm_default_table_printer);
  end
  
endmodule

# KERNEL: --------------------------------
# KERNEL: Name  Type      Size  Value     
# KERNEL: --------------------------------
# KERNEL: obj   obj       -     @335      
# KERNEL:   b   integral  8     'b10100101
# KERNEL: --------------------------------
```

```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;  // object class is used to implement all dynamic components of testbench
   
  typedef enum bit [1:0] {s0,s1,s2,s3} state_type;
  rand state_type state;
  
  real temp = 12.34;
  string str = "UVM";
  
  function new(string path = "OBJ");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(obj)
  	`uvm_field_enum(state_type, state, UVM_DEFAULT)
  	`uvm_field_string(str, UVM_DEFAULT)
  	`uvm_field_real(temp, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

module tb;
  
  obj o;
  
  initial begin
    o = new("obj");
    o.print(uvm_default_table_printer);
  end
  
endmodule

# KERNEL: ------------------------------------
# KERNEL: Name     Type        Size  Value    
# KERNEL: ------------------------------------
# KERNEL: obj      obj         -     @335     
# KERNEL:   state  state_type  2     s0       
# KERNEL:   str    string      3     UVM      
# KERNEL:   temp   real        64    12.340000
# KERNEL: ------------------------------------
```

```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class parent extends uvm_object;
  
  function new (string path = "parent");
    super.new(path);
  endfunction
  
  rand bit [3:0] data;
  
  `uvm_object_utils_begin(parent)
  	`uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

class child extends uvm_object;
  
  parent p;
  
  function new (string path = "child");
    super.new(path);
    p = new("parent");  // build_phase + create
  endfunction
  
  `uvm_object_utils_begin(child)
  	`uvm_field_object(p, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

module tb;
  
  child c;
  
  initial begin
    c = new("child");
    c.p.randomize();
    c.print();
  end
  
endmodule

# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: child     child     -     @335 
# KERNEL:   p       parent    -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
```
```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class array extends uvm_object;
  
  int arr1[3] = {1,2,3};  // static array
  
  int arr2[];             // dynamic array
  
  int arr3[$];            // queue
  
  int arr4[int];          // associative array
  
  function new (string path = "array");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(array)
    `uvm_field_sarray_int(arr1, UVM_DEFAULT)
    `uvm_field_array_int(arr2, UVM_DEFAULT)
    `uvm_field_queue_int(arr3, UVM_DEFAULT)
    `uvm_field_aa_int_int(arr4, UVM_DEFAULT)
  `uvm_object_utils_end
  
  task run();
    // Dynamic array value update
    arr2    = new[3];
    arr2[0] = 2;
    arr2[1] = 2;
    arr2[2] = 2;
    
    // Queue
    arr3.push_front(3);
    arr3.push_front(3);
    
    // Associative array
    arr4[1] = 4;
    arr4[2] = 4;
    arr4[3] = 4;
    arr4[4] = 4;
  endtask
  
endclass

////////////////////////////

module tb;
  
  array a;
  
  initial begin
    a = new("array");
    a.run();
    a.print();
  end
  
endmodule

# KERNEL: ----------------------------------
# KERNEL: array    array         -     @335 
# KERNEL:   arr1   sa(integral)  3     -    
# KERNEL:     [0]  integral      32    'h1  
# KERNEL:     [1]  integral      32    'h2  
# KERNEL:     [2]  integral      32    'h3  
# KERNEL:   arr2   da(integral)  3     -    
# KERNEL:     [0]  integral      32    'h2  
# KERNEL:     [1]  integral      32    'h2  
# KERNEL:     [2]  integral      32    'h2  
# KERNEL:   arr3   da(integral)  2     -    
# KERNEL:     [0]  integral      32    'h3  
# KERNEL:     [1]  integral      32    'h3  
# KERNEL:   arr4   aa(int,int)   4     -    
# KERNEL:     [1]  integral      32    'h4  
# KERNEL:     [2]  integral      32    'h4  
# KERNEL:     [3]  integral      32    'h4  
# KERNEL:     [4]  integral      32    'h4  
```

```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class first extends uvm_object;
  
  rand bit [3:0] data;
  
  function new (string path = "first");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first)
  `uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

module tb;
  
  first f;
  first s;
  
  initial begin
    f = new("first");
    s = new("second");
    f.randomize();
    s.copy(f);
    f.print();
    s.print();
  end
  
endmodule

# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: first   first     -     @335 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: second  first     -     @336 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
```

```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class first extends uvm_object;
  
  rand bit [3:0] data;
  
  function new (string path = "first");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first)
  	`uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

module tb;
  
  first f;
  first s;
  
  initial begin
    f = new("first");
    f.randomize();
    $cast(s, f.clone());  // f.clone() return a parent class (uvm_object)
    
    f.print();
    s.print();
  end
  
endmodule

# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: first   first     -     @335 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: first   first     -     @336 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
```

Shallow copy:
```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class first extends uvm_object;
  
  rand bit [3:0] data;
  
  function new (string path = "first");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first)
  	`uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

class second extends uvm_object;
  
  first f;
  
  function new (string path = "second");
    super.new(path);
    f = new("first");
  endfunction
  
  `uvm_object_utils_begin(second)
  	`uvm_field_object(f, UVM_DEFAULT)
  `uvm_object_utils_end
endclass

////////////////////////////

module tb;
  
  second s1, s2;
  
  initial begin
    s1 = new("s1");
    s2 = new("s2");
    
    s1.f.randomize();
    s1.print();
    s2 = s1;
    s2.print();
  end
  
endmodule

# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
```

```
initial begin
    s1 = new("s1");
    s2 = new("s2");
    
    s1.f.randomize();
    s1.print();
    s2 = s1;
    s2.print();
    
    s2.f.data = 12;
    s1.print();
    s2.print();
end

# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'hc  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'hc  
# KERNEL: -------------------------------
```

Deep copy:
```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class first extends uvm_object;
  
  rand bit [3:0] data;
  
  function new (string path = "first");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first)
  	`uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

class second extends uvm_object;
  
  first f;
  
  function new (string path = "second");
    super.new(path);
    f = new("first");
  endfunction
  
  `uvm_object_utils_begin(second)
  	`uvm_field_object(f, UVM_DEFAULT)
  `uvm_object_utils_end
endclass

////////////////////////////

module tb;
  
  second s1, s2;
  
  initial begin
    s1 = new("s1");
    s2 = new("s2");
    
    s1.f.randomize();
    s2.copy(s1);
    
    s1.print();
    s2.print();
  end
  
endmodule

# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s2        second    -     @337 
# KERNEL:   f       first     -     @339 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
```

```
  initial begin
    s1 = new("s1");
    s2 = new("s2");
    
    s1.f.randomize();
    s2.copy(s1);
    
    s1.print();
    s2.print();
    
    s2.f.data = 12;
    s1.print();
    s2.print();
  end

# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s2        second    -     @337 
# KERNEL:   f       first     -     @339 
# KERNEL:     data  integral  4     'hc  
# KERNEL: -------------------------------
```
Clone:
```
module tb;
  
  second s1, s2;
  
  initial begin
    s1 = new("s1");
    
    s1.f.randomize();
    $cast(s2,s1.clone());
    s1.print();
    s2.print();
    
    s2.f.data = 12;
    s1.print();
    s2.print();
  end
  
endmodule

# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @337 
# KERNEL:   f       first     -     @339 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @335 
# KERNEL:   f       first     -     @336 
# KERNEL:     data  integral  4     'h2  
# KERNEL: -------------------------------
# KERNEL: -------------------------------
# KERNEL: Name      Type      Size  Value
# KERNEL: -------------------------------
# KERNEL: s1        second    -     @337 
# KERNEL:   f       first     -     @339 
# KERNEL:     data  integral  4     'hc  
# KERNEL: -------------------------------
```

Compare method
```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class first extends uvm_object;
  
  rand bit [3:0] data;
  
  function new (string path = "first");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first)
  	`uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

module tb;
  
  first f1, f2;
  int status = 0;
  
  initial begin
    f1 = new("f1");
    f2 = new("f2");
    f1.randomize();
    f2.randomize();
    f1.print();
    f2.print();
    
    status = f1.compare(f2);
    $display("Value of status: %0d", status);
  end
  
endmodule

# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f1      first     -     @335 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f2      first     -     @336 
# KERNEL:   data  integral  4     'h2  
# KERNEL: -----------------------------
# KERNEL: UVM_INFO /home/build/vlib1/vlib/uvm-1.2/src/base/uvm_comparer.svh(351) @ 0: reporter [MISCMP] Miscompare for f1.data: lhs = 'h6 : rhs = 'h2
# KERNEL: UVM_INFO /home/build/vlib1/vlib/uvm-1.2/src/base/uvm_comparer.svh(382) @ 0: reporter [MISCMP] 1 Miscompare(s) for object f2@336 vs. f1@335
# KERNEL: Value of status: 0
```
```
initial begin
    f1 = new("f1");
    f2 = new("f2");
    f1.randomize();
    f2.randomize();
    f2.copy(f1);
    f1.print();
    f2.print();
    
    status = f1.compare(f2);
    $display("Value of status: %0d", status);
  end

# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f1      first     -     @335 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f2      first     -     @336 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
# KERNEL: Value of status: 1
```

Create method
```
# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f1      first     -     @335 
# KERNEL:   data  integral  4     'h6  
# KERNEL: -----------------------------
# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f2      first     -     @336 
# KERNEL:   data  integral  4     'h2  
# KERNEL: -----------------------------
```
Factory override: new vs create method

Let's say you completed a testbench and used it. In the next version, you want to add a control signal to the transaction class. Instead of changing the transaction class, you can extend the class and add the new signal.
Override method needs to have a UVM component.
```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class first extends uvm_object;
  
  rand bit [3:0] data;
  
  function new (string path = "first");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first)
  	`uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

class first_mod extends first;
  rand bit ack;
  
  function new (string path = "first_mod");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first_mod)
  	`uvm_field_int(ack, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

class comp extends uvm_component;
  `uvm_component_utils(comp)
  
  first f;
  
  function new (string path = "second", uvm_component parent = null);
    super.new(path,parent);
    f = first::type_id::create("f");
    f.randomize();
    f.print();
  endfunction
endclass
////////////////////////////

module tb;
  
  comp c;
  
  initial begin
    c = comp::type_id::create("c", null);
  end
  
endmodule

# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f       first     -     @344 
# KERNEL:   data  integral  4     'h3  
# KERNEL: -----------------------------
```
```
module tb;
  
  comp c;
  
  initial begin
    c.set_type_override_by_type(first::get_type, first_mod::get_type);  // replace first with first_mod
    c = comp::type_id::create("c", null);
  end
  
endmodule

# KERNEL: ------------------------------
# KERNEL: Name    Type       Size  Value
# KERNEL: ------------------------------
# KERNEL: f       first_mod  -     @344 
# KERNEL:   data  integral   4     'hc  
# KERNEL:   ack   integral   1     'h1  
# KERNEL: ------------------------------
```
Class not registered to a factory:
```
`include "uvm_macros.svh"
import uvm_pkg::*;

////////////////////////////

class first extends uvm_object;
  
  rand bit [3:0] data;
  
  function new (string path = "first");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first)
  	`uvm_field_int(data, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

class first_mod extends first;
  rand bit ack;
  
  function new (string path = "first_mod");
    super.new(path);
  endfunction
  
  `uvm_object_utils_begin(first_mod)
  	`uvm_field_int(ack, UVM_DEFAULT)
  `uvm_object_utils_end
  
endclass

////////////////////////////

class comp extends uvm_component;
  `uvm_component_utils(comp)
  
  first f;
  
  function new (string path = "second", uvm_component parent = null);
    super.new(path,parent);
    f = new("f");
    f.randomize();
    f.print();
  endfunction
endclass
////////////////////////////

module tb;
  
  comp c;
  
  initial begin
    c.set_type_override_by_type(first::get_type, first_mod::get_type);  // replace first with first_mod
    c = new("c", null);
  end
  
endmodule

# KERNEL: -----------------------------
# KERNEL: Name    Type      Size  Value
# KERNEL: -----------------------------
# KERNEL: f       first     -     @344 
# KERNEL:   data  integral  4     'hc  
# KERNEL: -----------------------------
```
do_print method

1) If we plan to use do methods, Field Macros are not required but registering class to factory is mandatory to get capabilities of Factory Override.

2) If we plan to use inbuilt implementation of the code methods then registering class tofactory as well as adding field macros to the data memebers is mandatory.

```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new (string path = "obj");
    super.new(path);
  endfunction
  
  bit [3:0] a = 4;
  string b = "UVM";
  real c = 12.34;
  
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    
    printer.print_field_int("a", a, $bits(a), UVM_HEX);
    printer.print_string("b", b);
    printer.print_real("c", c);
  endfunction  
endclass

module tb;
  obj o;
  
  initial begin
    o = obj::type_id::create("o");
    o.print();
  end
endmodule

# KERNEL: -------------------------------
# KERNEL: Name  Type      Size  Value    
# KERNEL: -------------------------------
# KERNEL: o     obj       -     @335     
# KERNEL:   a   integral  4     'h4      
# KERNEL:   b   string    3     UVM      
# KERNEL:   c   real      64    12.340000
# KERNEL: -------------------------------
```

Convert2string method
```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new (string path = "obj");
    super.new(path);
  endfunction
  
  bit [3:0] a = 4;
  string b = "UVM";
  real c = 12.34;
  
  virtual function string convert2string();
    string s = super.convert2string();
    
    s = {s, $sformatf("a: %0d; ", a)};
    s = {s, $sformatf("b: %0s; ", b)};
    s = {s, $sformatf("c: %0f", c)};
    
    return s;   
  endfunction
endclass

module tb;
  obj o;
  
  initial begin
    o = obj::type_id::create("o");
    $display("%0s", o.convert2string());
  end
endmodule

# KERNEL: a: 4; b: UVM; c: 12.340000
```
```
module tb;
  obj o;
  
  initial begin
    o = obj::type_id::create("o");
    //$display("%0s", o.convert2string());
    `uvm_info("TB_TOP", $sformatf("%0s", o.convert2string()), UVM_NONE);
  end
endmodule

# KERNEL: UVM_INFO /home/runner/testbench.sv(32) @ 0: reporter [TB_TOP] a: 4; b: UVM; c: 12.340000
```

do_copy method
```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new (string path = "obj");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  rand bit [4:0] b;
  
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    printer.print_field_int("a: ", a, $bits(a), UVM_DEC);
    printer.print_field_int("b: ", b, $bits(b), UVM_DEC);
  endfunction
  
  virtual function void do_copy(uvm_object rhs);
    obj temp;
    $cast(temp, rhs);
    super.do_copy(rhs);
    this.a = temp.a;
    this.b = temp.b;
  endfunction
endclass

module tb;
  obj o1,o2;
  
  initial begin
    o1 = obj::type_id::create("o1");
    o2 = obj::type_id::create("o2");
    
    o1.randomize();
    o1.print();
    o2.copy(o1);
    o2.print();
  end
endmodule

# KERNEL: ----------------------------
# KERNEL: Name   Type      Size  Value
# KERNEL: ----------------------------
# KERNEL: o1     obj       -     @335 
# KERNEL:   a:   integral  4     'd6  
# KERNEL:   b:   integral  5     'd5  
# KERNEL: ----------------------------
# KERNEL: ----------------------------
# KERNEL: Name   Type      Size  Value
# KERNEL: ----------------------------
# KERNEL: o2     obj       -     @336 
# KERNEL:   a:   integral  4     'd6  
# KERNEL:   b:   integral  5     'd5  
# KERNEL: ----------------------------
```

do_compare
```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new (string path = "obj");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  rand bit [4:0] b;
  
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    printer.print_field_int("a: ", a, $bits(a), UVM_DEC);
    printer.print_field_int("b: ", b, $bits(b), UVM_DEC);
  endfunction
  
  virtual function void do_copy(uvm_object rhs);
    obj temp;
    $cast(temp, rhs);
    super.do_copy(rhs);
    this.a = temp.a;
    this.b = temp.b;
  endfunction
  
  virtual function bit do_compare(uvm_object rhs, uvm_comparer comparer);
    obj temp;
    int status;
    $cast(temp,rhs);
    status = super.do_compare(rhs, comparer) && (a == temp.a) && (b == temp.b);
    return status;
  endfunction
endclass

module tb;
  obj o1,o2;
  int status;
  
  initial begin
    o1 = obj::type_id::create("o1");
    o2 = obj::type_id::create("o2");
    
    o1.randomize();
    o1.print();
    
    status = o2.compare(o1);
    $display("status: %0d", status);
  end
endmodule
```
```
`include "uvm_macros.svh"
import uvm_pkg::*;

class obj extends uvm_object;
  `uvm_object_utils(obj)
  
  function new (string path = "obj");
    super.new(path);
  endfunction
  
  rand bit [3:0] a;
  rand bit [4:0] b;
  
  virtual function void do_print(uvm_printer printer);
    super.do_print(printer);
    printer.print_field_int("a: ", a, $bits(a), UVM_DEC);
    printer.print_field_int("b: ", b, $bits(b), UVM_DEC);
  endfunction
  
  virtual function void do_copy(uvm_object rhs);
    obj temp;
    $cast(temp, rhs);
    super.do_copy(rhs);
    this.a = temp.a;
    this.b = temp.b;
  endfunction
  
  virtual function bit do_compare(uvm_object rhs, uvm_comparer comparer);
    obj temp;
    int status;
    $cast(temp,rhs);
    status = super.do_compare(rhs, comparer) && (a == temp.a) && (b == temp.b);
    return status;
  endfunction
endclass

module tb;
  obj o1,o2;
  int status;
  
  initial begin
    o1 = obj::type_id::create("o1");
    o2 = obj::type_id::create("o2");
    
    o1.randomize();
    o1.print();
    o2.print();
    
    o2.copy(o1);
    status = o2.compare(o1);
    $display("status: %0d", status);
  end
endmodule

# KERNEL: ----------------------------
# KERNEL: Name   Type      Size  Value
# KERNEL: ----------------------------
# KERNEL: o1     obj       -     @335 
# KERNEL:   a:   integral  4     'd6  
# KERNEL:   b:   integral  5     'd5  
# KERNEL: ----------------------------
# KERNEL: ----------------------------
# KERNEL: Name   Type      Size  Value
# KERNEL: ----------------------------
# KERNEL: o2     obj       -     @336 
# KERNEL:   a:   integral  4     'd0  
# KERNEL:   b:   integral  5     'd0  
# KERNEL: ----------------------------
# KERNEL: status: 1
```

Assignment 6:
```
`include "uvm_macros.svh";
import uvm_pkg::*;

class obj extends uvm_object;
  function new (string path = "obj"); 
    super.new(path);
  endfunction
  
  rand logic [1:0] a;
  rand logic [3:0] b;
  rand logic [7:0] c;
  
  `uvm_object_utils_begin(obj)
  	`uvm_field_int(a, UVM_BIN);
    `uvm_field_int(b, UVM_BIN);
    `uvm_field_int(c, UVM_BIN);
  `uvm_object_utils_end
endclass

module tb;
  obj o;
  
  initial begin
    o = new("obj");
    o.randomize();
    o.print();
  end
endmodule

# KERNEL: ------------------------------
# KERNEL: Name  Type      Size  Value   
# KERNEL: ------------------------------
# KERNEL: obj   obj       -     @335    
# KERNEL:   a   integral  2     'b10    
# KERNEL:   b   integral  4     'b101   
# KERNEL:   c   integral  8     'b100011
# KERNEL: ------------------------------
```

Assignment 7:
```
`include "uvm_macros.svh";
import uvm_pkg::*;

class my_object extends uvm_object; 
  rand logic [1:0] a;
  rand logic [3:0] b;
  rand logic [7:0] c;
  
  `uvm_object_utils_begin(my_object)
    `uvm_field_int(a, UVM_DEFAULT)
    `uvm_field_int(b, UVM_DEFAULT)
    `uvm_field_int(c, UVM_DEFAULT)
  `uvm_object_utils_end
  
  function new (string path = "my_object");
    super.new(path);
  endfunction
  
  virtual function void do_copy(uvm_object rhs);
    my_object temp;
    $cast(temp, rhs);
    super.do_copy(rhs);
    this.a = temp.a;
    this.b = temp.b;
    this.c = temp.c;
  endfunction
endclass

module tb;
  my_object o1,o2;
  int status;
  
  initial begin
    o1 = my_object::type_id::create("o1");
    o2 = my_object::type_id::create("o2");
    
    o1.randomize();
    //$cast(o2,o1.clone());
    o2.copy(o1);
    
    o1.print();
    o2.print();
    
    status = o1.compare(o2);
    $display("status: %0d", status);
  end
endmodule

# KERNEL: ----------------------------
# KERNEL: Name  Type       Size  Value
# KERNEL: ----------------------------
# KERNEL: o1    my_object  -     @335 
# KERNEL:   a   integral   2     'h2  
# KERNEL:   b   integral   4     'h5  
# KERNEL:   c   integral   8     'h23 
# KERNEL: ----------------------------
# KERNEL: ----------------------------
# KERNEL: Name  Type       Size  Value
# KERNEL: ----------------------------
# KERNEL: o2    my_object  -     @336 
# KERNEL:   a   integral   2     'h2  
# KERNEL:   b   integral   4     'h5  
# KERNEL:   c   integral   8     'h23 
# KERNEL: ----------------------------
# KERNEL: status: 1
```
