---
title: EP 11. 제어 패스
date: 2025-09-04 11:06:00 +0900
categories: CS
tags: ["CS", "컴퓨터 구조"]
---

[참고 영상](https://youtu.be/WtaHFtUOFIc?si=n_Z6koyLC17niK0o)

지금까지 특정 명령을 받았을 때 회로가 어떻게 작동하는지를 알아봤다! 이제는 회로를 제어하는 부분, 제어 패스에 대해 알아보려고 한다!!! 이게 마지막이다!!!

## 명령어 집합

말그대로의 프로세서가 처리하는 명령어 집합이다! 여기서는 Hex8을 기준으로 설명한다.

| Hex | Binary | Opcode | Operation                          | 설명                                  |
|-----|--------|--------|------------------------------------|---------------------------------------|
| 0   | 0000   | LDAM   | areg <- mem[oreg]                  | 메모리에서 A 레지스터로 로드           |
| 1   | 0001   | LDBM   | breg <- mem[oreg]                  | 메모리에서 B 레지스터로 로드           |
| 2   | 0010   | STAM   | mem[oreg] <- areg                  | A 레지스터를 메모리에 저장             |
| 3   | 0011   | LDAC   | areg <- oreg                       | 상수를 A 레지스터에 로드               |
| 4   | 0100   | LDBC   | breg <- oreg                       | 상수를 B 레지스터에 로드               |
| 5   | 0101   | LDAP   | areg <- pc + oreg                  | PC 오프셋을 A 레지스터에 로드           |
| 6   | 0110   | LDAI   | areg <- mem[areg + oreg]           | A 레지스터 인덱스 로드                 |
| 7   | 0111   | LDBI   | breg <- mem[breg + oreg]           | B 레지스터 인덱스 로드                 |
| 8   | 1000   | STAI   | mem[breg + oreg] <- areg           | 인덱스 저장                            |
| 9   | 1001   | BR     | pc <- pc + oreg                    | 상대 분기                              |
| A   | 1010   | BRZ    | if areg = 0 then pc <- pc + oreg   | 조건 분기 (A = 0일 때)                 |
| B   | 1011   | BRN    | if areg < 0 then pc <- pc + oreg   | 조건 분기 (A < 0일 때)                 |
| C   | 1100   | BRB    | pc <- breg                         | B 레지스터를 이용한 절대 분기          |
| D   | 1101   | ADD    | areg <- areg + breg                | 덧셈, 결과는 A 레지스터에 저장          |
| E   | 1110   | SUB    | areg <- areg - breg                | 뺄셈, 결과는 A 레지스터에 저장          |
| F   | 1111   | PFIX   | oreg <- oreg << 4                  | 오프셋 레지스터 상위 비트 설정          |

> 메모리 명령에서 A 레짓터는 주소로 사용된다.

> 인덱스 가져오기/저장을 통해 주소에 좌표를 더할 수 있다. 

각 명령어는 제어 신호를 보낸다. 우리가 만든 회로를 기준으로 해 제어 신호를 설정하면 이렇다.

![](img/computer-architecture/control_signal.png)

- 해당 명령어가 특정 제어 신호를 활성화할 경우 1이 표시된다.
- 각 열의 명령 신호들을 모두 **OR 연산**해 제어 경로를 생성한다.

이제 전체를 연결하면 이렇다! 이렇게 8bit 컴퓨터가 만들어진다!!

![](img/computer-architecture/full_logic.png)