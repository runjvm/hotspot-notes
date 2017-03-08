
Pseudocode
```c++

/* each OopMap has an entry in array nonstatic_oop_counts and nonstatic_oop_offsets 
 * If an OopMap has more than one oop_count, then it's oop fields are contigeous
 * Example (suppose HeapWord is 8):
 * OopMap {offset: 8, oop_count: 3} = > offset 8, 16, 24 are all oop type
 */

nonstatic_oop_map_count = 0;
nonstatic_oop_counts = new int[MAX_OOP_COUNT];
nonstatic_oop_offsets = new int[MAX_OOP_COUNT];


for (AllFieldStream fs(_fields, _cp); !fs.done(); fs.next()) {
	int real_offset;
	FieldAllocationType atype = (FieldAllocationType) fs.allocation_type();
    switch (atype) {
    	case NONSTATIC_OOP: {
    		real_offset = next_nonstatic_oop_offset;
            next_nonstatic_oop_offset += heapOopSize;
            if (this oop is adjacent to the previous one) {
            	nonstatic_oop_counts[nonstatic_oop_map_count]++;
            }
            else {
            	// This oop is not adjacent to the previous one, create new oop map
                nonstatic_oop_map_count += 1;
                nonstatic_oop_offsets[nonstatic_oop_map_count] = real_offset;
                nonstatic_oop_counts[nonstatic_oop_map_count] = 1;
                
            }
        }
        ...
    }

}

```