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
- Atomicity
	- #synchronized block

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
		- Locks usage
			- Compound actions (check-then-act, read-modify-write) on shared state (an object's member variables, or a group of them) must be made atomic to avoid race condition.
			- Synchronization needs to be used not only when writing to a shared (object's member) variable, but also everywhere that variable is accessed. The synchronization must be used with the same lock. We say that the variable is guarded by that lock.
			- Every shared, mutable variable should be guarded by exactly one lock.
			- For every invariant (object's state) that involves more than 1 variable, all that involved variables must be guarded by the same lock.