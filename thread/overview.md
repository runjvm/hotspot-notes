```
---------------  P R O C E S S  ---------------                                                                                                                                                              
                                                                                                                                                                                                             Java Threads: ( => current thread )                                                                                                                                                                          
  0x00007f75e4181800 JavaThread "Thread-2" [_thread_blocked, id=12864, stack(0x00007f758de81000,0x00007f758df82000)]                                                                                         
  0x00007f75e4121000 JavaThread "Service Thread" daemon [_thread_blocked, id=12860, stack(0x00007f75957d8000,0x00007f75958d9000)]                                                                            
  0x00007f75e4119800 JavaThread "Sweeper thread" daemon [_thread_blocked, id=12859, stack(0x00007f75958d9000,0x00007f75959da000)]                                                                            
  0x00007f75e4117800 JavaThread "C1 CompilerThread11" daemon [_thread_blocked, id=12858, stack(0x00007f75959da000,0x00007f7595adb000)]                                                                       
  0x00007f75e4115000 JavaThread "C1 CompilerThread10" daemon [_thread_blocked, id=12857, stack(0x00007f7595adb000,0x00007f7595bdc000)]                                                                       
  0x00007f75e4113000 JavaThread "C1 CompilerThread9" daemon [_thread_blocked, id=12856, stack(0x00007f7595bdc000,0x00007f7595cdd000)]                                                                          0x00007f75e4110800 JavaThread "C1 CompilerThread8" daemon [_thread_blocked, id=12855, stack(0x00007f7595cdd000,0x00007f7595dde000)]                                                                        
  0x00007f75e410e000 JavaThread "C2 CompilerThread7" daemon [_thread_blocked, id=12854, stack(0x00007f7595dde000,0x00007f7595edf000)]                                                                        
  0x00007f75e410c000 JavaThread "C2 CompilerThread6" daemon [_thread_blocked, id=12853, stack(0x00007f7595edf000,0x00007f7595fe0000)]                                                                        
  0x00007f75e4109800 JavaThread "C2 CompilerThread5" daemon [_thread_blocked, id=12852, stack(0x00007f7595fe0000,0x00007f75960e1000)]                                                                        
  0x00007f75e40ff000 JavaThread "C2 CompilerThread4" daemon [_thread_blocked, id=12851, stack(0x00007f75960e1000,0x00007f75961e2000)]                                                                        
  0x00007f75e40fd000 JavaThread "C2 CompilerThread3" daemon [_thread_blocked, id=12850, stack(0x00007f75961e2000,0x00007f75962e3000)]                                                                        
  0x00007f75e40fb000 JavaThread "C2 CompilerThread2" daemon [_thread_blocked, id=12849, stack(0x00007f75962e3000,0x00007f75963e4000)]                                                                        
  0x00007f75e40f7800 JavaThread "C2 CompilerThread1" daemon [_thread_blocked, id=12848, stack(0x00007f75963e4000,0x00007f75964e5000)]                                                                        
  0x00007f75e40f5800 JavaThread "C2 CompilerThread0" daemon [_thread_blocked, id=12847, stack(0x00007f75964e5000,0x00007f75965e6000)]                                                                        
  0x00007f75e40f3800 JavaThread "Signal Dispatcher" daemon [_thread_blocked, id=12846, stack(0x00007f75965e6000,0x00007f75966e7000)]                                                                           0x00007f75e40c4800 JavaThread "Finalizer" daemon [_thread_blocked, id=12845, stack(0x00007f75966e7000,0x00007f75967e8000)]                                                                                   0x00007f75e40c2000 JavaThread "Reference Handler" daemon [_thread_blocked, id=12844, stack(0x00007f75967e8000,0x00007f75968e9000)]                                                                         
  0x00007f75e400e800 JavaThread "main" [_thread_blocked, id=12824, stack(0x00007f75e9143000,0x00007f75e9244000)]                                                                                                                                                                                                                                                                                                          Other Threads:                                                                                                                                                                                               
=>0x00007f75e40b6000 VMThread [stack: 0x00007f75968e9000,0x00007f75969ea000] [id=12843]                                                                                                                      
  0x00007f75e4123800 WatcherThread [stack: 0x00007f75956d7000,0x00007f75957d8000] [id=12861]                                                                                                                 
                                                                                                                                                                                                             VM state:at safepoint (normal execution)                                                                                                                                                                     
                                                                                                                                                                                                             VM Mutex/Monitor currently owned by a thread:  ([mutex/lock_event])                                                                                                                                          
[0x00007f75e400be20] Threads_lock - owner thread: 0x00007f75e40b6000                                                                                                                                         
[0x00007f75e400c3c0] Heap_lock - owner thread: 0x00007f75e4181800 
```

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
