A new expression results in two separate things happening: allocation of the memory needed by the object begin created, and initialization of the object.

The new operator just handles the allocation part, then the object's constructor is called.

[This question](http://stackoverflow.com/questions/31106449/why-overloaded-new-operator-is-calling-constructor-even-i-am-using-malloc-inside) gives an example code.

There's even something called *placement new*, which initializes object in a preallocated space.

<http://www.cprogramming.com/tutorial/operator_new.html>
