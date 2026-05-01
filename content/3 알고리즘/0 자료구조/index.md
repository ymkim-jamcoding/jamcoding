---
description:
aliases:
  - "- 0 자료구조"
created: 2025-09-10
modified: 2025-09-11
title: "- 0 자료구조"
status:
  - 🗺️
tags:
  - moc
---

# 자료구조?
- Data structures are specialized formats for organizing and storing data in a computer so that it can be used efficiently.
# 왜 중요한가
- Data structures are crucial in the field of computer science and coding because they offer a method of organizing and storing data in an efficient and manageable format. They're critical because they form the foundation for modern algorithm design. Your ability to choose or design the most suited data structure for a particular task can be the difference between a solution that's functional and efficient and one that isn't.

# 복잡도
- 시간 복잡도
	- 알고리즘이 문제를 해결하는 데 걸리는 **시간**을 입력 크기 n에 대한 함수로 나타낸 것
- 공간 복잡도
	- 알고리즘이 문제를 해결하는 데 필요한 **메모리 공간**을 입력 크기 n에 대한 함수로 나타낸 것
- 빅오 표기법
	- (최악의 경우를 산정)
	- ![[image-Data Structures.png]]
- 좋은 알고리즘은?
	- 시간-공간 복잡도가 낮고, 문제를 해결하는 알고리즘

# 기초 자료 구조
- Sequence Container
	- Array
	- Linked Lists
	- Stacks
	- Queues
	- Hash Tables
- 관련 알고리즘
	- 탐색
		- 완전 탐색
		- 이진 탐색
	- 정렬
		- bubble
		- insertion
		- selection
		- merge
		- quick
		- heap



## Array (배열)
- 정의
	- 같은 자료형의 원소들을 일정한 크기의 *연속된 메모리 공간에 순서대로 저장*하고, *인덱스를 통해* 각 원소에 빠르게 접근할 수 있는 자료구조
- 언어별
	- cpp - `vector, array, []`
	- c `[], alloc`
	- ![[image-Data Structures-1.png]]
	- python - `list, []`
		- 파이썬에선 리스트라고 부름
		- ![[image-Data Structures-2.png]]
- 시간 복잡도
	- 삽입
		- $O(n)$
		- 끝에서만 삽입하면 $O(1)$
	- 삭제
		- $O(n)$
		- 끝에서만 삽입하면 $O(1)$
	- 접근
		- $O(1)$
- 공간 복잡도
	- $O(n)$
	- 메모리가 인접되어 있어 효율이 좋음
- 배열 종류
	- ![[image-Data Structures-3.png]]
	- 크기 기반
		- 고정
		- 동적
	- 차원 기반
		- 1차원
			- ![[image-Data Structures-4.png|500]]
		- 2차원
			- ![[image-Data Structures-5.png|500]]
		- 3차원
			- ![[image-Data Structures-6.png|500]]

## Linked Lists (연결 리스트)
- 정의
	- 데이터를 담은 노드들이 *포인터(참조)를 이용해 서로 연결되어 있는 선형 자료구조*로, *메모리 상에서 연속적으로 배치되지 않고* 필요할 때마다 동적으로 생성되어 삽입과 삭제가 용이한 구조
	- ![[image-Data Structures-7.png]]
- 언어별
	- cpp - `list`
- 시간 복잡도
	- 삽입
		- $O(1)$
	- 삭제
		- $O(1)$
	- 접근
		- $O(n)$
- 공간 복잡도
	- $O(n)$
	- 메모리가 연속적으로 할당되지않아, 배열보다 미세하게 느리다(저수준에서)

## Stack (스택)
- 정의
	- *후입선출(LIFO, Last-In First-Out) 방식*으로 데이터를 저장하고 꺼내는 선형 자료구조
	- 데이터의 삽입은 push, 삭제는 pop 연산을 통해 이루어지며, 가장 마지막에 삽입된 원소가 가장 먼저 제거
	- ![[image-Data Structures-8.png]]
- 언어별
	- cpp - `stack`
	- python - `list, []`
- 시간 복잡도
	- 삽입
		- $O(1)$
	- 삭제
		- $O(1)$
	- 접근
		- 맨위의 요소만 접근 가능
			- $O(1)$
		- 중간 요소에 접근하려면 꺼내야한다
			- $O(n)$
- 공간 복잡도
	- $O(n)$

## Queue (큐)
- 정의
	- *선입선출(FIFO, First-In First-Out) 방식*으로 데이터를 저장하고 꺼내는 선형 자료구조
	- 데이터의 삽입은 rear(뒤)에서, 삭제는 front(앞)에서 이루어진다
	- ![[image-Data Structures-9.png]]
	- 앞뒤로 둘다 삽입, 삭제 할 수 있는 `Dequeue`도 있다
- 언어별
	- cpp - `queue`
	- python - `deque`
		- `from collections import deque`
- 시간 복잡도
	- 삽입 / 삭제 / 접근
		- 뒤에서만 가능
			- deque는 앞뒤로 둘다 가능
		- $O(1)$
- 공간 복잡도
	- 구현에 따라 다름
		- 배열 기반 구현
		- 연결 리스트 기반 구현
		- ... 

## Hash Table (해쉬 맵)
- 정의
	- {key, value}
	- 해시맵(Hash Map)은 *키(key)와 값(value)을 쌍으로 저장*
	- *해시 함수(hash function)를 이용해 키를 해시값으로 변환*하여 배열 인덱스에 매핑해 빠르게 검색·삽입·삭제할 수 있는 자료구조
	- ![[image-Data Structures-10.png]]
	- 해싱
		- 해쉬 함수를 실행함을 뜻함
	- 해싱의 결과 같은 키로 배정될 수 있는데, 이를 `키 충돌`이라 부른다
		- 키 충돌 해결 방법론
			- 체이닝
				- 같은 키에다가 리스트(배열)로 저장
			- 개방 주소법
				- 다른 빈 슬롯을 찾아서 저장
			- ...
- 언어별
	- cpp - `unordered_map, unordered_set`
	- python - `dict, {}`
- 시간 복잡도
	- 삽입 / 삭제 / 접근
		- $O(1)$
	- 충돌이 많을 때의 삽입 / 삭제 / 접근
		- $O(n)$

# 그래프
- 그래프
	- *노드(정점, Vertex)와 노드들을 연결하는 간선(Edge)으로 구성*된 자료구조
	- ![[image-Data Structures-29.png]]
	- 종류
		- 방향성
			- ![[image-Data Structures-30.png]]
			- 방향 그래프
				- 단방향
				- 양방향
			- 무방향 그래프
		- 연결성
			- ![[image-Data Structures-31.png]]
			- 연결 그래프
			- 비연결 그래프
		- 가중치
			- ![[image-Data Structures-35.png]]
			- 가중 그래프
			- 비가중 그래프
		- 특수 형태
			- 사이클 그래프
			- 트리
			- ...
	- 그래프 표현법
		- 인접 행렬
			- ![[image-Data Structures-11.png]]
			- ![[image-Data Structures-14.png]]
		- 인접 리스트
			- ![[image-Data Structures-12.png]]
			- ![[image-Data Structures-13.png]]
	- 그래프에서의 검색
		- *BFS* (너비 우선 탐색, Breadth First Search)
		- *DFS* (깊이 우선 탐색, Depth First Search)
		- ![[image-Data Structures-21.png]]
- 트리
	- ![[image-Data Structures-32.png]]
	- 그래프의 한 종류, 사이클이 없는 경우를 뜻함
		- ![[image-Data Structures-19.png]]
	- 이진 트리
		- ![[image-Data Structures-20.png]]
	- 힙
		- 최소, 최대 값을 빠르게 얻기 위한 자료 구조
			- $O(1)$로 값을 구함
			- 삽입시 $O(\log n)$으로 정렬
		- min heap
			- ![[image-Data Structures-16.png]]
			- ![[image-Data Structures-17.png]]
		- max heap
			- ![[image-Data Structures-15.png]]
			- ![[image-Data Structures-18.png]]
	- 트리에서의 순회
		- ![[image-Data Structures-34.png]]
		- Pre-Order Traversal (전위 순회)
			- *Root Node*, Left Node, Right Node
		- In-Order Traversal (중위 순회)
			- Left Node, *Root Node*, Right Node
		- Post-Order Traversal (후위 순회)
			- Left Node, Right Node, *Root Node*
- 고급
	- 트라이(trie)
	- 펜윅 트리(Fenwick trees, Binary Indexed Trees)
	- 세그먼트 트리(Segment trees)
	- Red-Black trees
	- AVL
	- B-trees
	- B+trees
	- ...


- Shortest Path
	- ![[image-Data Structures-22.png]]
	- dijkstra
	- bellman-ford
	- floyd-warshall
	- A-star (A*)
	- ...
- Minimum Spanning Tree (MST)
	- 전국에 인터넷 망을 최소비용으로 깔고 싶다!
	- ![[image-Data Structures-24.png]]
	- ![[image-Data Structures-23.png]]
	- ![[image-Data Structures-25.png]]
	- Prim
	- Kruskal
- topological sorting (위상 정렬)
	- 방향 그래프(DAG, Directed Acyclic Graph)의 모든 노드를 *선행 관계를 만족하도록 일렬로 나열*하는 것
	- 테크 트리, 스킬 트리
	- ![[image-Data Structures-28.png]]
		- 수강 비유
		- `1`번 수업을 듣기 위해 선행 수업 `4` or `3` 가 필요하다
			- 어느 수업부터 들어야 수업을 다 들을 수 있는지
	- ![[image-Data Structures-26.png]]
	- ![[image-Data Structures-27.png]]

# 알고리즘 테크닉
- math
- implementation
- sorting
- searching
- brute force
- backtracking
- greedy
- divide and conquer
- recursion
- dynamic programming
- two pointer
- sliding window
- dfs
- bfs
- prefix sum array
	- imos method
- shortest path