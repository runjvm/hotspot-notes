
OopMaps are filled as fields are [parsed](./classfile/field-parser.md) at load time.

Pseudocode for oop_iterate

```c++
INLINE void InstanceKlass::oop_oop_iterate_oop_maps_specialized(oop obj, OopClosureType* closure) {
	OopMapBlock* map = start_of_nonstatic_oop_maps();
    OopMapBlock* const end_map = map + nonstatic_oop_map_count();
    for (; map < end_map; ++map) {
    	T* p = (T*)obj->obj_field_addr<T>(map->offset());
        T* const end = p + map->count();
        for (; p < end; ++p) {
        	Devirtualizer<nv>::do_oop(closure, p);
        }
    }
}
```
