Two types of roots in universe
- java.lang.Class objects of primitive types
  - instance of java.lang.Class, represent a class, see [below](#Java.lang.Class)
  - created by java_lang_Class::create_basic_type_mirror(..)
- pre-defined exception objects
  - includes OutOfMemoryError, OutOfMemoryError, OutOfMemoryError..
  - instance of java.lang.Throwable, which is actually a class instead of an interface

## Java.lang.Class
According to Java.lang.Class's source code comment, only the Java Virtual Machine creates Class objects. To create Class objects, Class's klass needs to be created first. These Class objects of primitive types are created as part of SystemDictionary's initialization.

Consider the following code:

```Java
A a1 = new A();  
A a2 = new A();
System.out.println(a1.getClass() == a2.getClass()); // prints true
```
My guess is that the corresponding Class object may be created at parse time or created lazily on demand, and only one copy of an object is created for each Java class. a1.getClass() and a2.getClass() are different oops that point to the same object.

## HotSpot code
```c++
void Universe::oops_do(OopClosure* f, bool do_all) {  
  f->do_oop((oop*) &_int_mirror);
  f->do_oop((oop*) &_float_mirror);
  f->do_oop((oop*) &_double_mirror);
  f->do_oop((oop*) &_byte_mirror);
  f->do_oop((oop*) &_bool_mirror);
  f->do_oop((oop*) &_char_mirror);
  f->do_oop((oop*) &_long_mirror);
  f->do_oop((oop*) &_short_mirror);
  f->do_oop((oop*) &_void_mirror);

  for (int i = T_BOOLEAN; i < T_VOID+1; i++) {
    f->do_oop((oop*) &_mirrors[i]); // seems to be repeating the pointers above
  }
  assert(_mirrors[0] == NULL && _mirrors[T_BOOLEAN - 1] == NULL, "checking");

  f->do_oop((oop*)&_the_empty_class_klass_array);
  f->do_oop((oop*)&_the_null_string);
  f->do_oop((oop*)&_the_min_jint_string);
  f->do_oop((oop*)&_out_of_memory_error_java_heap);
  f->do_oop((oop*)&_out_of_memory_error_metaspace);
  f->do_oop((oop*)&_out_of_memory_error_class_metaspace);
  f->do_oop((oop*)&_out_of_memory_error_array_size);
  f->do_oop((oop*)&_out_of_memory_error_gc_overhead_limit);
  f->do_oop((oop*)&_out_of_memory_error_realloc_objects);
    f->do_oop((oop*)&_preallocated_out_of_memory_error_array);
  f->do_oop((oop*)&_null_ptr_exception_instance);
  f->do_oop((oop*)&_arithmetic_exception_instance);
  f->do_oop((oop*)&_virtual_machine_error_instance);
  f->do_oop((oop*)&_main_thread_group);
  f->do_oop((oop*)&_system_thread_group);
  f->do_oop((oop*)&_vm_exception);
  f->do_oop((oop*)&_allocation_context_notification_obj);
  debug_only(f->do_oop((oop*)&_fullgc_alot_dummy_array);)
}

```



