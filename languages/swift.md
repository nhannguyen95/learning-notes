---
mindmap-plugin: basic
tags:
  - swift
---

# Swift

## Fundamental data types
- Integers
	- `Int8`, `Int16`, `Int32`, `Int64`: signed
	- `UInt8`, `UInt16`, `UInt32`, `UInt64`: unsigned
	- `Int`: `Int32` on 32-bit platform, `Int64` on 64-bit platform.
	- `UInt`: `Uint32` on on 32-bit platform, `UInt64` on 64-bit platform.
	- Bounds: `Int8.min`, `Int8.max`, etc.
	- Literal values
		- Decimal: no prefix
		- Binary: `0b` prefix
		- Octal: `0o` prefix
		- Hexa: `0x` prefix
	- Type conversion
		- E.g. `UInt16(n)`
		- Floating-points are always truncated when converted to ints (`Double(-3.14)` = -3)
- Floating-point numbers
	- `Float`
		- 32-bit floating-point number.
		- Precision of as little as 6 decimal digits.
	- `Double`
		- 64-bit floating-point number.
		- Precision of at least 15 decimal digits.
		- In situations where either type would be appropriate, `Double` is preferred.
	- Type conversion
		- E.g. `Double(3)`
- Booleans
	- `Bool`
	- Swift's type safety prevents non-Boolean values from being substituted for `Bool`: `let i = 1; if i {}` causes compile error.
- Tuples
	- Values within a tuple can be of any type and don't have to be of the same type as each other.
	- Tuple decomposing
		- `let (x, y) = tuple`
		- Ignore parts of tuple with `_`: `let (_, y) = tuple`
	- Accessing element in tuple using 0-based index: `tuple.0`
	- Naming element in tuple
		- `let tuple = (x: 1, y: 2)`
		- You can then access element by its name: `tuple.x`
- Optionals
	- You set optional variables to a valueless state by assigning it `nil`.
	- You can use `nil` with non-optional constants or variables.
	- In Swift, `nil` isn't a pointer, it represents the absence of value of a certain type.
		- Optionals of any type can be set to `nil`, not just object types.
	- Optional binding
		- Used to find out if an optional contains a value.
		- Can be used with `if`, `guard`, `while`.
		- You can just write the name of the constant or variable to unwrap it: `if let myNumber {}`
		- You can include as many optional bindings and Boolean conditions in a single boolean-evaluating statement, separated by `,`
			- `,` will act like `&&`, optional bindings resolve to true if it contains a value.
		- You can provide a default value of optionals using nil coalescing operator `??`
		- When `nil` represents an unrecoverable failure, you can force unwrap an optional with `!`
			- `let x = Int(y); let z = x!`
			- If the optional contains `nil`, a runtime error is thrown.
		- Implicitly unwrapped optionals
			- Sub title

## Type inference
- `42` is inferred to be of type `Int`.
- `3.14` is inferred to be of type `Double`.