boolean parameters can be passed as a template parameter, so the value of it can be determined at compile time, 
the compiler can only keep the if part or the else part.

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
``
And I call it like this:

doSomething<true>();
doSomething<false>();
It would pring out:

Hello
Goodbye
