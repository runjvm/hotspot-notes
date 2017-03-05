
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
