- Metaspace memory is collected if it reaches the MaxMetaspaceSize, which by default is unlimited, but can be set.
- Metaspace uses native memory and is no longer contiguous to the Java heap
- java.lang.Class instances are not stored in metaspace or permgen. Metadata stored at metaspace is only used internally by the VM.

More formally, per classloader storage area is called “a metaspace”. And these metaspaces are collectively called “the Metaspace”. The reclamation of metaspace per classloader can happen only after its classloader is no longer alive and is reported dead by the garbage collector. There is no relocation or compaction in these metaspaces. But metadata is scanned for Java references.

The Metaspace VM manages the Metaspace allocation by employing a chunking allocator. The chunking size depends on the type of the classloader. There is a global free list of chunks. Whenever a classloader needs a chunk, it gets it out of this global list and maintains its own chunk list. When any classloader dies, its chunks are freed, and returned back to the global free list. The chunks are further divided into blocks and each block holds a unit of metadata. The allocation of blocks from chunks is linear (pointer bump). The chunks are allocated out of memory mapped (mmapped) spaces. There is a linked list of such global virtual mmapped spaces and whenever any virtual space is emptied, its returned back to the operating system.

!(linked-list)[https://cdn.infoq.com/statics_s1_20170221-0307u1/resource/articles/Java-PERMGEN-Removed/en/resources/2Fig2.jpg]
