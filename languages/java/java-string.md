---
mindmap-plugin: basic
tags:
  - java
  - java-string
---

# Java String

## `StringBuffer` (since 1.0)
- `StringBuffer` represents growable and writable character sequences.
- ==It's like a `String`, but can be modified==
- Constructors
	- `StringBuffer()`
		- Empty string buffer with 16-character initial capacity.
	- `StringBuffer(int capacity)`
		- Empty string buffer with specified initial capacity.
	- `StringBuffer(String str)`
		- String buffer initialized to content of the specified string, and reserve capactiy for 16 more characters.
	- `StringBuffer(CharSequence chars)`
		- String buffer initialized to characters in specified char sequence, and reserve capacity for 16 more characters.
- Methods
	- `int length()`
	- `void setLength(int len)`
		- If less than current length, characters beyond the new length will be lost.
		- If greater than current length, null characters are added to the end.
	- `int capacity()`
	- `void ensureCapacity(int minCapacity)`
		- Preallocate capacity after a `StringBuffer` has been constructed.
		- Useful if you know in advance that you will be appending a large number of small strings, avoid reallocations.
	- `char charAt(int i)`
	- `void setCharAt(int i, char c)`
		- 0 <= i < length()
	- Appends
		- `StringBuffer append(String)`
		- `StringBuffer append(Primitive)`
		- etc.
		- `StringBuffer` is returned to allow subsequent calls to be chained together.
	- Inserts
		- `StringBuffer insert(int index, String str)`
			- `str` will start at index `index` in the new string.
			- 0 <= index <= length()
		- `StringBuffer insert(int index, Primitive)`
	- Deletes
		- `StringBuffer delete(int startIndex, int endIndex)`
		- `StringBuffer deleteCharAt(int i)`
	- `StringBuffer reverse()`
	- `StringBuffer replace(int startIndex, int endIndex, String str)`
	- `String substring(int startIndex)`
	- `String substring(int startIndex, int endIndex)`
	- etc.

## `String` (since 1.0)
- Unlike other languages that implement strings as character arrays, Java implements string as objects of type `String`.
- `String` represents ==fixed-length, immutable character sequence==.
- Methods
	- Character extraction
		- `char charAt(int i)`
		- `char[] toCharArray()`
	- Comparison
		- `boolean equals(String str)`
			- `equals` compare characters inside a `String` object.
			- `==` compares 2 object references.
		- `boolean equalsIgnoreCase(String str)`
		- `int compareTo(String str)`
			- `0`: equals
			- `<0`: the invoking string is less than `str`.
			- `>0`: the invoking string is greater than `str`.
		- `boolean startsWith(String str)`
		- `boolean startsWith(String str, int startIndex)`
		- `boolean endsWith(String str)`
	- Searching
		- `int indexOf(String str)`
			- Search for first occ
		- `int lastIndexOf(String str)`
			- Search for last occ
	- Modification
		- Since you can't change the content of a `String` object; these methods returns the new, modified one.
			- `String substring(int startIndex)`
			- `String substring(int startIndex, int endIndex)`
			- `String concat(String str)`
			- `String replace(CharSequence original, CharSequence replacement)`
			- `String trim()`
				- Removes leading, trailing spaces.
			- `String strip()`: since Java 11
				- Removes leading, trailing whitespace characters (spaces, tabs, returns, line feeds)
			- `String toLowerCase()`
			- `String toUpperCase()`
			- `static String join(CharSequence delim, CharSequence ...strs)`
			- `String split(String regexp)`
	- Creating strings
		- Using constructors
			- `String()`
			- `String(char[])`
				- Used to create strings containing 16-bit basic Unicode characters.
			- `String(byte[])`
				- Used to create strings containing 8-bit ASCII characters (with default encoding).
			- `String(String)`
		- Using string literals
			- `String s = "abc"`
	- etc.

## `StringBuilder` (since 1.5)
- Similar to `StringBuffer` except for one important difference: it is ==not synchronized (not thread-safe)==.
	- Can be used the string is accessed by only 1 thread.
	- Advantage: faster performance.

## Performance notes
- The task is to concat a large number of strings in a loop
- The most optimal way is to use `StringBuilder` with pre-supplied capacity.

	-
	  ```java
	  // n = total length of result
	  StringBuilder sb = new StringBuilder(n);
	  for(String string : strings) {
	  sb.append(string);
	  }
	  String result = sb.toString();
	  ```

	- This solution is performant in terms of running time and memory used since the `sb` is modified in place.
- A worse solution is to use string concatenation (`+` operator or `concat` method)

	-
	  ```java
	  String result = "";
	  for(String string : strings) {
	  result += string;
	  }

	- This is worse since `String` objects are mutable, so in order to concat 2 strings, runtime has to create a new `String` object and copy the 2 strings over.
	- Since Java 1.5, with the introduction of `StringBuilder`, the `result += string` is compiled to `result = new StringBuilder().append(result).append(string).toString()`. This is still worse than using one `StringBuilder` due to overhead of creating and operating on a new `StringBuilder` instance on every loop.