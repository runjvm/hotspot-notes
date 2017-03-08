
Serial GC uses genCollectedHeap, which is in gc/share/, some related files in gc/share/ are listed below
- collectedHeap
	- base heap; Both genCollectedHeap and ParallelScavengeHeap inherit from it
- genCollectedHeap
	- inherit from collectedHeap, defines members like
    - Generation\* _young_gen, _old_gen;

in gc/serial/
- defNewGeneration
	- serial "default new generation"
- genMarkSweep
	- inherit class MarkSweep, defines four phases of generational mark and sweep
- markSweep
	- define small basic operations of mark-sweep, such as iterating oops and updating pointers
- tenuredGeneration
	- 

