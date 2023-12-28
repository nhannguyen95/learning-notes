---

mindmap-plugin: basic

tags:

- java

- java-util

---

# Java Operators

## Bitwise Operators
- How Java stores integers
	- All integer types (except `char`) are signed integers.
	- Negative integers are represented by 2's complement.
- Bitwise operators
	- `&` (and)
	- `|` (or)
	- `^` (xor)
	- `~` (not)
	- `<<` (left shift)
		- ==Left bits are lost after shifting==
			- For `int`/`long`, left bits are lost once they are shifted past bit position 31/63.
			- For `byte`/`short`, since they are promoted to `int` in expressions, shifted-left bits are not lost until they shift past bit 31.
				- This means when left-shift a `byte`, you must discard the top 3 bytes of the result if you want a shifted byte result.
				- This can easily be done by casting the result back to `byte`.
		- 0s are brought to the right
	- `>>` (right shift)
		- Right bits are lost after shifting
		- ==Left bits are filled with the previous value of the top bit==
			- E.g. 11111000 (-8) >> 1 = 11111100 (-4)
			- This is called ==sign extension== and serves to preserve the sign of negative numbers.
			- Note that as a result, -1 shifting right will always be -1.
	- `>>>` (unsigned shift right)
		- Left bits are always filled with 0s.
		- Used when the sign extension behavior of right shift is undesired.

## Relational Operators
- `==`, `!=`, `>`, `<`, `>=`, `<=`
- The outcome of these operations is a `boolean` value.

## Boolean Logical Operators
- `&`, `|`, `^`, `||` )(short-circuit), `&&` (short-circuit), `!`, `==`, `!=`, `?:`
- Operate only on `boolean` operands, resulting in a `boolean` value.

## Assignment Operator
- The result of an assignment operator is the value of the right-hand expression.