---

mindmap-plugin: basic

---

# Java Concurrency

## #process-vs-thread
- #process
	- Isolated, independently executing program
- #thread
	- Sometimes called #lightweight-process
	- A process can spawn multiple threads
	- Same-process threads share process-wide resources (e.g. memory address space)
	- Each thread has its own program counter, stack, local vars
	- Basic unit of scheduling in modern OSs

## References
- #java-concurrency-in-practice-2006

## Threads' risks
- Safety hazards
	- Safety = nothing bad ever happens
	- Examples
		- #race-condition
			- Occurs when the correctness of a computation depends on the relative timing of interleaving of multiple threads.
			- Types
				- #check-then-act - when a potentially stale observation is used to make a decision on what to do next.
					- Common example: lazy initialization (`getInstance` method in singleton pattern)
				- #read-modify-write
			- To prevent, we need #atomicity
				- Ops A and B (can be same) are atomic with respect to each other if, from the perspective of a thread executing A, when another one executes B, either all or none of B has executed.
				- Atomicity can be achieved by Java's `synchronized`.
- Liveness hazards
	- Liveness = something good eventually happens
	- Examples
		- #deadlock
		- #starvation
		- #livelock
- Performance hazards
	- Performance = good thing happens quickly
	- Examples
		- #context-switch
			- The scheduler suspends active thread and runs next one
			- Costs
				- Saving/restoring execution context
				- Loss of locality
				- Extra CPU time spent on scheduling
				- Synchronous mechanisms can inhibit compiler optimizations

## Thread safety
- A class is #thread-safe if it behaves correctly when accessing from multiple threads, with no additional synchronization or other coordination on the part of calling code.
- Stateless objects (objects created from classes that have no member variables) are always thread-safe.

## Java concurrency tools
- Locking / #synchronized block

	-
	  ```java
	  synchronized (lock) {
	  // Block of code guarded by the lock object.
	  }
	  
	  // Synchronize the whole method
	  synchronized (lock) void func();
	  ```

	- #intrinsic-locks (or monitor locks)
		- `lock` is left empty.
		- On the synchronized block enter: the object on which the method is being invoked is acquired by the executing thread as the lock.
		- On block exit (normal control path or exception throw): the lock is automatically released.
		- Intrinsic locks act as #mutexes (mutual exclusion)
			- At most one thread may own the lock.
			- When a thread attempts to acquire a lock, it must wait until the lock is released (by another thread acquiring it).
		- Intrinsic locks are #reentrant
			- Locks are acquired on a per-thread (rather than per-invocation) basis.
			- This means if a thread requests a lock it already hold, the request succeeds.
			- Reentrancy saves us from some deadlock situations (e.g. recursion, override synced method calls to its super, also synced method, etc.)
	- #visibility
	- Usage
		- Compound actions (check-then-act, read-modify-write) on shared state (an object's member variables, or a group of them) must be made atomic to avoid race condition.
		- #memory-visibility
			- Without synchronization, there is no guarantee that the reading thread will see a value written by another thread on a timely basis, or even at all.
			- This is because without synchronization, compiler/processor/runtime can do some weird things to optimize the performance.
			- Solution: synchronization needs to be used not only when writing to a shared (object's member) variable, but also everywhere that variable is accessed. The synchronization must be used with the **same lock**. We say that the variable is guarded by that lock.
			- This way, locking ensures all threads see the most up-to-date values of shared mutual variables.
		- Every shared, mutable variable should be guarded by exactly one lock.
		- For every invariant (object's state) that involves more than 1 variable, all that involved variables must be guarded by the same lock.
		- Making sure synchronized blocks are short enough to achieve a balance between simplicity (synchronizing all code paths) and concurrency performance (synchronizing shortest possible code paths since there's overhead of acquiring and releasing locks).
		- Avoid holding locks for blocks whose computations or operations at risk of not completing quickly.
- #volatile variables
	- `volatile` is an alternative, weaker form of synchronization/locking than `synchronized`.
	- Locking can guarantee both visibility and atomicity, volatile variables can only guarantee visibility.
		- Visibility
			- Compiler/runtime put on notice that a variable declared with `volatile` is shared and operations on it should not be reordered with other memory operations.
			- Volatile variables are not cached in registers or in caches where they can be stale.
			- This means a read of a volatile variable always return the most recent write by any thread.
		- No atomicity
			- Accessing a volatile variable performs no locking, so cannot block the executing thread.
			- This means with volatile `count`, `count++` is not atomic.
	- On most current processor architectures, volatile reads are only slightly more expensive than nonvolatile ones.
	- Usage
		- Using volatile variables for status flags.
			- State information of objects.
			- Indicating object's important life-cycle events (shutdown, initialization) has occurred.
		- Can be used when all these criteria are met
			- Writes to the variable do not depend on its current value.
			- The variable does not participate in invariants (invariant = object state) with other state variables.
			- Locking is not required while the variable is being accessed.
- #thread-confinement
	- If data is only accessed from a single thread, no synchronization is needed.
	- If an object is confined to a thread, its usage is automatically thread-safe even if the object is not.
	- Thread confinement is one of the simplest ways to achieve thread safety.
	- Achieving thread confinement
		- Ad-hoc thread confinement
			- The implementation (not any library) is responsible for maintaining thread confinement.
			- Should be used sparingly due to its fragility.
		- #stack-confinement
			- Each thread has their own stack, so threads do not share their local variables.
			- This means local variables are intrinsically confined to the executing thread.
			- As a consequence, if an object can only be reached through local variables, it is confined to the thread executing those variables.
			- This is called stack confinement (or #within-thread, #thread-local, not to be confused with `ThreadLocal` library), it is a special case of thread confinement.
			- Stack confinement is simpler to maintain and less fragile than ad-hoc thread confinement.
			- Example

				-
				  ```java
				  private long numberOfPeopleNamedJohn(List<Person> people) {
				  // We're passing a list of person but do not directly use it.
				  
				  // Instead, we create our own list that is local to the
				  // executing thread.
				  List<Person> localPeople = new ArrayList<>();
				  
				  // Using the local list is thread-safe (even if its implementation
				  // might not), we just need to make sure
				  // it won't escape the method scope.
				  localPeople.addAll(people);
				  
				  // Note that it is the List<Person> is stack confined, not the
				  // People elements it contains.
				  
				  // Stack confinement implies a need of a deep copy
				  // (in this case from people list to localPeople list) (?).
				  
				  return localPeople
				  .stream()
				  .filter(person -> person.getFirstName().equals("John"))
				  .count();
				  }
				  ```

		- `ThreadLocal`
			- Allows you to associate a per-thread value (`T`) with a value-holding object (`ThreadLocal<T>`).
			- You can think of a `ThreadLocal<T>` as holding a `Map<Thread, T>` that stores the thread-specific value.
			- `ThreadLocal` provides `get` and `set` methods such that `get` returns the most recent value passed to set from the currently executing thread.
			- It is a more formal means of maintaining thread confinement.