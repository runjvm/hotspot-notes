
Serial GC uses genCollectedHeap, which is in gc/share/, some related classes in gc/share/ are listed below

## Classes in gc/share/

- CollectedHeap
	- Base heap; Both genCollectedHeap and ParallelScavengeHeap inherit from it
- GenCollectedHeap
	- Inherit from collectedHeap, defines members like
    - `Generation* _young_gen, _old_gen;`
    - Actual allocation and GC invocation routine is defined in GenCollectorPolicy
- Generation
	- Base class for serial defNewGeneration, tenuredGeneration
- GenCollectorPolicy
	- Defines actual `mem_allocate_work`, which is called by GenCollectedHeap' `mem_allocate`
    - Defines actual `satisfy_failed_allocation`, which is called as VMOperation
    - `satisfy_failed_allocation` is like PSHeap's `failed_mem_allocate`, they both invoke GC and has a policy to determine if full GC is needed
    
## Classes in gc/serial/

- DefNewGeneration
	- Serial "default new generation"
    - Inherit from Generation, defines
    - Real collect routine for young gen
    - `ContiguousSpace* _eden_space, _from_space, _to_space;`
- GenMarkSweep
	- Inherit class MarkSweep, defines four phases of generational mark and sweep
- MarkSweep
	- Define small basic operations of mark-sweep, such as iterating oops and updating pointers
- TenuredGeneration
	- Inherit from Generation
    

## Flags

### ParNewGC vs ParallelScavenge

They are both parallel collectors for young gen. ParNewGC works with CMS but ParallelScavenge does not. For other discrepancies between the two, refer to [this post](http://hllvm.group.iteye.com/group/topic/37095).

### Serial garbage collector

- -XX:+UseSerialGC

### Throughput collector

- -XX:+UseParallelGC
	- -XX:+UseParallelOldGC is automatically on after JDK 7u4

The CMS collector
The G1 collector

- -XX:+UseConcMarkSweepGC
	- -XX:+UseParNewGC is enabled automatically
    - Parallel for young gen, CMS for old gen
- -XX:+UseConcMarkSweepGC -XX:-UseParNewGC
	- Serial (DefNew) for young gen, CMS for old gen
    



## GC call stack in GenCollectedHeap

- GenCollectedHeap::do_collection( ..
	- collect_generation(_young_gen, ..
		- gen->collect(full, clear_soft_refs, size, is_tlab); // goes to defNewGeneration.cpp

in DefNewGeneration::collect
- gch->gen_process_roots(..
- evacuate_followers.do_void()
	- GenCollectedHeap::oop_since_save_marks_iterate(..
    	- ContiguousSpace::oop_since_save_marks_iterate##nv_suffix( // in gc/share/space.cpp
        	- oopDesc::oop_iterate(.. // in oops/oop.inline.hpp
            	- klass()->oop_oop_iterate##nv_suffix(this, blk);
                	- oop_oop_iterate_oop_maps<nv>(obj, closure); // see [this](../../oops/instanceKlass.md)
