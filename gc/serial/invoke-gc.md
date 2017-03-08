
Serial GC uses genCollectedHeap, which is in gc/share/, some related classes in gc/share/ are listed below
- CollectedHeap
	- Base heap; Both genCollectedHeap and ParallelScavengeHeap inherit from it
- GenCollectedHeap
	- Inherit from collectedHeap, defines members like
    - `Generation\* _young_gen, _old_gen;`
    - Actual allocation and GC invocation routine is defined in GenCollectorPolicy
- Generation
	- Base class for serial defNewGeneration, tenuredGeneration
- GenCollectorPolicy
	- Defines actual `mem_allocate_work`, which is called by GenCollectedHeap' `mem_allocate`
    - Defines actual `satisfy_failed_allocation`, which is called as VMOperation
    - `satisfy_failed_allocation` is like PSHeap's `failed_mem_allocate`, they both invoke GC and has a policy to determine if full GC is needed
    
GC call stack in GenCollectedHeap
- GenCollectedHeap::do_collection( ..
- collect_generation(_young_gen, ..
- gen->collect(full, clear_soft_refs, size, is_tlab);


in gc/serial/
- DefNewGeneration
	- Serial "default new generation"
    - Inherit from Generation, defines
    - ContiguousSpace\* _eden_space, _from_space, _to_space;
- GenMarkSweep
	- Inherit class MarkSweep, defines four phases of generational mark and sweep
- MarkSweep
	- Define small basic operations of mark-sweep, such as iterating oops and updating pointers
- TenuredGeneration
	- Inherit from Generation
    




