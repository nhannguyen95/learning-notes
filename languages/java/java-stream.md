---

mindmap-plugin: basic

---

# Stream

## Streams vs Collections
- Collections
	- In-memory data structures
	- Elements are eagerly constructed
	- External iteration
- Streams
	- Elements are lazily constructed (on demand)
	- Can be consumed only once
	- Internal iteration

## Components
- Data source
	- Building streams
		- Finite streams
			- `Stream<T> Stream::of(T...)`
			- `PrimitiveStream Arrays::stream(primitive[])`
			- `Stream<T> Arrays::stream(T[])`
			- `Stream<T> Collection<T>.stream()`
		- Infinite streams
			- `Stream<T> Stream::iterate(T seed, T -> T)`
			- `Stream<T> Stream::generate(void -> T)`
- Intermediate ops
	- Not process the stream
	- Can be chained
	- Categories
		- Slicing
			- `dropWhile`
			- `takeWhile`
		- Filtering
			- `filter`
			- `distinct`
			- `limit`
				- Ordered streams: first N
				- Unordered streams: no order
			- `skip`
		- Mapping
			- `map`
			- `flatMap`
				- `List<Stream<T>>` -> `Stream<T>`
- Terminal ops
	- Trigger intermediate ops execution
	- Produce a non-stream value result
	- Categories
		- `forEach`
		- Matching
			- `allMatch`
			- `anyMatch`
			- `noneMatch`
			- Making use of short-circuiting
		- Finding
			- `findFirst`
			- `findAny`
		- Reducing
			- `collect(Collector)`
				- Traverse the stream, each element is process by Collector
				- Collectors
					- Categories
						- Reducing & summarizing
							- `toList` / `toSet` / `toCollection`
							- `maxBy` / `minBy`(Comparator)
							- summing/averaging/summarizing Int/Long/Double…(T -> int/long/double)
							- `counting`
							- `joining`
							- `reducing(U init, T -> U mapper, (U, U) -> U reducer)`
								- Generalized version of all the above
							- `collectingAndThen`
						- Grouping
							- `groupingBy`
								- 1-level grouping: groupingBy(classifier)
								- n-level grouping: groupingBy(classifier, `groupingBy(..)`)
								- Aggregating data in subgroups: `groupingBy(classifier, Collector)`
							- `partitioningBy`
								- Special case of grouping (2 groups)
								- More performant than `groupingBy` with internal specialized map impl
						- Custom Collector
							- Implement `Collector<T, A, R>`
					- characteristics
						- Gives hints about parallel potential and optimizations
						- Types
							- `UNORDERED`
								- Stream items traverse order won’t affect result
							- `CONCURRENT`
								- accumulator can be called concurrently from many threads
							- `IDENTITY_FINISH`
								- accumulator is the final result
			- `reduce(U init, (U, T) -> U acc, (U, U) -> U combiner)`
			- `count`

## Primitive specialized streams
- Builtin
	- `IntStream`
	- `DoubleStream`
	- `LongStream`
- Avoiding auto-boxing/auto-unboxing cost when using Stream<Integer/Double/Long,,,>

## Parallel streams
- `parallel()`
	- A flag is set to tell terminal ops to process the stream parallely
	- The stream is internally divided into many chunks
- Using effectively
	- Perf measure
	- Watch out for boxing
	- How well the stream's underlying data structure gets divided
		- `ArrayList`: excellent
		- `LinkedList`: poor
		- `IntStream.range`: excellent
		- `Stream.iterate`: poor
		- `HashSet`: good
		- `TreeSet`: good
	- Operations
		- Some perform worse
			- `limit`
			- `findFirst`
		- Some perform better
			- `findAny`
	- Not good for small data
	- Perf of terminal ops's merge step

## References
- Modern Java in Action (2018)

## Tags
- #java
- #stream
- #modern-java-in-action-2018