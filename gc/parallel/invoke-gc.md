```c++
bool PSScavenge::invoke() {
  assert(SafepointSynchronize::is_at_safepoint(), "should be at safepoint");
  assert(Thread::current() == (Thread*)VMThread::vm_thread(), "should be in vm thread");
  assert(!ParallelScavengeHeap::heap()->is_gc_active(), "not reentrant");

  if (ScavengeAtRegularIntervals) {
    RegularScavenge::set_collecting(true);
  }

  ParallelScavengeHeap* const heap = ParallelScavengeHeap::heap();
  PSAdaptiveSizePolicy* policy = heap->size_policy();
  IsGCActiveMark mark;

  const bool scavenge_done = PSScavenge::invoke_no_policy(); // invoke Minor GC
  const bool need_full_gc = !scavenge_done ||
    policy->should_full_GC(heap->old_gen()->free_in_bytes());
  bool full_gc_done = false;

  if (UsePerfData) {
    PSGCAdaptivePolicyCounters* const counters = heap->gc_policy_counters();
    const int ffs_val = need_full_gc ? full_follows_scavenge : not_skipped;
    counters->update_full_follows_scavenge(ffs_val);
  }

  if (need_full_gc) {
    GCCauseSetter gccs(heap, GCCause::_adaptive_size_policy);
    CollectorPolicy* cp = heap->collector_policy();
    const bool clear_all_softrefs = cp->should_clear_all_soft_refs();

    if (UseParallelOldGC) { // Default enabled when UseParallelGC is enabled
      tty->print_cr("old");
      full_gc_done = PSParallelCompact::invoke_no_policy(clear_all_softrefs);
    } else {
      tty->print_cr("mark sweep");
      full_gc_done = PSMarkSweep::invoke_no_policy(clear_all_softrefs);
    }
  }

  return full_gc_done;
}
```
