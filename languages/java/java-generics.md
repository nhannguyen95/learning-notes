---
mindmap-plugin: basic
tags:
  - "#java"
---

# Java Generics

## How it works?
- Compilers does not create multiple versions of the generic class, it creates only one version.
- Compilers remove all generic type from the generic class, substituting the type, to make your code behave as if a specific version of the generic class were created. The removal process is called *erasure*.
- The type argument can't be primitive types.

## Syntax
- Declaring

	-
	  ```
	  class class-name<type-param-list > { // ...
	  ```

- Instantiating

	-
	  ```
	  class-name<type-arg-list> var-name = new class-name<type-arg-list >(cons-arg-list);
	  ```


## Bounded Types

-
  ```
  <T extends superclass>
  ```

	- This specifies that T can only be replaced by superclass, or subclasses of superclass.
- When specifying a bound that has a class and one or more interfaces, we can use `&` operator to connect them. This creates an *intersection type*.

	-
	  ```
	  // A bound can include both class and one or more interfaces.
	  // in this case, the class type must be specified first.
	  <T extends MyClass & MyInterface1 & MyInterface>
	  ```

- Wildcard Arguments
	- `MyClass<?>` matches any MyClass objects.
	- Bounded Wildcard Arguments
		- `<? extends superclass>`
			- Establish an inclusive upper bound for a wildcard
		- `<? super subclass>`
			- Establish an inclusive lower bound for a wildcard