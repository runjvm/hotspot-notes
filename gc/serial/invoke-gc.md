
Serial GC uses genCollectedHeap, which is in gc/share/, some related classes in gc/share/ are listed below
- CollectedHeap
	- Base heap; Both genCollectedHeap and ParallelScavengeHeap inherit from it
- GenCollectedHeap
	- Inherit from collectedHeap, defines members like
    - Generation\* _young_gen, _old_gen;
- Generation
	- Base class for serial defNewGeneration, tenuredGeneration

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
	- 

