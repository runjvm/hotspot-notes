
## C++

`new` expression in C++ code = `new` operator + call object's constructor.

The new operator just handles the allocation part, then the object's constructor is called.

[This question](http://stackoverflow.com/questions/31106449/why-overloaded-new-operator-is-calling-constructor-even-i-am-using-malloc-inside) gives an example code.

There's even something called *placement new*, which initializes object in a preallocated space.

<http://www.cprogramming.com/tutorial/operator_new.html>

## Java

`new` in Java code = `new` + `invokespecial` in bytecode where `new` only allocates memory and `invokespecial` calls a non-virtual method, which is the constructor in this case. 
