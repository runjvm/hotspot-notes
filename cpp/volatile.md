Volatile has a different semantic meaning in C/C++ and Java. For more on this refer to the java bookmark and volatile's wiki page..

## C/C++
It is mainly used for memory mapped files, use in the case that the memory may be modified by other hardwares so that the compiler won't optimize away the variable even if it's not changed by the program itself.

Volatile type variables are not atomic, thus can not be used as a portable synchronization technique.

## Java
- Volatile provides atomic read or write on a variable. But it can't make the whole "read-update-write" process atomic.
- It forces write through everytime, so the update by a thread is also visible to other threads

