`c++
// The THREAD & TRAPS macros facilitate the declaration of functions that throw exceptions.
// Convention: Use the TRAPS macro as the last argument of such a function; e.g.:
// int this_function_may_trap(int x, float y, TRAPS)

#define THREAD __the_thread__ 
#define TRAPS  Thread* THREAD 

// Usage example:
instanceOop InstanceKlass::allocate_instance(TRAPS) { 
  ...
  KlassHandle h_k(THREAD, this);
  ...
}
`
