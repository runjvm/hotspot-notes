## 64 Bit 

Eden space is 252M, it can satisfy 50 TLAB refills. Each TLAB is ~5.04M (5160K).

This is just in the beginning. Both eden space and TLAB's size could grow after a certain times of GC.

Besides, there are a few things that I didn't figure out 

1. the eden space is always initialized to 4032M in the beginning in psYoungGen. But by the time the first GC is triggered, always 252M is used, which is 1/16 of 4032M

2. eden space could grow after a certain times of GC. Do we need to adapt to that and always use a fixed portion of eden as migration space?

3. compiler threads are also a major source of tlab requests, why would they need to allocate objects in the heap? most of the space they requested are actually wasted