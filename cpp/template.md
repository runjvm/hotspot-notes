boolean parameters can be passed as a template parameter, so the value of it can be determined at compile time, 
the compiler can only keep the if part or the else part.

The parallelScavenge's promotionManager, the bool value prmote_immediately is passed as a template parameter.

## Example
```c++
template <bool stuff>
inline void doSomething() {
    if(stuff) {
        cout << "Hello" << endl;
    }
    else {
        cout << "Goodbye" << endl;
    }
}
```
call it like this:
```c++
doSomething<true>(); // print hello
doSomething<false>(); // print Goodbye
```
