---
description:
aliases:
created: 2026-08-01
modified: 2026-08-01
---

# 참고자료
- [CCW와 선분 교차 - YouTube](https://www.youtube.com/watch?v=iIDgR5uFy9o&t=376s)

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

```text
long long ccw(Point a, Point b, Point c) {
    return (b.x - a.x) * (c.y - a.y)
         - (b.y - a.y) * (c.x - a.x);
}
```

## Python 코드

```text
def ccw(a, b, c):
    return (
        (b[0] - a[0]) * (c[1] - a[1])
        - (b[1] - a[1]) * (c[0] - a[0])
    )
```

# 연습문제
- [CCW - 11758](https://joj.kr/problem/11758)
- [선분 교차 1 - 17386](https://joj.kr/problem/17386)
- [선분 교차 2 - 17387](https://joj.kr/problem/17387)
- [다각형의 면적 - 2166](https://joj.kr/problem/2166)
- [볼록 껍질 - 1708](https://joj.kr/problem/1708)
	- [그레이엄 스캔 - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%97%84_%EC%8A%A4%EC%BA%94)