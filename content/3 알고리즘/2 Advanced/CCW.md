---
description:
aliases:
created: 2026-08-01
modified: 2026-08-11
---

# 참고자료
- ![CCW와 선분 교차 - YouTube](https://www.youtube.com/watch?v=iIDgR5uFy9o&t=376s)

# CCW

CCW(Counter Clockwise)는 세 점의 방향을 판별하는 알고리즘이다.
세 점 (A), (B), (C)를 순서대로 연결했을 때, 어느 방향으로 꺾이는지 확인한다.

![[image-CCW.png]]
- 양수: 반시계 방향
- 음수: 시계 방향
- 0: 세 점이 일직선
    

## 공식
벡터 $\overrightarrow{AB}$와 $\overrightarrow{AC}$의 2차원 외적

세 점을 $A=(a_x,a_y),\quad B=(b_x,b_y),\quad C=(c_x,c_y)$ 로 두자

### A를 기준점으로 옮긴다
먼저 $A$에서 $B$로 가는 벡터와 $A$에서 $C$로 가는 벡터를 구하기

$$
\overrightarrow{AB} = (b_x-a_x,\ b_y-a_y) \quad \overrightarrow{AC} = (c_x-a_x,\ c_y-a_y)
$$

### 두 벡터의 외적을 계산한다

2차원 벡터 $\vec{u}=(u_x,u_y),\quad \vec{v}=(v_x,v_y)$ 의 외적 값은 다음과 같이 정의

$$
\vec{u}\times\vec{v} = u_xv_y-u_yv_x
$$

여기에 $\overrightarrow{AB}$와 $\overrightarrow{AC}$를 넣으면,
$$
\overrightarrow{AB}\times\overrightarrow{AC}
$$

$$
= (b_x-a_x)(c_y-a_y) - (b_y-a_y)(c_x-a_x)
$$

```cpp
(b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x)
```

- 이렇게 구해진 값은 **이 값은 두 벡터가 만드는 평행사변형의 부호 있는 넓이**

## 언제 쓰는가

- 세 점의 방향을 판별할 때
    
- 두 선분이 교차하는지 확인할 때
    
- 볼록 껍질을 구할 때
    
- 다각형의 방향을 판별할 때
    

## C++ 코드

```cpp
long long ccw(Point a, Point b, Point c) {
    return (b.x - a.x) * (c.y - a.y)
         - (b.y - a.y) * (c.x - a.x);
}
```

```cpp
bool onSegment(Point A, Point B, Point C) {
    return min(A.x, B.x) <= C.x && C.x <= max(A.x, B.x)
        && min(A.y, B.y) <= C.y && C.y <= max(A.y, B.y);
}

int CCW(Point A, Point B, Point C) {
    long long t1 = A.x * B.y + B.x * C.y + C.x * A.y;
    long long t2 = A.y * B.x + B.y * C.x + C.y * A.x;
    // long long t1 = (B.x - A.x) * (C.y - A.y);
    // long long t2 = (C.x - A.x) * (B.y - A.y);

    if (t1 - t2 > 0) return 1;   // 반시계
    else if (t1 - t2 == 0) return 0; // 일직선
    else return -1;              // 시계
}

bool isIntersect(Point A, Point B, Point C, Point D) {
    int ab_c = CCW(A, B, C);
    int ab_d = CCW(A, B, D);
    int cd_a = CCW(C, D, A);
    int cd_b = CCW(C, D, B);

    // 일반 교차
    if (ab_c * ab_d < 0 && cd_a * cd_b < 0)
        return true;

    // 일직선 + 선분 위
    if (ab_c == 0 && onSegment(A, B, C)) return true;
    if (ab_d == 0 && onSegment(A, B, D)) return true;
    if (cd_a == 0 && onSegment(C, D, A)) return true;
    if (cd_b == 0 && onSegment(C, D, B)) return true;

    return false;
}
```

## Python 코드

```python
def ccw(a, b, c):
    return (
        (b[0] - a[0]) * (c[1] - a[1])
        - (b[1] - a[1]) * (c[0] - a[0])
    )
```


# 볼록 껍질
- ![볼록 껍질 알고리즘 - YouTube](https://www.youtube.com/watch?v=ZzQGZ97ax0k)

## 그라함 스캔
- 그라함 스캔(Graham scan)은 볼록 껍질을 구하는 알고리즘
- [그레이엄 스캔 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%97%84_%EC%8A%A4%EC%BA%94)
	- ![그레이엄 스캔](https://upload.wikimedia.org/wikipedia/commons/7/71/GrahamScanDemo.gif)
- 로직
	1. 가장 작은 y값을 기준점 잡기, 같다면 x가 작은
	2. 점을 각도 순으로 정렬
	3. 순서대로 점을 보면서, 스택에 차례대로 집어 넣기
	4. 만약에 스택의 마지막 두 점과 새로 추가되는 점이 CCW가 아니라면, CCW가 될때까지 스택을 pop
	5. 마지막에 스택에 남아있는 점들이 볼록 껍질을 구성하는 점들이다

## 모노톤 체인
- 모노톤 체인(Monotone chain)은 볼록 껍질을 구하는 알고리즘, Andrews's Algorithm이라고도 불린다
- 로직
	1. 점을 x좌표 순으로 정렬한다(같을 시 y좌표)
	2. 순서대로 점을 보면서, 스택에 차례대로 집어 넣는다
	3. 만약에 스택의 마지막 두 점과 새로 추가되는 점이 CW가 아니라면, CW가 될때까지 스택을 pop 한다
	4. 마지막에 스택에 남아 있는 점들이 윗 껍질을 구성하는 점들이다
	5. CW대신 CCW로 바꾸어서 아래 껍질을 찾는다
	6. 윗 껍질과 아랫 껍질의 합이 볼록 껍질이다.
- 각도 정렬 필요 없고, 특정 x 범위 내의 convex hull을 빠르게 구할 수 있다

# 연습문제
- ccw
	- [CCW - 11758](https://joj.kr/problem/11758)
	- [선분 교차 1 - 17386](https://joj.kr/problem/17386)
	- [선분 교차 2 - 17387](https://joj.kr/problem/17387)

- Convex hull
	- [다각형의 면적 - 2166](https://joj.kr/problem/2166)
	- [볼록 껍질 - 1708](https://joj.kr/problem/1708)
	- [단순 다각형 - JUNGOL](https://jungol.co.kr/problem/15585)
		- [단순 다각형 - 3679](https://joj.kr/problem/3679)
	- [미술관 - JUNGOL](https://jungol.co.kr/problem/2917)
