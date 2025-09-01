---
title: EP 4. 래치 메모리
date: 2025-09-01 09:55:00 +0900
categories: CS
tags: ["CS", "컴퓨터 구조"]
---

[참고 영상](https://youtu.be/QFZ5_UwTTII?si=YgBhpfSDQOC7VF1r)

## Combinational & Sequential

지난 영상들은 조합 회로Combinational logic을 주로 다뤘다. 조합 회로는 다음과 같은 특징을 가진다.

- 출력은 입력의 조합이다.
- 출력은 오직 입력으로만 구성된다.
- 이전 입력에 대한 메모리가 없다. 이전 입력을 저장하지 않는다.

이번에는 순차 회로Sequential logic을 다룰 것이다. 순차 회로는 조합 회로와 달리 **이전 입력을 기억한다**. 순차 회로의 출력은 이전에 받았던 입력도 계산에 사용하는 게 주요 특징이다.

## SR Latch

![](img/computer-architecture/sr_latch.png)

비트 하나를 기억하는 회로다. SR은 Set - Reset의 약자다. 이 회로는 두 개의 안정적인 상태를 가진다고 해서 쌍안정Bistable 회로라고 부른다.

> Set을 1로 지정하면 출력은 1, Reset을 1로 지정하면 출력은 0이 된다.

SR 래치에서 Q'는 Q의 정반대 비트를 나타낸다. 

> Q'는 출력과 연결된 다음 게이트를 최적하는데 유용하다.

| S | R | Q | Q' | Action    |
|---|---|---|----|-----------|
| 0 | 0 | 0 | 1  | Stable    |
| 1 | 0 | 1 | 0  | 1         |
| 0 | 1 | 0 | 1  | 0         |
| 1 | 1 | X | X  | Undefined |

## Clock-gated SR Latch

![](img/computer-architecture/clock_gated_sr_latch.png)

위 회로는 **Clk 값이 1일 때에만** S와 R에 따라 값이 바뀐다. Clk 값이 0이면 바뀌지 않는다. 즉, **Clk가 0이면 출력값을 계속 기억한다.**

이 래치를 가지고 플립플롭을 만들고, 플립플롭을 통해 레지스터를, 메모리를 구성한다.