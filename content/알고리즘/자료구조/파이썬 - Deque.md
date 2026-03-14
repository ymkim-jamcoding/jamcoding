---
description:
aliases:
created: 2026-03-14
modified: 2026-03-14
---

- A deque stands for **Double-Ended Queue**. It is a special type of data structure that allows you to add and remove elements from both ends efficiently. This makes it useful in applications like task scheduling, sliding window problems and real-time data processing.
- 시간 복잡도
	- 삽입
		- `append(x)` - $O(1)$
			- Add _x_ to the right side of the deque.
		- `appendleft(x)` - $O(1)$
			- Add _x_ to the left side of the deque.
		- `insert(i, x)` - $O(n)$
			- Insert _x_ into the deque at position _i_.
	- 삭제
		- `pop()` - $O(1)$
			- Remove and return an element from the right side of the deque.
		- `popleft()` - $O(1)$
			- Remove and return an element from the left side of the deque.
		- `remove(x)` - $O(n)$
			- Remove the first occurrence of _x_.
	- 접근
		- `de[0]` - $O(1)$
			- access the first element.
		- `de[-1]` - $O(1)$
			- access the last element.
		- `de[n]` - $O(n)$
			- $O(n)$ in the middle. For fast random access, use lists instead.