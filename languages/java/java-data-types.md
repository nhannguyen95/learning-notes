---
mindmap-plugin: basic
tags:
  - java
---

# Java Data Types

## Primitive Types
- Integers
	- `byte`
		- 8 bits
		- Useful when working with stream of raw binary data from a network or file.
	- `short`
		- 16 bits
		- Probably the least used Java type.
	- `int`
		- 32 bits
	- `long`
		- 64 bits
	- Java do not support unsigned integers.
	- Literals
		- Decimals: no prefix, no leading zero.
		- Octal: `0` prefix.
		- Hexa: `0x` or `0X` prefix.
		- Binary: `0b` or `0B` prefix.
		- `long`: `l` or `L` suffix.
- Floating-point numbers
	- `float`
		- 32 bits
	- `double`
		- 64 bits
	- Literals
		- Double: no suffix nor prefix (optional `D` or `d` suffix).
		- Float: `F` or `f` suffix.
- Characters
	- `char`
		- Designed to hold Unicode characters.
- Boolean
	- `boolean`

## Arrays
- Default value of arrays when created with `new`
	- 0 for numeric types.
	- false for booleans.
	- null for reference types.
- When using *array initializer* when creating arrays, `new` is not needed.
	- e.g. `int[] arr = {1, 2, 3};`

## Primitive Wrappers
- `Byte`, `Short`, `Integer`, `Long`, `Float`, `Double`, `Character`, `Boolean`
- Numerics common (except `Boolean`)
	- Static fields

		-
		  ```java
		  MIN_VALUE
		  MAX_VALUE
		  
		  // Number of bytes, bits used to represent this value.
		  BYTES
		  SIZE
		  
		  // Only on floating-point types.
		  // Represent +oo (1.0f / 0.0f, 1.0 / 0.0)
		  POSITIVE_INFINITY
		  // Represent -oo (-1.0f / 0.0f, -1.0 / 0.0)
		  NEGATIVE_INFINITY
		  ```

	- Static methods

		-
		  ```java
		  // E.g. parseInt("1"). Not on boolean and character.
		  primitive parseprimitive(String);
		  // Not on boolean, character and floating-point.
		  primitive parseprimitive(String, int radixOfString);
		  
		  // Not on boolean and character.
		  Primitive valueOf(String)
		  // Not on boolean, character and floating-point.
		  Primitive valueOf(String, int radixOfString);
		  ```

	- Methods

		-
		  ```java
		  // E.g. intValue(). Returns the underlying primitive
		  primitive primitiveValue();
		  ```

- `Character`
	- Static methods

		-
		  ```java
		  boolean isDigit(char/int)
		  boolean isLetter(char/int)
		  boolean isLetterOrDigit(char/int)
		  
		  boolean isLowerCase(char/int)
		  boolean isUpperCase(char/int)
		  ```