## Mark Word

<img src="https://snag.gy/ZgtiqF.jpg" width=500>

Note that the two tag bits take advantage of the fact that the pointers are at least 4 bytes aligned (primitive types are aligned according to their sizes). So the last two bits will also be 00 when mark word is assigned to a pointer. Then the corresponding tag bit is set, which is called "encoding". When interpreting the mark word as a pointer, the two tag bits are cleared before deferencing.

### Header Layout

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

```c++
class oopDesc {
	volatile markOop* _mark;
    union {
    	Klass* _klass;
        narrowKlass _compressed_klass;
    }
    ...
}
```
narrowKlass is typedef as jint. It's compressed the same way as narrowOop.

## UseCompressedOops: 32-bit Represents 32 GB Space
- 35 bits reference 32 GB of memory
- needs to be scaled by a factor of 8 and added to the heap base address to find the object to which they refer
- CompressedOops can reference up to 32 GB
- 33 GB heap can't use CompressedOop

## UseCompressedClassPointers
In the current version (as in Dec. 2017) of Java, `UseCompressedOops` has to be on in order to use `UseCompressedClassPointers`. Compressing class pointers are implemented the same way as compressing oops, there is a narrow klass base. Someone proposed [this](https://bugs.openjdk.java.net/browse/JDK-6916625) issue to use `UseCompressedClassPointers` (if Perm Gen size < 4GB) even when `UseCompressedOops` is not applicable. 

By contrast, the Mark Word part can not be compressed because the pointers contained in the mark word are not allocated in this "base+offset\*8" fashion as with klass and oops. 

### OutOfMemoryError: Compressed class space  
If `UseCompressedClassPointers` is on then amount of space available for class metadata is fixed (i.e. specified by vm option)

If amount of space available for class metadata is exceeds `CompressedClassSpaceSize`, then java.lang.OutOfMemoryError Compressed class space is thrown.


