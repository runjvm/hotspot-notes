
Serial GC uses genCollectedHeap, which is in gc/share/

in gc/serial/
- defNewGeneration
	- serial "default new generation"
- genMarkSweep
	- inherit class MarkSweep, defines four phases of generational mark and sweep
- markSweep
	- define small basic operations of mark-sweep, such as iterating oops and updating pointers
- tenuredGeneration
	- 

