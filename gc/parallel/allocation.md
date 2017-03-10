
## Overview

Allocation routines are in CollectedHeap. basically, TLAB allocation is first tried, if that failed (by returning a NULL pointer or turning of UseTLAB), which means no new tlab can be allocated, the failed_allocation event will be handled by the derived heap. For example

```c++
HeapWord* ParallelScavengeHeap::mem_allocate(
  // trigger GC if allocation failed.
  ...
```

## Filler Object

Filler objects are instances of java_lang_Object. Appendix of well-known classes:

SystemDictionary::Object_klass() => java_lang_Object
SystemDictionary::Class_klass()  => java_lang_Class
... etc

## Call chain

- instanceOop InstanceKlass::allocate_instance(TRAPS)
  - CollectedHeap::obj_allocate(h_k, size, CHECK_NULL)
    

```c++

oop CollectedHeap::obj_allocate(KlassHandle klass, int size, TRAPS) {
  debug_only(check_for_valid_allocation_state());
  assert(!Universe::heap()->is_gc_active(), "Allocation during gc not allowed");
  assert(size >= 0, "int won't convert to size_t");
  HeapWord* obj = common_mem_allocate_init(klass, size, CHECK_NULL);
  post_allocation_setup_obj(klass, obj, size);
#ifdef PROFILE_OBJECT_ADDRESS_INFO
  if (ProfileObjectAddressInfo) {
    ObjectAddressInfoTable *oait = Universe::object_address_info_table();
    oait->insert((oop)obj, size, klass());
  }
#endif
  NOT_PRODUCT(Universe::heap()->check_for_bad_heap_word_value(obj, size));
  return (oop)obj;
}

HeapWord* CollectedHeap::common_mem_allocate_init(KlassHandle klass, size_t size, TRAPS) {
  HeapWord* obj = common_mem_allocate_noinit(klass, size, CHECK_NULL);
  init_obj(obj, size);
  return obj;
}


HeapWord* CollectedHeap::common_mem_allocate_noinit(KlassHandle klass, size_t size, TRAPS) {

  // Clear unhandled oops for memory allocation.  Memory allocation might
  // not take out a lock if from tlab, so clear here.
  CHECK_UNHANDLED_OOPS_ONLY(THREAD->clear_unhandled_oops();)

  if (HAS_PENDING_EXCEPTION) {
    NOT_PRODUCT(guarantee(false, "Should not allocate with exception pending"));
    return NULL;  // caller does a CHECK_0 too
  }

  HeapWord* result = NULL;
  if (UseTLAB) {
#ifdef COLORED_TLABS
    if (UseColoredSpaces) {
      result = CollectedHeap::allocate_from_tlab(klass, THREAD, size, UnknownObjectHeapColor);
    } else {
      result = CollectedHeap::allocate_from_tlab(klass, THREAD, size);
    }
#else
    result = allocate_from_tlab(klass, THREAD, size);
#endif
    if (result != NULL) {
      assert(!HAS_PENDING_EXCEPTION,
             "Unexpected exception, will result in uninitialized storage");
      return result;
    }
  }
  
  bool gc_overhead_limit_was_exceeded = false;
  result = Universe::heap()->mem_allocate(size,
                                          &gc_overhead_limit_was_exceeded);
  if (result != NULL) {
    NOT_PRODUCT(Universe::heap()->
      check_for_non_bad_heap_word_value(result, size));
    assert(!HAS_PENDING_EXCEPTION,
           "Unexpected exception, will result in uninitialized storage");
    THREAD->incr_allocated_bytes(size * HeapWordSize);

    AllocTracer::send_allocation_outside_tlab_event(klass, size * HeapWordSize);

    return result;
  }


  if (!gc_overhead_limit_was_exceeded) {
    // -XX:+HeapDumpOnOutOfMemoryError and -XX:OnOutOfMemoryError support
    report_java_out_of_memory("Java heap space");

    if (JvmtiExport::should_post_resource_exhausted()) {
      JvmtiExport::post_resource_exhausted(
        JVMTI_RESOURCE_EXHAUSTED_OOM_ERROR | JVMTI_RESOURCE_EXHAUSTED_JAVA_HEAP,
        "Java heap space");
    }

    THROW_OOP_0(Universe::out_of_memory_error_java_heap());
  } else {
    // -XX:+HeapDumpOnOutOfMemoryError and -XX:OnOutOfMemoryError support
    report_java_out_of_memory("GC overhead limit exceeded");

    if (JvmtiExport::should_post_resource_exhausted()) {
      JvmtiExport::post_resource_exhausted(
        JVMTI_RESOURCE_EXHAUSTED_OOM_ERROR | JVMTI_RESOURCE_EXHAUSTED_JAVA_HEAP,
        "GC overhead limit exceeded");
    }

    THROW_OOP_0(Universe::out_of_memory_error_gc_overhead_limit());
  }
}

HeapWord* CollectedHeap::allocate_from_tlab(KlassHandle klass, Thread* thread, size_t size,
  HeapColor color) {
  assert(UseTLAB, "should use UseTLAB");

  if (color==HC_NOT_COLORED) {
    color = UnknownObjectHeapColor;
  }

  HeapWord* obj = thread->tlab(color).allocate(size);
  if (obj != NULL) {
    return obj;
  }
  // Otherwise...
  return allocate_from_tlab_slow(klass, thread, size, color);
}


 inline HeapWord* ThreadLocalAllocBuffer::allocate(size_t size) {
   invariants();
   HeapWord* obj = top();
   if (pointer_delta(end(), obj) >= size) {
     // successful thread-local allocation                                                                                                                                              
 #ifdef ASSERT
     // Skip mangling the space corresponding to the object header to                                                                                                                   
     // ensure that the returned space is not considered parsable by                                                                                                                    
     // any concurrent GC thread.                                                                                                                                                       
     size_t hdr_size = oopDesc::header_size();
     // fill the object with badHeapWordVal(0xBAADBABE) except the header part.
     // fill_to_word, zero_to_word, or xxx_to_bytes are all cpu-dependent operations
     Copy::fill_to_words(obj + hdr_size, size - hdr_size, badHeapWordVal);
 #endif // ASSERT                                                                                                                                                                       
     // This addition is safe because we know that top is                                                                                                                               
     // at least size below end, so the add can't wrap.                                                                                                  // update top pointer                               
     set_top(obj + size);
 
     invariants();
     return obj;
   }
   return NULL;
 }



HeapWord* CollectedHeap::allocate_from_tlab_slow(KlassHandle klass, Thread* thread, size_t size,
  HeapColor color) {

  if (color==HC_NOT_COLORED) {
    color = UnknownObjectHeapColor;
  }

  // Retain tlab and allocate object in shared space if the amount free in the tlab is too large to discard.                                           
  // tz: allocating new tlab means the remaining space in old tlab will be wasted, which could otherwise hold smaller objects.                                                                                   
  if (thread->tlab(color).free() > thread->tlab(color).refill_waste_limit()) {
    thread->tlab(color).record_slow_allocation(size);
    return NULL;
  }

  // Discard tlab and allocate a new one.                                                                                                                                              
  // To minimize fragmentation, the last TLAB may be smaller than the rest.                                                                           // Compute the size for the new TLAB.                                 
  size_t new_tlab_size = thread->tlab(color).compute_size(size);

  // create filler object to make the mutableSpace seemingly contiguous
  thread->tlab(color).clear_before_allocation();

  if (new_tlab_size == 0) {
    return NULL;
  }

  // Allocate a new TLAB...                                                                                                                                                            
  HeapWord* obj = Universe::heap()->allocate_new_tlab(new_tlab_size, color);
  if (obj == NULL) {
    return NULL;
  }
  if (ZeroTLAB) {
    // ..and clear it.                                                                                                                                                                 
    Copy::zero_to_words(obj, new_tlab_size);
  } else {
    // ...and clear just the allocated object.                                                                                                                                         
    Copy::zero_to_words(obj, size);
  }
  thread->tlab(color).fill(obj, obj + size, new_tlab_size);
  return obj;
}

Caller of allocate_from_tlab =>

HeapWord* CollectedHeap::common_mem_allocate_init(KlassHandle klass, size_t size, TRAPS) {
  HeapWord* obj = common_mem_allocate_noinit(klass, size, CHECK_NULL);
  init_obj(obj, size);
  return obj;
}

// tlab->allocate() already did similar initialization, not sure why it is done here again
void CollectedHeap::init_obj(HeapWord* obj, size_t size) {
  assert(obj != NULL, "cannot initialize NULL object");
  const size_t hs = oopDesc::header_size();
  assert(size >= hs, "unexpected object size");
  ((oop)obj)->set_klass_gap(0);
  Copy::fill_to_aligned_words(obj + hs, size - hs);
}

Caller of common_mem_allocate_init =>
oop CollectedHeap::obj_allocate(KlassHandle klass, int size, TRAPS) {
  debug_only(check_for_valid_allocation_state());
  assert(!Universe::heap()->is_gc_active(), "Allocation during gc not allowed");
  assert(size >= 0, "int won't convert to size_t");
  HeapWord* obj = common_mem_allocate_init(klass, size, CHECK_NULL);
  post_allocation_setup_obj(klass, obj, size);
#ifdef PROFILE_OBJECT_ADDRESS_INFO
  if (ProfileObjectAddressInfo) {
    ObjectAddressInfoTable *oait = Universe::object_address_info_table();
    oait->insert((oop)obj, size, klass());
  }
#endif
  NOT_PRODUCT(Universe::heap()->check_for_bad_heap_word_value(obj, size));
  return (oop)obj;
}





 void CollectedHeap::post_allocation_install_obj_klass(KlassHandle klass,
                                                    oop obj) {
   // These asserts are kind of complicated because of klassKlass                                                                                                                       
   // and the beginning of the world.                                                                                                                                                   
   assert(klass() != NULL || !Universe::is_fully_initialized(), "NULL klass");
   assert(klass() == NULL || klass()->is_klass(), "not a klass");
   assert(obj != NULL, "NULL object pointer");
   obj->set_klass(klass());
 #ifdef PROFILE_OBJECT_INFO
   /* we call profile_object_alloc here and one other spot -- in                                                                                                                        
    * cpCacheKlass.cpp                                                                                                                                                                  
    */
   if (ProfileObjectInfo) {
     SharedRuntime::profile_object_alloc((oop)obj, obj->size(), klass());
   }
 #endif
   assert(!Universe::is_fully_initialized() || obj->klass() != NULL,
          "missing klass");
 }


```

















