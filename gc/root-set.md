Definition
  - A garbage collection root is an object that is accessible from outside the heap.
  - A general reference on ["what kind of references are in root sets"](http://stackoverflow.com/questions/6366211/what-are-the-roots).

In HotSpot JVM, there are the following types of root sets.
  - [universe](root-set-universe.md)
  - jni_handles
  - object_synchronizer
  - flat_profiler
  - managment
  - system_dictionary
  - class_loader_data
  - jvmti
  - code_cache
  - thread_roots
