---
title: BackTracking
date: 2024-02-14 +09:00
categories:
  - Study
  - Algorithm
tags:
  - 알고리즘
  - 백트래킹
---
## 백트래킹(BackTracking) 이란?

백트래킹은 해를 찾는 도중 해가 될 가능성이 없는 경우 계속 진행하지 않고 되돌아가는 알고리즘임

## 완전탐색과 백트래킹

백트래킹도 완전탐색의 일부라고 볼 수 있음

백트래킹도 조건에 따라 모든 노드를 방문할 수 있지만 일반적으로 조건에 따라 가지치기를 하게 됨

백트래킹은 불필요한 탐색을 줄인다는 점에서 시간복잡도가 많이 줄어듦
- DFS의 경우 N! 가지의 경우의 수를 가진 문제에 대해 해결이 불가능함
- 백트래킹의 경우 가지치기를 통해 조기 차단 가능, 하지만 최악의 경우에는 처리 불가
- 따라서 대규모 문제를 효과적으로 해결 가능함

## 백트래킹의 핵심

백트래킹의 핵심은 가지치기(pruning)를 얼마나 효율적으로 했냐임
- 최적화문제, 결정 문제에서 많이 사용

다중 for문을 써야할 때 많이 사용

가지치기는 해로 가는 도중 막히게 되면 더이상 진행하지 않도록 하는 <b>조건</b>임

보통은 재귀를 통해서 구현됨

## 백트래킹의 기본적인 코드구조

백준의 n과 m(1)을 통해 익혀 보면 쉬움

```cpp
#include <bits/stdc++.h>
using namespace std;

int m, n;
bool isused[9] = {false, };
int arr[9];
int k=1;

void recursive(int count){
    if(count == m){
        for(int i=0; i<m; i++) cout << arr[i] << " ";
        cout << "\n";
    }
    
    for(int i=1; i<=n; i++){
        if(!isused[i]){
            arr[count] = i;
            isused[i] = true;
            recursive(count + 1);
            isused[i] = false;
        }
    }
}


int main(){
    cin >> n >> m;

    recursive(0);

    return 0;
}
```

위코드 에서 알 수 있듯이 재귀함수를 통해 구현하게 됨    
재귀함수를 구현할 때는 base condition(탈출 조건)을 잘생각해줘야함

위 코드의 경우 가지치기를 해준 부분은 아래와 같음
```cpp
if(!isused[i]){
            arr[count] = i;
            isused[i] = true;
            recursive(count + 1);
            isused[i] = false;
        }
```

해당 문제는 순열을 출력하는 문제임    
재귀를 돌다가 만약 해당 숫자를 이미 사용한 경우 해당 숫자는 다시 사용할 필요가 없음   
따라서 isused 배열을 통해 가지치기를 해줄 수 있음    
isused는 배열이기 때문에 O(1)만에 접근 가능하기에 시간복잡도에서도 유리함
