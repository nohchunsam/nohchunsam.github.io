---
title: 프로그래머스 귤 고르기 - vector<pair> vs unordered_map
date: 2025-07-31 10:18:00 +0900
categories: 코딩테스트
tags: ["코딩 테스트", "알고리즘", "C++", "공부"]
---

**주의: 해당 포스트는 문제 해결보다 실행 시간 비교에 더 초점을 뒀다!! 답은 나와있지만 문제 해결 과정을 보고싶은 사람은 다른 포스트를 참고해라!!**

## 문제

### 문제 설명
경화는 과수원에서 귤을 수확했습니다. 경화는 수확한 귤 중 'k'개를 골라 상자 하나에 담아 판매하려고 합니다. 그런데 수확한 귤의 크기가 일정하지 않아 보기에 좋지 않다고 생각한 경화는 귤을 크기별로 분류했을 때 서로 다른 종류의 수를 최소화하고 싶습니다.

예를 들어, 경화가 수확한 귤 8개의 크기가 [1, 3, 2, 5, 4, 5, 2, 3] 이라고 합시다. 경화가 귤 6개를 판매하고 싶다면, 크기가 1, 4인 귤을 제외한 여섯 개의 귤을 상자에 담으면, 귤의 크기의 종류가 2, 3, 5로 총 3가지가 되며 이때가 서로 다른 종류가 최소일 때입니다.

경화가 한 상자에 담으려는 귤의 개수 k와 귤의 크기를 담은 배열 tangerine이 매개변수로 주어집니다. 경화가 귤 k개를 고를 때 크기가 서로 다른 종류의 수의 최솟값을 return 하도록 solution 함수를 작성해주세요.

### 제한사항
1 ≤ k ≤ tangerine의 길이 ≤ 100,000
1 ≤ tangerine의 원소 ≤ 10,000,000

### 입출력 예

| k | tangerine                 | result |
| - | ------------------------- | ------ |
| 6 | \[1, 3, 2, 5, 4, 5, 2, 3] | 3      |
| 4 | \[1, 3, 2, 5, 4, 5, 2, 3] | 2      |
| 2 | \[1, 1, 1, 1, 2, 2, 2, 3] | 1      |


## solution1 = 내 풀이

나는 이렇게 풀었다.

1. pair로 귤의 종류와 개수를 저장
2. 귤 개수를 기준으로 내림차순으로 정렬
3. 귤 개수만큼 순회, 귤 개수마다 answer++, k를 귤 개수만큼 빼서 k가 0이 될 때까지 반복

```cpp
int solution(int k, vector<int> tangerine) {
    int answer = 0;
    vector<pair<int, int>> v;
    
    sort(tangerine.begin(), tangerine.end());
    
    v.push_back(make_pair(tangerine[0], 1));
    for(int i=1; i<tangerine.size(); i++){
        if(tangerine[i] == v.back().first){
            v.back().second++;
        } else {
            v.push_back(make_pair(tangerine[i], 1));
        }
    }
    
    sort(v.begin(), v.end(), [](pair<int, int>a, pair<int, int>b){
        return a.second > b.second;
    });
    
    for(int i=0; i<v.size(); i++){
        if(k <= v[i].second){
            answer++;
            break;
        }
        
        k -= v[i].second;
        answer++;
    }
    
    
    return answer;
}

```

## solution2 = 답지 풀이

그리고 뒤늦게 unordered_map으로 해결하는 걸 떠올렸다. 처음에는 vector<pair>로 변환해 정렬을 돌렸으나, 중요한 건 개수이기 때문에 vector<int>로만 정렬 돌리고 종류 수를 세도 된다는 해설을 봤다. 코드를 다음과 같이 고쳤다.


```cpp
int solution(int k, vector<int> tangerine) {
    int answer = 0;
    unordered_map<int, int> um;
    
    for(int i=0; i<tangerine.size(); i++){
        um[tangerine[i]]++;
    }
    
    vector<int> v;
    for(auto e: um){
        v.push_back(e.second);
    }
    
    sort(v.rbegin(), v.rend());
    
    for(int i=0; i<v.size(); i++){
        if(k <= v[i]){
            answer++;
            break;
        }
        
        k -= v[i];
        answer++;
    }
    
    
    return answer;
}
```

## 그런데 어떨 때는 solution1이 더 빠르다.

정렬 횟수가 적으니 답지 풀이가 더 빠를거라 생각했는데, 정작 체점을 돌려보면 어떤 테스트에서는 내 풀이가, 어떤 테스트에서는 답지 풀이가 더 빨랐다. 

### 테스트하기

제대로 알아보기 위해 테스트를 해봤다. 내 풀이는 solution1로, 답지 풀이는 solution2로 함수명을 고치고, Chat GPT로 각 테스트 케이스를 만들었다. 시간 계산은 [이 글](https://woo-dev.tistory.com/6)을 참고했다.

#### 테스트 입력 1: 모든 원소가 동일

```cpp
int k = 50000;
vector<int> tangerine(100000, 42);  // 모두 42

solution1 = 0s(25ms)
solution2 = 0s(13ms)
```

#### 테스트 입력 2: 무작위 분포 (종류 수 많음)

```cpp
#include <vector>
#include <random>
using namespace std;

int k = 50000;
vector<int> tangerine;

mt19937 rng(42);
uniform_int_distribution<int> dist(1, 10000000);

for (int i = 0; i < 100000; ++i) {
    tangerine.push_back(dist(rng));
}

solution1 = 0s(67ms)
solution2 = 0s(190ms)
```

#### 테스트 입력 3: 값이 1부터 100000까지 하나씩 (값의 종류가 최대)
```cpp
int k = 100000;
vector<int> tangerine;

for (int i = 1; i <= 100000; ++i) {
    tangerine.push_back(i);
}

solution1 = 0s(61ms)
solution2 = 0s(158ms)
```

#### 테스트 입력 4: 10개의 값이 반복
```cpp
int k = 50000;
vector<int> tangerine;

for (int i = 0; i < 100000; ++i) {
    tangerine.push_back(i % 10);  // 0~9 반복
}

solution1 = 0s(34ms)
solution2 = 0s(11ms)
```

#### 테스트 입력 5: 값 하나만 9999999, + 나머지는 전부 1
```cpp
int k = 1;
vector<int> tangerine(99999, 1);
tangerine.push_back(9999999);

solution1 = 0s(28ms)
solution2 = 0s(9ms)
```

### 테스트 결과

| 테스트 케이스 | 종류 수 추정 | soltuon1 속도 | solution2 속도 | 더 빠른 쪽 |
| ------- | ------- | ------- | ------- | ------ | 
| 1       | 1       | 25ms    | 13ms    | B      | 
| 2       | 많음      | 67ms    | 190ms   | A      | 
| 3       | 100,000 | 61ms    | 158ms   | A      | 
| 4       | 10      | 34ms    | 11ms    | B      | 
| 5       | 2       | 28ms    | 9ms     | B      | 


**결론적으로 solution1 (내 풀이, vector<pair>)는 귤 종류가 많을 때 더 빠르고, solution2 (답지 풀이, unordered_map)는 귤 종류가 적을 때 더 빠르다.**

## 왜 더 빠르지??

### solution1 = 내 풀이의 시간복잡도
- tangerine 정렬: O(NlogN)
- 빈도 개수 세기: O(N)
- 빈도 개수 정렬: O(MlogM)
- 답 구하기: O(M)

결론적으로는 O(NlogN + MlogM)이다.

### solution2 = 답지 풀이의 시간복잡도
- 빈도 개수 세기: O(N)
- 빈도 벡터로 변경: O(M)
- 빈도 개수 정렬: O(MlogM)
- 답 구하기: O(M)

결론적으로는 O(N + MlogM)이다.

두 풀이의 차이는 tangerine을 정렬했냐 아니냐, 개수를 세는데 vector<pair>를 사용했냐, unordered_map을 사용했냐에 있다.

빈도 개수 정렬과 답 구하기의 시간복잡도는 같으니, 이 시간을 빼면 solution1 = O(NlogN), solution2 = O(N+M) 이렇게 된다. 단, M은 귤 종류의 개수인만큼 N과 같거나 작다. 그러니 O(N)인 solution2가 더 빠를거라 생각했으나...

> 귤 종류가 아무리 많다해도 귤 개수를 넘진 못한다. (테스트 입력 3같이)


### 이유

#### 1. unordered_map의 추가 연산

unordered_map은 **해시 테이블** 기반 자료구조다.
종류가 많아질수록 해시 충돌이 발생할 확률이 높아지고, 충돌을 처리하기 위해 분리 체이닝 등의 추가 연산을 진행해야 한다.

> unordered_map은 충돌을 해결하기 위해 분리 체이닝chaining을 사용한다. 충돌이 발생하면 해당 해시에 배열을 만들어, 기존 값에 이어 신규 값을 추가한다. 이때 충돌된 해시에서 값을 찾으려면 최악의 경우 끝까지 순회해야한다. (참고: https://woo-dev.tistory.com/106)

분리 체이닝을 사용하면 삽입을 할 때도 각 원소에 접근할 때도 추가적인 시간이 걸린다.

### 2. 캐시

solution1은 vector<pair>를 사용한다. vector는 메모리에 연속적으로 담기 때문에 캐시 효율이 좋다. 하지만 solution2는 unordered_map을 사용하는데, unordered_map은 메모리를 불연속적으로 할당한다. 이 때문에 종류가 많을수록 캐시 미스가 발생하기 쉽다.

```
A vector of std::pair will likely end up being one contiguous block of memory for the whole thing. std::map and std::hash_map will have many tiny blocks.

Particularly if the table is large, your L1 and L2 caches will perform much better when you iterate over a table that's packed that way. If you nearly always iterate in order and rarely do random access lookups, you could very reasonably get substantial speedups in real world code

```

즉, 귤 종류가 을수록 unordered_map의 성능은 더 떨어지고, vector<pair>는 오히려 안정적인 성능을 보여준다. 이 문제가 해시 테이블의 단점을 보여주는 예시가 됐다.

하지만 반대로 unordered_map은 데이터가 적으면 vector<pair>보다 더 빠른 성능을 보여준다. 이건 해시 테이블의 장점이다. 데이터가 적다면 삽입/접근/삭제에 빠른 성능을 보여준다.

그러니 실제로 사용한다면 각 장단점을 알고 사용하는 게 좋을 것 같다.