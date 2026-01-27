---
description:
aliases:
created: 2026-01-27
modified: 2026-01-27
---

# test
- a
- b
- c
- d
- e
- f
- g
- h
- i
- j
- k

# 2
- 1
- 2
- 3
- 4
- 5
- 6
- 7

# 3
```cpp
int CCW(Point a, Point b, Point c) {
    long long t1 = a.x * b.y + b.x * c.y + c.x * a.y;
    long long t2 = a.y * b.x + b.y * c.x + c.y * a.x;
    // long long t1 = (b.x - a.x) * (c.y - a.y);
    // long long t2 = (c.x - a.x) * (b.y - a.y);

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


# 4
- $\vec{a} \cdot \vec{b} = \lVert \vec{a} \rVert \, \lVert \vec{b} \rVert \cos\theta$