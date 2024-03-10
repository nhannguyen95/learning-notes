---

mindmap-plugin: basic

---

# Java Concurrency

## Fundamentals
- Thread
	- #process-vs-thread
		- #process
			- Isolated, independently executing program
		- #thread
			- Sometimes called #lightweight-process
			- A process can spawn multiple threads
			- Same-process threads share process-wide resources (e.g. memory address space)
			- Each thread has its own program counter, stack, local vars
			- Basic unit of scheduling in modern OSs
	- Threads' risks
		- Safety hazards
			- Safety = nothing bad ever happens
			- Examples
				- #race-condition
					- Occurs when the correctness of a computation depends on the relative timing of interleaving of multiple threads.
					- Types
						- #check-then-act - when a potentially stale observation is used to make a decision on what to do next.
							- Common example: lazy initialization (`getInstance` method in singleton pattern)
						- #read-modify-write
					- To prevent, we need atomicity
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
- Thread safety
	- Definition
		- A class is #thread-safe if it behaves correctly when accessing from multiple threads, with no additional synchronization or other coordination on the part of calling code.
		- Stateless objects (objects created from classes that have no member variables) are always thread-safe.
	- #atomicity
		- Ops A and B (can be same) are atomic with respect to each other if, from the perspective of a thread executing A, when another one executes B, either all or none of B has executed.
		- #locking - Java's builtin mechanism for ensuring atomicity

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
			- Usage
				- Compound actions (check-then-act, read-modify-write) on shared state (an object's member variables, or a group of them) must be made atomic to avoid race condition.
				- Every shared, mutable variable should be guarded by exactly one lock.
				- For every invariant (object's state) that involves more than 1 variable, all that involved variables must be guarded by the same lock.
				- Making sure synchronized blocks are short enough to achieve a balance between simplicity (synchronizing all code paths) and concurrency performance (synchronizing shortest possible code paths since there's overhead of acquiring and releasing locks).
				- Avoid holding locks for blocks whose computations or operations at risk of not completing quickly.
			- Locking is not just about mutual exclusion, it's also about memory visibility (see I.3.)
- Sharing objects
	- #memory-visibility
		- *Stale value* problem
			- Without synchronization, there is no guarantee that the reading thread will see a (new) value written by another thread on a timely basis, or even at all.
			- Even worse, the staleness is not all-or-nothing: the reading thread can see an up-to-date value of one variable, but a stale value of another variable that was written first
			- Stale values can cause serious safety or liveness failures
			- *Out-of-thin-air safety*
				- A thread when reads a variable without synchronization, is guaranteed to get a value that was previously written by another thread, not a random value
				- This safety guarantee applies to all variables, except 64-bit numeric variables
		- This is because without synchronization, compiler/processor/runtime can do some weird things to the execution order of operations to optimize the performance.
			- Attempt to reason about the execution order of operations in insufficiently synchronized multithreaded programs will almost certainly incorrect
		- Solution: synchronization needs to be used not only when writing to a shared (object's member) variable, but also everywhere that variable is accessed. The synchronization must be used with the *same lock*. We say that the variable is *guarded* by that lock.
		- This way, locking ensures all threads see the most up-to-date values of shared mutual variables. It is guaranteed that values written by one thread are made visible to other threads.
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
	- Publication and escape
		- *Publishing* an object means making it available outside of its current scope
			- Storing the object in a reference where other code can access it
			- Returning it from a non-private method
			- Passing it to a method in another class
		- We generally don't want to publish objects' internals
			- Publishing internal state variables can compromise encapsulation
			- Making it more difficult to preserve invariants
			- Publishing objects before they are fully constructed can compromise thread safety
				- E.g. [Letting `this` escape during construction with the use of inner class](https://stackoverflow.com/a/28678279)
				- Solution: safe construction - using a factory method to prevent `this` from escaping during construction
		- An object that is published when it should not have been is said to have *escaped*
			- Once an object is escaped, another class/thread may misuse it
	- #thread-confinement
		- Examples
			- In Swing, visual components and data model objects are not thread safe. Safety is achieved by confining them to the Swing event dispatch thread.
		- Introduction
			- If data is only accessed from a single thread, no synchronization is needed.
			- If an object is confined to a thread, its usage is automatically thread-safe even if the object is not.
			- Thread confinement is one of the simplest ways to achieve thread safety, and programmer is responsible for enforcing it.
		- Achieving thread confinement
			- Ad-hoc thread confinement
				- The implementation (not any library) is responsible for maintaining thread confinement.
				- Should be used sparingly due to its fragility.
				- Example: it's safe (no race condition) to read-modify-write a shared volatile variable if the modification is confined to a single thread.
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
				- Example
					- When porting a single-threaded app to a multi-threaded one, thread safety can be preserved by converting shared variables into `ThreadLocal`s.
	- Immutability
		- Immutable objects can't be modified (post construction), thus are always thread-safe.
		- Immutability is not formally defined in Java, a possible definition is:
			- It is properly constructed (`this` does not escape)
			- Its state cannot be modified post construction
			- All its field are `final`
				- This condition only might not make an object immutable, since a final field can hold a reference to a mutable object
				- Good practice: make fields `final` unless they need to be mutate
		- Practices
			- Whenever a group of related data must be acted on atomically, consider creating an immutable holder class for them