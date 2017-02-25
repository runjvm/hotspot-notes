Definition
  - A garbage collection root is an object that is accessible from outside the heap.
  - A general reference on ["what kind of references are in root sets"](http://stackoverflow.com/questions/6366211/what-are-the-roots).

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

