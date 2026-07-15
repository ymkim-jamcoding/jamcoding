---
description:
aliases:
created: 2026-07-11
modified: 2026-07-11
---

# DFT
- $[1, 2, 3, 4]$
- $X[k]=\sum_{n=0}^{3}x[n]\omega^{kn}$
- $\omega=e^{-2\pi i/4}=-i$
- 단위원 값
	- $\omega^0=1,\quad \omega^1=-i,\quad \omega^2=-1,\quad \omega^3=i,\quad \omega^4=1 \quad \dots$


- $X[0] = 1\cdot\omega^{0} +2\cdot\omega^{0} +3\cdot\omega^{0} +4\cdot\omega^{0}$

| **구하는 값** | n=0          | n=1           | n=2           | n=3           |
| ------------- | ------------ | ------------- | ------------- | ------------- |
| X[0]          | $\omega^0=1$ | $\omega^0=1$  | $\omega^0=1$  | $\omega^0=1$  |
| X[1]          | $\omega^0=1$ | $\omega^1=-i$ | $\omega^2=-1$ | $\omega^3=i$  |
| X[2]          | $\omega^0=1$ | $\omega^2=-1$ | $\omega^4=1$  | $\omega^6=-1$ |
| X[3]          | $\omega^0=1$ | $\omega^3=i$  | $\omega^6=-1$ | $\omega^9=-i$ |

- $X[0]=1(1)+2(1)+3(1)+4(1)$
- $X[1]=1(1)+2(-i)+3(-1)+4(i)$
- $X[2]=1(1)+2(-1)+3(1)+4(-1)$
- $X[3]=1(1)+2(i)+3(-1)+4(-i)$
- $\boxed{[10,\,-2+2i,\,-2,\,-2-2i]}$

## 역변환
- $x[n]=\frac{1}{N}\sum_{k=0}^{N-1}X[k]\omega^{-kn}$


# FFT
- 