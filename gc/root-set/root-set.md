## Definition
  - A garbage collection root is an object that is accessible from outside the heap.
  
Root set might also refer to the root reference instead of root objects in some contexts.

## General GC Roots

- System class
  - A class that was loaded by the bootstrap loader, or the system class loader. For example, this category includes all classes in the rt.jar file (part of the Java™ runtime environment), such as those in the java.util.\* package.
- JNI local
  - A local variable in native code, for example user-defined JNI code or JVM internal code.
- JNI global
  - A global variable in native code, for example user-defined JNI code or JVM internal code.
- Thread block
  - An object that was referenced from an active thread block.
- Thread
  - A running thread.
- Busy monitor
  - Everything that called the wait() or notify() methods, or that is synchronized, for example by calling the synchronized(Object) method or by entering a synchronized method. If the method was static, the root is a class, otherwise it is an object.
- Java local
  - A local variable. For example, input parameters, or locally created objects of methods that are still in the stack of a thread.
- Native stack
  - Input or output parameters in native code, for example user-defined JNI code or JVM internal code. Many methods have native parts, and the objects that are handled as method parameters become garbage collection roots. For example, parameters used for file, network, I/O, or reflection operations.
- Finalizer
  - An object that is in a queue, waiting for a finalizer to run.
- Unfinalized
  - An object that has a finalize method, but was not finalized, and is not yet on the finalizer queue.
- Unreachable
  - An object that is unreachable from any other root, but was marked as a root by Memory Analyzer so that the object can be included in an analysis.


In HotSpot JVM, there are the following types of root sets. When objects are moved, the old oops are updated. So a rootClosure's `do_oop` method expects an oop\* type argument p, and eventually update the \*p content to be the new address `*p = new_obj;` 
  - [universe](universe-roots.md)
  - [jni_handles](jni-handles-roots.md)
  - object_synchronizer
  - flat_profiler
  - managment
  - system_dictionary
  - class_loader_data
  - jvmti
  - code_cache
  - thread_roots

