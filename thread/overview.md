![](https://cdn.discordapp.com/attachments/298832757327659008/298832899006791682/unknown.png)

- Primordial thread
  - The initial thread of HotSpot VM. Because this thread is created by the OS, HotSpot can't control the size of it's stack accurately, so this thread doesn't do much things. Pretty much the only thing it does is to create a new thread as Java's main thread and wait for joining, then exits.
  
- Main thread
  - This thread initializes JVM, and creates other threads below. It also starts Java'main method via JNI call.

- VM thread
  - This thread waits for operations to appear that require the JVM to reach a safe-point. The reason these operations have to happen on a separate thread is because they all require the JVM to be at a safe point where modifications to the heap can not occur. The type of operations performed by this thread are "stop-the-world" garbage collections, thread stack dumps, thread suspension and biased locking revocation.
  
- Periodic task thread
  - This thread is responsible for timer events (i.e. interrupts) that are used to schedule execution of periodic operations
  
- GC threads
  - These threads support the different types of garbage collection activities that occur in the JVM
  
- Compiler threads
  - These threads compile byte code to native code at runtime
  
- Signal dispatcher thread
  - This thread receives signals sent to the JVM process and handle them inside the JVM by calling the appropriate JVM methods.

A class hierarchy of some threads:
- Thread
  - NamedThread
    - VMThread
    - ConcurrentGCThread
    - WorkerThread
      - GangWorker
      - GCTaskThread
  - JavaThread
  - WatcherThread

## Reference
https://www.zhihu.com/question/27859000
