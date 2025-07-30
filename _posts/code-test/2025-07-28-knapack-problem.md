---
title: 배낭문제와 동적 계획법
date: 2025-07-28 13:42:00 +0900
categories: 코딩테스트
tags: ["코딩 테스트", "알고리즘", "C++", "공부"]
---

백준 12856번 평범한 배낭 문제를 푸는 과정을 정리했다. 내 풀이의 문제점에 걸쳐 동적 계획법 위주로 설명하려고 한다.

## 배낭 알고리즘
> n개의 물건에는 각각 가치와 무게가 존재한다. 배낭은 담을 수 있는 최대 무게가 존재한다. 이 무게를 초과하지 않고 최대 가치를 가지도록 짐을 담아야 한다.

짐을 쪼갤 수 있는 부분 배낭 문제는 그리디 알고리즘으로 해결 가능하지만, 짐을 쪼갤 수 없을 경우 동적 계획법으로 해결해야한다. 그리디는 현재의 선택이 다음 선택에 영향을 끼치면 사용할 수 없기 때문이다.

## 내 풀이

주먹구구식으로 풀었다.

dp는 0에서 무게 k까지의 배열이다.

```
dp[i].first = 배낭이 무게 i일때의 최대 가치
dp[i].second = 배낭이 무게 i일 때 넣은 아이템 인덱스들 (unordered_set)
```
1. dp를 기본적으로 주어지는 아이템으로 초기화한다. 
2. 0에서 무게 k까지, 아이템이 최소 하나 들어가 있는 무게 i마다 다른 아이템을 추가한다.
3. 이때 무게가 최대이면 그 아이템들로 저장한다.
4. 순회가 끝나면 dp에서 최대를 구한다.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <unordered_set>
#include <utility>

using namespace std;

int main(){
	int n, k;

	cin >> n >> k;
	
	vector<pair<int, int>> arr(n, make_pair(0, 0));
	vector<pair<int, unordered_set<int>>> dp(k+1);
	
	for(int i=0; i<n; i++){
		cin >> arr[i].first >> arr[i].second;
		if (arr[i].first <= k && dp[arr[i].first].first < arr[i].second) { // out of bounds 에러 
			dp[arr[i].first].first = arr[i].second;
            dp[arr[i].first].second.clear();
            dp[arr[i].first].second.insert(i); // 초기화할 때 아이템은 하나만 넣도록
		}
	}
	
	for(int i=0; i<=k; i++){
		if(dp[i].first == 0){
			continue;
		}
		
		for(int j=0; j<n; j++){
			if(i + arr[j].first <= k && dp[i].second.find(j) == dp[i].second.end()){
				if(dp[i + arr[j].first].first < dp[i].first + arr[j].second){
					dp[i + arr[j].first].first = dp[i].first + arr[j].second;
					dp[i + arr[j].first].second = dp[i].second;
					dp[i + arr[j].first].second.insert(j);
				} 
			}
		}
	}
	
	int maxValue =0;
	for(int i=0; i<=k; i++){
		maxValue = max(maxValue, dp[i].first);
	}
	
	cout << maxValue;
	
	return 0;
}
```

간단하지만 코드도 길고 용량도 많이 차지하고 실행 시간도 길다. (메모리는 4448KB에 224ms 나온다.) 무게의 크기만큼의 배열은 좋은 아이디어 같지만, unordered_set으로 일일이 비교하는 것 말고도 다른 방법이 있지 않을까 생각했다. 이에 몇가지 방법을 찾아봤다.


## 이중 배열을 사용

> 코딩 테스트 합격자 되기 C++편을 참고했다.

**0~(N-1)번째 아이템까지 있다고 했을 때, 각 아이템을 추가한 경우와 추가하지 않은 경우를 비교해가며 최적의 해를 구하는 방식이다.**

부분 문제의 결과를 저장하는 배열은 dp[0...아이템 개수n][0...배낭 무게k]이다. 각 배열의 원소는 다음과 같은 의미를 가진다.

```
dp[i][w] = i번째 아이템를 고려하고(선택하거나 선택 안하거나), w의 무게일 때의 최대 가치
```

이제 추가하는 경우와 추가하지 않는 경우를 생각하면 된다. 단 고려해야할 점이 있는데, **현재 고려하는 i번째 아이템의 무게가 현재 무게 w보다 작아야 한다.** 그래야 i번째 아이템를 문제없이 담을 수 있다. 그래서 w가 i번째 아이템의 무게보다 작을 경우와 큰 경우를 두 가지를 고려해야 한다.

그렇게 나온 발화식은 다음과 같다. weight(i)는 i번째 아이템의 무게, value(i)는 i번째 아이템의 가치를 뜻한다.

```
dp[i][w] = max(dp[i-1][w], dp[i-1][w-weight(i)]+value(i)) (w >= weight(i)인 경우) 
dp[i][w] = dp[i-1][w] (w < weight(i)인 경우)
```
dp[i][w] = dp[i-1][w]는 i번째 아이템을 고려하지 않는 경우다.

이런 발화식을 사용해 코드를 작성하면 이렇다.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main(){
	int n, k;

	cin >> n >> k;
	
	vector<pair<int, int>> arr(n, make_pair(0, 0));
	vector<vector<int>> dp(n, vector<int>(k+1, 0));
	
		
	for(int i=0; i<n; i++){
		cin >> arr[i].first >> arr[i].second;
	}
	
	for(int i=arr[0].first; i<=k; i++){
		dp[0][i] = arr[0].second;
	}
	
	for(int i=1; i<n; i++){
		int w = arr[i].first;
		int v = arr[i].second;
		
		for(int j=1; j<=k; j++){
			if(j >= w){
				dp[i][j] = max(dp[i-1][j], dp[i-1][j-w] + v);
			} else {
				dp[i][j] = dp[i-1][j];
			}
		}
	}
	
	cout << dp[n-1][k];

	return 0;
}
```

> dp 배열을 [0...n-1][0...k]로 설정한 이유는 아이템의 무게가 1 이상이기 때문이다. [0...n-1][0...k-1]로 설정하면 무게가 1인 아이템은 dp[][1]이 아니라 dp[][0]에 들어가야 한다. (물론 1씩 빼서 k-1로 범위를 설정해도 문제 풀이에는 지장 없다.)

해당 코드로 하니 시간이 40ms로 대폭 줄었다. 그런데 좀 더 알아보니 이것보다 더 간단히 풀 수 있는 방법이 있다고 한다...

## 역순 배열

**무게가 허락하는 최대한 큰 가치의 아이템을 집어넣는 방식이다.** 사실 풀이 자체는 위의 이중 배열과 비슷한데, 일차원 배열을 사용하기 위해 역순을 사용하는 점이 특이하다.

부분 문제의 결과를 저장하는 배열은 dp[0...배낭 무게k]이다. 각 배열의 원소는 다음과 같은 의미를 가진다.

```
dp[w] = w의 무게일 때의 최대 가치
```

해당 배열을 통해 i번째 아이템을 고려할 때 dp[w]의 최대 가치, i+1번째 아이템을 고려할 때 dp[w]의 최대 가치... 이런 식으로 계속해서 dp의 값을 갱신하면 된다. 하지만 해당 배열은 이중 배열이 아니다. i+1번째 아이템을 고려할 때, i번째 아이템을 선택해서 구한 값을 제대로 반영해야한다.

이를 위해 최종 무게부터 현 아이템의 무게까지, 역순으로 순회하며 이전의 값을 반영한다.

### 예시

예로 최대 무게 k = 5, 아이템[무게, 가치]은 [[1, 6], [2, 10], [3, 12]]가 있다고 해보자.

초기화된 상태에서는 다음과 같다.

| 무게 w | 0 | 1 | 2 | 3 | 4 | 5 |
|--------|---|---|---|---|---|---|
| 가치 v | 0 | 0 | 0 | 0 | 0 | 0 |


#### 0번째 아이템

0번째 아이템 [1, 6]을 고려해 각 항목에 최대 가치를 넣으려고 한다. 이때 최대 무게를 넘으면 안되므로 **현재 무게 w에서, 아이템의 무게 weight(0)를 뺀 항목과 가치를 비교해야 한다.**

```
dp[w] = max(dp[w], dp[w-weight(i)] + value(i)) 
(단, w는 weight(i) <= k, i는 0 < 아이템 개수 n)
```

w의 범위를 위와 같이 설정한 이유는 아이템 i가 들어갈 수 있는 무게의 범위가 자신의 무게에서 가방 내 최대 무게까지이기 때문이다.

하지만 해당 초기화를 정방향으로 고려할 경우, 

| 무게 w | 0 | 1 | 2 | 3 | 4 | 5 |
|--------|---|---|---|---|---|---|
| 가치 v | 0 | 6 | 12 | 18 | 24 | 30 |

**이전에 넣은 아이템 [1, 6]들을 중복해서 넣는 대참사가 벌어진다.**

예:
```
dp[2] = max(dp[1], dp[2-1]+6) = 23
```

이를 방지하기 위해 역방향으로 순회한다. 즉, k부터 시작해 weight(i)일 때까지를 돌아 아이템을 중복해서 넣는 걸 막는다. **역순으로 사용하면 이번 단계에서는 아직 사용하지 않은 dp 항목을 참조하게 돼, 아이템을 한 번만 사용하는 것이 보장된다.**

다시 초기화된 배열로 돌아가, 역순으로 할 경우 어떻게 아이템을 넣는지 보자.

| 무게 w | 0 | 1 | 2 | 3 | 4 | 5 |
|--------|---|---|---|---|---|---|
| 가치 v | 0 | 0 | 0 | 0 | 0 | 0 |

위와 같을 때, 이제 [1, 6]이므로 5에서부터 1까지 순회한다.
```
dp[w] = max(dp[w], dp[w-weight(i)] + value(i)) 

dp[5] = max(dp[5]=0, dp[4]+6) = 6
dp[4] = max(dp[4]=0, dp[3]+6) = 6
dp[3] = max(dp[3]=0, dp[2]+6) = 6
dp[2] = max(dp[2]=0, dp[1]+6) = 6
dp[1] = max(dp[1]=0, dp[0]+6) = 6
```

이 과정을 거쳐 dp는 다음과 같이 값을 가진다.

| 무게 w | 0 | 1 | 2 | 3 | 4 | 5 |
|--------|---|---|---|---|---|---|
| 가치 v | 0 | 6 | 6 | 6 | 6 | 6 |


#### 1번째 아이템

아직은 좀 헷갈릴 수도 있다. 이제 1번째 아이템[2, 10]도 고려해보자. 5에서 2까지 순회한다.

```
dp[w] = max(dp[w], dp[w-weight(i)] + value(i)) 

dp[5] = max(dp[5]=6, dp[3]+10) = 16
dp[4] = max(dp[4]=6, dp[2]+10) = 16
dp[3] = max(dp[3]=6, dp[1]+10) = 16
dp[2] = max(dp[2]=6, dp[0]+10) = 10
```

dp[0]+10의 경우는 아무것도 없는 가방에 10을 추가했다고 보면 된다. dp[w-weight(i)] 는 `(단, w는 weight(i) <= k, i는 0 < 아이템 개수 n)` 조건을 통해 해당 아이템이 들어갈 수 있는 모든 공간은 전부 참고해 가치를 갱신한다.

> 물론 weight(i)가 k보다 크다면 그건 별개의 이야기다만... 넣지도 못하는 아이템을 굳이 문제로 추가할까.

결과는 다음과 같다.

| 무게 w | 0 | 1 | 2 | 3 | 4 | 5 |
|--------|---|---|---|---|---|---|
| 가치 v | 0 | 6 | 10 | 16 | 16 | 16 |


#### 2번째 아이템
2번째 아이템은 [3, 12]다. 5에서 3까지 순회한다.

```
dp[w] = max(dp[w], dp[w-weight(i)] + value(i)) 

dp[5] = max(dp[5]=16, dp[2]+12) = 22
dp[4] = max(dp[4]=16, dp[1]+12) = 18
dp[3] = max(dp[3]=16, dp[0]+12) = 16
```

| 무게 w | 0 | 1 | 2 | 3 | 4 | 5 |
|--------|---|---|---|---|---|---|
| 가치 v | 0 | 6 | 10 | 16 | 18 | 22 |

이렇게 계산을 진행해, dp 중 최대치인 22를 답으로 계산할 수 있다.


### 코드


이 풀이를 반영한 코드는 다음과 같다.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main() {
	int n, k;
	cin >> n >> k;

	vector<pair<int, int>> items(n);
	for (int i = 0; i < n; i++) {
		cin >> items[i].first >> items[i].second;
	}

	vector<int> dp(k + 1, 0);

	for (int i = 0; i < n; i++) {
		int w = items[i].first;
		int v = items[i].second;

		for (int j = k; j >= w; j--) {  // 역순!
			dp[j] = max(dp[j], dp[j - w] + v);
		}
	}

	cout << *max_element(dp.begin(), dp.end()) << endl;
	return 0;
}
```

해당 풀이의 핵심은 **역순을 통해 이번 단계에서 아직 안쓴 항목을 참고**하는 것이다. 해당 방법을 사용하면 메모리도 시간도 획기적으로 줄어든다.


역순 배열을 설명하느라 이중 배열에 대한 설명은 좀 간략하게 했다. 이중 배열 동적 계획법은 다른 블로그에서도 많은 설명을 다루니 그쪽을 참고해보자.