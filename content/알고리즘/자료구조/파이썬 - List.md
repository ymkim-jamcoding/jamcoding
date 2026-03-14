---
description:
aliases:
created: 2026-03-14
modified: 2026-03-14
---

- In Python, a list is a built-in data structure that can hold an ordered collection of items. Unlike arrays in some languages, Python lists are very flexible:
	- Can contain duplicate items
	- **Mutable:** items can be modified, replaced, or removed
	- **Ordered:** maintains the order in which items are added
	- **Index-based:** items are accessed using their position (starting from 0)
	- Can store mixed data types (integers, strings, booleans, even other lists)
- 시간 복잡도
	- 삽입
		- `append(x)` - $O(1)$
			- Add an item to the end of the list
		- `insert(i, x)` - $O(n)$
			- Insert an item at a given position.
	- 삭제
		- `remove(x)` - $O(n)$
			- Remove the first item from the list whose value is equal to _x_.
		- `pop()` - $O(1)$
			- removes and returns the last item in the list.
		- `pop(i)` - $O(n)$
			- Remove the item at the given position in the list, and return it.
		- `del L[i]`
	- 접근
		- `L[i]` - $O(1)$
			- random access