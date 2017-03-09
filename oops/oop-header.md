## Mark Word

<img src="https://snag.gy/ZgtiqF.jpg" width=500>

Note that the two tag bits take advantage of the fact that the pointers are at least 4 bytes aligned. So the last two bits will also be 00 when mark word is assigned to a pointer. Then the corresponding tag bit is set, which is called "encoding". When interpreting the mark word as a pointer, the two tag bits are cleared before deferencing.

A detailed layout

32 bit JVM
```
|----------------------------------------------------------------------------------------|--------------------|
|                                    Object Header (64 bits)                             |        State       |
|-------------------------------------------------------|--------------------------------|--------------------|
|                  Mark Word (32 bits)                  |      Klass Word (32 bits)      |                    |
|-------------------------------------------------------|--------------------------------|--------------------|
| identity_hashcode:25 | age:4 | biased_lock:1 | lock:2 |      OOP to metadata object    |       Normal       |
|-------------------------------------------------------|--------------------------------|--------------------|
|  thread:23 | epoch:2 | age:4 | biased_lock:1 | lock:2 |      OOP to metadata object    |       Biased       |
|-------------------------------------------------------|--------------------------------|--------------------|
|               ptr_to_lock_record:30          | lock:2 |      OOP to metadata object    | Lightweight Locked |
|-------------------------------------------------------|--------------------------------|--------------------|
|               ptr_to_heavyweight_monitor:30  | lock:2 |      OOP to metadata object    | Heavyweight Locked |
|-------------------------------------------------------|--------------------------------|--------------------|
|                                              | lock:2 |      OOP to metadata object    |    Marked for GC   |
|-------------------------------------------------------|--------------------------------|--------------------|
```
[64 bit JVM](http://arturmkrtchyan.com/java-object-header)

## CompressedOop: 32-bit Represents 32 GB Space
- 35 bits reference 32 GB of memory
- The last three bits are always 0 in an oop, 35 - 3 = 32
- 33 GB heap can't use CompressedOop

The Mark Word part can not be compressed because the pointers are not in the heap.
