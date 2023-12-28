---
mindmap-plugin: basic
tags:
  - java
---

# Java Classes

## Access Control
- | Accessed modifiers/Access in | Same class | Same pkg > Subclass | Same pkg > Non-Subclass | Diff pkg > Subclass | Diff pkg > Non-Subclass |
|---|---|---|---|---|---|
| Public | ✅  | ✅  | ✅  | ✅  | ✅  |
| Protected | ✅  | ✅  | ✅  | ✅  | ❌ |
| Default | ✅  | ✅  | ✅  | ❌ | ❌ |
| Private | ✅  | ❌ | ❌ | ❌ |❌|

## `final`
- Applied to variables/class fields
	- Prevent them from being changed (essentially make them constant).
- Applied to class methods
	- Disallow them from being overridden.
	- Methods declared as final can sometimes provide performance enhancement.
		- The compiler is free to inline calls them (i.e. copies its bytecode directly to the bytecode of the caller) cause it knows they won't be overridden by subclasses.
		- This is called **early binding**: method calls can be resolved at compile time.
		- This is opposite to **late binding**: methods calls are resolved dynamically at runtime.
- Applied to class
	- Prevent a class from being inherited.
	- This implicitly declares all its methods as final too.
	- It's illegal to declare a class as both `final` and `abstract` since an abstract class needs subclasses to complete its implementation.

## Nested Classes
- Since Java 1.1.
- A class declared within another class.
- A nested class declared directly within its enclosing class scope is a member of its enclosing class.
- It is also possible to declare a nested class that is local to a block.
- Access
	- A nested class can access its enclosing class's members (including private).
	- The enclosing class does not have access to its nested class's members.
- Types
	- Static nested class
		- Has `static` modifier.
		- Can't refer to its enclosing class's non-static members directly, has to do so through an object.
	- Non-static nested class (**inner class**)
		- Has direct access to its enclosing class's members.
		- **Anonymous inner class**
			- Inner classes that don't have name.
- An instance of a nested class can only be created in the context of its enclosing class.
	- Therefore in general, an inner class instance is often created by code within its enclosing class.

## The **Object** class
- All other classes are subclasses of Object.
- Methods
	- `int hashCode()`
		- Used to support hash tables such as `HashMap`.
		- The hashCode method defined by Object does return distinct integers for distinct objects.
			- This is typically implemented by converting internal address of the object to an integer.
		- General contracts for implementing (overriding) hashCode
			- During an execution of an Java application, hashCode must consistently return same integer (provided no information used in equals is modified).
				- This integer need not remain consistent from 1 execution to another one of the same application.
			- If 2 objects are equal according to `equals(Object)`, hashCode must produce same integer for both.
			- The opposite is not required: if 2 objects are unequal according to `equals(Object)`, hashCode is not required to produce distinct integeres.
				- However, producing distinct integers for unequal objects may improve the performance of hash tables.
	- `boolean equals(Object)`
		- Contracts for implementing (overriding) equals
			- It implements an equivalence relation on non-null object references
				- *reflexive*
					- x.equals(x) = true ∀ x ≠ null
				- *symmetric*
					- x.equals(y) = true ⇔ y.equals(x) = true ∀ x, y ≠ null
				- *transitive*
					- x.equals(y) = true, y.equals(z) = true ⇒ x.equals(z) = true ∀ x, y, z ≠ null
				- *consistent*
					- x.equals(y) consistently returns true (or false) over multiple invocations (provided no information used in equals is modified) ∀ x, y ≠ null
				- x.equals(null) = false ∀ x ≠ null
			- In general, it necessary to override hashCode whenever equals is overridden to maintain the contract between the 2.
		- The equals method defined by Object implements "strictest and the most and precise" possible equivalence relation on objects: x.equals(y) = true ⇔ x == y is true, i.e. x and y refer to the same object.

## Interface
- Variables defined inside interface declarations are implicitly `final` and `static`.
	- ==You can use (no-method) interfaces to import shared constants into multiple classes==.
	- When a class implements an interface, all variables defined in the interface will be in the class's scope as constants.
- All methods and variables are implicitly `public`, methods must keep being declared `public` in implementation classes.
- If a class does not fully implement the methods required by the interface, it must be declared as `abstract`.
- **Nested interface** (Member interface)
	- An interface declared  as a member of a class or another interface.
	- A nested interface can be declared as `public`, `private`, or `protected`.
- A (top level) interface must either be declared as `public` or use default access level.
- **Default Interface Methods** (Java 1.8)
	- A default method defines a default implementation for an interface method.
	- Primary motivation was to expand interfaces (adding new methods) without breaking existing implementations.
	- Default method is declared using `default` keyword
- **Static Interface Methods** (Java 1.8)
- **Private Interface Methods** (Java 1.9)
	- Can only be called by default interface methods or another private interface methods defined in the same interface.
	- Private Interface Methods are not inherited by subinterfaces.