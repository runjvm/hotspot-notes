
- A thread can be at a safepoint or not be at a safepoint. When at a safepoint, the thread's representation of it's Java machine state is well described, and can be safely manipulated and observed by other threads in the JVM.
- "Being at a safepoint" does not mean "being blocked" (e.g. JNI code runs at a safepoint), but "being blocked" always results a safepoint.
- 3. The JVM may choose to reach a global safepoint (aka Stop-The-World), where all threads are at a safepoint and can't leave the safe point until the JVM decides to let it do so. This is useful for doing all sorts of work (like certain GC operations, deoptimization during class loading, etc.) that require ALL threads to be at a well described state.

4. Some JVMs can bring individual threads to safepoint without requiring a global safepoint. E.g. Zing uses the term Checkpoint (first published in [1]) to describe a JVM mechanism that individually passes threads through thread-specidfic safepoints to perform certain very short operations on individual thread state without requiring a Stop-The-Wolrd pause.

7. All [practical] JVMs apply some highly efficient mechanism for frequently crossing safepoint opportunities, where the thread does not actually enter a safepoint unless someone else indicates the need to do so. E.g. most call sites and loop backedges in generated code will include some sort of safepoint polling sequence that amounts to "do I need to go to a safepoint now?". Many HotSpot variants (OpenJDK and Oracle JDK) currently use a simple global "go to safepoint" indicator in the form of a page that is protected when a safepoint is needed, and unprotected otherwise. The safepoint polling for this mechanism amounts to a load from a fixed address in that page. If the load traps with a SEGV, the thread knows it needs to go to enter a safepoint. Zing uses a different, per-thread go-to-safepoint indicator of similar efficiency.

8. All JNI code executes at a safepoint. No Java machine state of the executing thread can be changed or observed by it's JNI code while at a safepoint. Any manipulation or observation of Java state done by JNI code is achieved via JNI API calls that leave the safepoint for the duration of the API call, and then enter a safepoint again before returning to the calling JNI code. This "crossing of the JNI line" is where most of the JNI call and API overhead lies, but it's fairly quick (entering and leaving safepoint normally amounts to some CAS operations).

For compiled code, JIT inserts safepoint checks in code at certain points (usually, after return from calls or at back jump of loop). For interpreted code, JVM have two byte code dispatch tables and if safepoint is required, JVM switches tables to enable safepoint check.
Safepoint status check itself is implemented in very cunning way. Normal memory variable check would require expensive memory barriers. Though, safepoint check is implemented as memory reads a barrier. Then safepoint is required, JVM unmaps page with that address provoking page fault on application thread (which is handled by JVM’s handler). This way, HotSpot maintains its JITed code CPU pipeline friendly, yet ensures correct memory semantic (page unmap is forcing memory barrier to processing cores).

## Reference
- [http://blog.ragozin.info/2012/10/safepoints-in-hotspot-jvm.html](http://blog.ragozin.info/2012/10/safepoints-in-hotspot-jvm.html)
- [http://chriskirk.blogspot.com/2013/09/what-is-java-safepoint.html](http://chriskirk.blogspot.com/2013/09/what-is-java-safepoint.html)

