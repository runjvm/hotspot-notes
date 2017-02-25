A general reference on ["what kind of references are in root sets"](http://stackoverflow.com/questions/6366211/what-are-the-roots).

The reason I refered to these references as root sets instead of root set is that there are more than one set of these references, and a reference can be of more than one type. In HotSpot JVM, there are the following types of root sets.

      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::universe));
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::jni_handles));
      // We scan the thread roots in parallel
      Threads::create_thread_roots_tasks(q);
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::object_synchronizer));
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::flat_profiler));
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::management));
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::system_dictionary));
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::class_loader_data));
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::jvmti));
      q->enqueue(new ScavengeRootsTask(ScavengeRootsTask::code_cache));
