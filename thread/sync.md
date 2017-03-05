
## Mark Word

<img src="https://snag.gy/ZgtiqF.jpg" width=500>

## Light-weight Locking

- When object is locked, mark word copied into lock record in frame on thread stack, aka, displaced mark.
- Use atomic compare-and-swap (CAS) to attempt to make mark word point to lock record
  - If CAS succeeds, thread owns lock
  - If CAS fails, contention: lock inflated (made heavyweight)
- Lock records track objects locked by currently executing methods
  - Walk thread stack to find thread's locked objects
- Effective because most locking is uncontended


## Heavy-weight Locking
- OS lock, each object is associated with a [Monitor](https://github.com/runjvm/hotspot-notes/blob/master/java/monitor.md)

## Biased Locking

Since most objects are locked by at most one thread during their lifetime, we allow that thread to bias an object toward itself. Once biased, that thread can subsequently lock and unlock the object without resorting to expensive atomic instructions. Obviously, an object can be biased toward at most one thread at any given time. (We refer to that thread as the bias holding thread). If another thread tries to acquire a biased object, however, we need to revoke the bias from the original thread. (At this juncture we can either rebias the object or simply revert to normal locking for the remainder of the object's lifetime). The key challenge in revocation is to coordinate the revoker and the revokee (the bias holding thread) -- we must ensure that the revokee doesn't lock or unlock the object during revocation. As noted in the paper, revocation can be implemented in various ways - signals, suspension, and safepoints to name a few. Critically, for biased locking to prove profitable, the benefit conferred from avoiding the atomic instructions must exceed the revocation costs. *Another equivalent way to conceptualize biased locking is that the original owner simply defers unlocking the object until it encounters contention*. Biased locking bears some semblance to the concept of lock reservation, which is well-described in Kawachiya's dissertation. It's also similar in spirit to "opportunistic locks" (oplocks) found in various file systems; the scheme describe in Mike Burrows's "How to implement unnecessary mutexes" (In Computer Systems: Theory, Technology and Applications, Dec. 2003); or Christian Siebert's one-sided mutual exclusion primitives.

- [https://blogs.oracle.com/dave/entry/biased_locking_in_hotspot]

In certain circumstances, it is better to deactivate this kind of optimization: -XX:-UseBiasedLocking

Please note also that BiasedLocking is enabled only 4 seconds after startup. You can tune it by -XX:BiasedLockingStartupDelay=4000


## Bulk Rebiasing and Revocation
- Detect if many revocations occurring for a given data type
- Try invalidating all biases for objects of that type

## Reference
- [https://www.cs.princeton.edu/picasso/mats/HotspotOverview.pdf]
- [http://ds.cs.ut.ee/courses/course-files/lockOptimizationHotSpot.pdf]
