---

mindmap-plugin: basic

---

# Optional

## `java.util.Optional<T>`
- Origin
   - Introduced in Java 8
   - Inspired by Haskell and Scala
- Why?
   - More elegant way (than `null`) to model the presence/absence of a value
   - More readable API in which several ops are chained together (like in [[Stream]])
- Usage
   - Creation
      - From an empty obj
         - `Optional.empty()`
      - From a non-null value
         - `Optional.of(t)`
         - Throwing `NullPointerException` if `t` is `null`
      - From a possibly null value
         - `Optional.ofNullable(t)`
      - Think of `Optional<T>` as a `Stream<T>` with only 1 item
   - Reading wrapped value
      - `get()`
         - Return wrapped value if one is present, throw exp otherwise.
         - Simplest, least safe.
      - `orElse(T other)`
      - `orElseGet(Supplier<? extends T> supplier)`
         - Lazy version of `orElse`.
         - Supplier only gets invoked if value is not present.
      - `or(Supplier<? extends Optional<? extends T>> supplier)`
         - Lazily provides different Optional when original one is empty
      - `orElseThrow(Supplier<? extends X> exceptionSupplier)`
         - Similar to `get`, but with custom exp type.
      - `ifPresent(Consumer<? super T> consumer)`
      - Java 9: `ifPresentOrElse(Consumer<? super T> consumer, Runnable action)`
   - Patterns
      - `map`

         -
           ```java
           T t;
           Optional<T> optT = Optional.ofNullable(t);
           // If optT is not empty, mapper passed into map is invoked.
           Optional<String> name = optT.map(T::toString);
           
           // Instead of dealing directly with null:
           String name = null;
           if (t != null) name = t.toString();
           ```

      - `flatMap`

         -
           ```java
           String name =
           // using map would return Optional<Optional<U>>
           optT.flatMap(T::getU);
           .map(U::toString)
           .orElse("");
           
           // Instead of:
           U u = null;
           String name = null;
           if (t != null) {
           u = t.toString();
           if (u != null) {
           name = u.toString();
           }
           }
           ```

      - `stream`

         -
           ```java
           List<T> ts;
           
           // Goal: find a set of distinct name of U in ts.
           // Stream<T>
           ts.stream()
           // Stream<Optional<U>> (T's getU returns Optional<U>)
           .map(T::getU)
           // Stream<Optional<String>>
           .map(optU -> optU.map(U::toString)
           // Optional::stream returns 1-size Stream that contains the item if it is not null, empty Stream otherwise
           .flatMap(Optional::stream)
           .collect(toSet());
           ```

      - `filter`

         -
           ```java
           Optional.ofNullable(t)
           // return Optional<T> describe the item match the predicate,
           // empty Optional otherwise.
           .filter(t -> "abc".equals(t.toString()))
           .ifPresent(t -> {});
           
           // Instead of:
           T t;
           if (t != null && "abc".equals(t.toString()) {}
           ```


## Problems with `null`
- Breaking Java philosophy: hide pointers from dev
- Bloating the code with oftenly nested `null` checks

## References
- Modern Java in Action (2018)

## Tags
- #java
- #optional
- #modern-java-in-action-2018