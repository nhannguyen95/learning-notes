---
mindmap-plugin: basic
tags:
  - java
  - lambda
  - modern-java-in-action-2018
---

# Lambda

## Functional interface (FI)
- One abstract method interface
- A lambda = an implementation of a FI
- A lambda is used when a FI type is expected
- Common built-in FIs
   - `Predicate<T>`
   - `Consumer<T>`
   - `Function<T, R>`
   - `Supplier<T>`
   - `UnaryOperator<T>`
   - `BinaryOperator<T>`
   - `BiPredicate<T, U>`
   - `BiConsumer<T, U>`
   - `BiFunction<T, U, R>`
   - Primitive specialized FIs

## Vars capture
- Can
   - Instance vars
   - Static vars
   - Immutable local vars `(*)`
- Can’t
   - Mutable local vars `(*)`
- Why? `(*)`
   - Thread-unsafe
   - Lambdas on different threads can work on stale copies of same mutable local var

## Method references
- Shorthand
   - (args) -> ClassName.staticMethod(args) = ClassName::staticMethod
   - (arg0, rest) -> arg0.instanceMethod(rest) = ClassName::instanceMethod
   - (args) -> obj.instanceMethod(args) = obj::instanceMethod
- Constructor references
   - Signature: (args) -> ClassName = ClassName::new (assume args is ClassName’s constructor’s params)