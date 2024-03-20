---
title: "[C++] Dijkstra(다익스트라) - 최단거리"
date: 2024-03-02 +09:00
categories:
  - Study
  - Algorithm
tags:
  - 알고리즘
  - 다익스트라
---
## Dijkstra(다익스트라)란?
---
다익스트라 알고리즘은 최단 경로를 찾는 알고리즘 중 하나임

이 알고리즘을 사용할 때는 주의할 특징들이 있음
- 방향그래프이건 무방향 그래프이건 상관이 없음
- 간선의 값이 음수이면 안됨

## Floyd vs Dijkstra
---
두 알고리즘 모두 최단 경로를 찾는 것은 동일함    
하지만 아래와 같은 차이가 있음
- 간선이 음수여도 되는가
	- 플로이드 - 음수여도 되지만 사이클이 있으면 안됨
	- 다익스트라 - 음수이면 안됨 (벨만포드 알고리즘 쓰면 가능, 다루진 않음)
- 어떤 최단거리를 구하는가
	- 플로이드 - 모든 정점 사이의 최단거리를 구함
	- 다익스트라 - 한 정점에서 다른 정점들과의 최단거리를 구함
	- (기능은 플로이드가 좋은데 효율성은 다익스트라가 더 좋음)


## Dijkstra 시간 복잡도
---
처음 다익스트라님이 만든 알고리즘임   
이때는 <b>O(V<sup>2</sup> + E)</b>였음

근데 더 개선이 가능함   
(보통 개선된 방법으로만 통과될 수 있게 만듦)    
그러면  <b>O(E logE)</b>가 됨

## O(V<sup>2</sup> + E) 방식 - 사용 X
---
1. 최단거리 테이블에서 가장 작은 노드를 고름 (이미 거리가 고정된 경우 패스)
	- 시작점에서 가장 가까운 노드를 찾고 확정 시키는 것(일종의 그리디)
2. 해당 노드를 고정시킴
3. 해당 노드에서 뻗어나가는 모든 노드를 체크 후 최단거리 테이블을 업데이트 함
4. 위의 1~3 과정을 반복함

아래의 예시를 보면 더 이해가 쉬움

![](images/2024-03-02-Algorithm-Dijkstra-1.png)

- 처음에 1을 고르게 됨 -> 1을 고정시킴 -> 2, 3, 4를 업데이트 함
- (1)은 고정이고 가장 작은게 3임 -> 3을 고정시킴 -> 4를 업데이트 함(5에서 4로 변경)
- (1, 3) 고정이고 2가 가장 작음 -> 2를 고정시킴 -> 5업데이트
- (1, 3, 2) 고정이고 4가 가장 작음 -> 4를 고정시킴 -> 5랑 비교하는데 5+6 == 3+8 == 11 이라 그대로 둠
- (1, 3, 2, 4) 고정이고 5가 가장 작음 -> 5를 고정시킴 -> 6업데이트
- 6이 가장 작음 -> 6에서 뻗어 나가는거 없음

추가로 뻗어나가는게 없는 경우 외에 d[s]가 INF인 경우 탈출 함   
이 경우 해당 노드로 오는 경로가 하나도 없다는 말임

## 구현(BOJ-1753) - 시간초과
---
- O(V<sup>2</sup> + E) 코드 (정점이 2만개라 시간초과)
```cpp
#include <bits/stdc++.h>
using namespace std;

int v, e, st;
int u, g, w;
vector<pair<int, int>> vec[20001];
bool fix[20001];
int d[20001];
int INF = 0x3f3f3f3f;

int main(){
    cin >> v >> e >> st;
    fill(d, d+v+1, INF);
    d[st] = 0;

    for(int i=1; i<=e; i++){
        cin >> u >> g >> w;
        vec[u].push_back({w, g});
    }

    while(true){
        int idx = -1;
        for(int i=1; i<=v; i++){
            if(fix[i]) continue;
            if(idx == -1) idx = i;
            else if(d[i] < d[idx]) idx = i;
        }

        if(idx == -1 || d[idx] == INF) break;
        fix[idx] = true;
        for(auto nxt : vec[idx]){
            d[nxt.second] = min(d[nxt.second], d[idx] + nxt.first);
        } 
    }

    for(int i=1; i<=v; i++){
        if(d[i] == INF) cout << "INF" << "\n";
        else cout << d[i] << "\n";
    }

    return 0;
}
```


## O(E logE) 방식 - 우선순위 큐(min 힙) 이용
---
우선순위 큐에 추가될 수 있는 원소의 수를 생각해보면 간선 1개당 1개임 -> E logE

1. 우선순위 큐에 (0, 시작점) 추가
2. 우선순위 큐에서 가장 거리가 작은 정점 선택 (해당 거리가 최단거리 테이블과 다르면 패스)
3. 선택한 정점에서 뻗어나가는 정점들을 업데이트(비교를 통해)
4. 우선순위 큐가 빌 때 까지 2, 3과정을 반복

아래의 예시를 보면 이해가 쉬움

![](images/2024-03-02-Algorithm-Dijkstra-1.png)

* 처음에 (0,1)을 push
* 1과 연결된 2, 3, 4의 최단거리 수정 -> 2, 3, 4를 push
* 2, 3, 4중에 거리가 가장 짧은 3을 pop -> 3과 연결된 4의 최단거리 수정(5->4) -> 4를 push
* 2, 4(5), 4(4) 중 거리가 가장 짧은 2를 pop -> 2와 연결된 5의 최단거리 수정(11) -> 5를 push
* 4(5), 4(4), 5 중 거리가 가장 짧은 4(4)를 pop -> 4(4)와 연결된 5의 최단거리 수정안함(11로 동일)
* 4(5), 5 중 거리가 가장 짧은 4(5)를 pop -> 4(5)는 최단거리 테이블의 4와 값이 다름 -> 패스
* 5만 남아서 pop -> 5와 연결된 6의 최단거리 수정 -> 6을 push
* 6을 pop -> 6과 연결된 노드 없음 -> 종료
## 구현(BOJ-1753) - 통과
---
```cpp
#include <bits/stdc++.h>
using namespace std;

int v, e, st;
int u, g, w;
priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
vector<pair<int, int>> vec[20001];
int d[20001];
int INF = 0x3f3f3f3f;

int main(){
    cin >> v >> e >> st;
    fill(d, d+v+1, INF);
    d[st] = 0;

    for(int i=1; i<=e; i++){
        cin >> u >> g >> w;
        vec[u].push_back({w, g});
    }

    pq.push({d[st], st});
    while(!pq.empty()){
        auto cur = pq.top(); pq.pop();
        if(d[cur.second] != cur.first) continue;
        for(auto nxt : vec[cur.second]){
            if(d[nxt.second] <= d[cur.second] + nxt.first) continue;
            d[nxt.second] = d[cur.second] + nxt.first;
            pq.push({d[nxt.second], nxt.second});
        }
    }

    for(int i=1; i<=v; i++){
        if(d[i] == INF) cout << "INF" << "\n";
        else cout << d[i] << "\n";
    }

    return 0;
}
```