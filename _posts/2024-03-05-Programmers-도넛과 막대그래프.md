---
title: "[C++] Programmers: 도넛과 막대그래프"
date: 2024-03-05 +09:00
categories:
  - Study
  - Programmers
tags:
  - 프로그래머스
  - DFS
---
[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/258711)

## 알고리즘 분류
DFS(약간의 변형)

## 알게된 점
DFS인건 알고 있었지만 생성된 정점 찾기가 힘들었음

indegree와 outdegree를 이용하는 것이  생소했음

도저히 못 풀겠어서 찾아봄 아직 많이 모르구나라는걸 또 한번 깨닫게 됨

그리고 조건을 좀 꼼꼼히 읽어야했음 조건 안읽고 나혼자 반례때문에 안되네 이러고 있었음

## 해설

생성된 정점만 찾으면 생각보다 간단함    
근데 생성된 정점 찾는게 어려움

1. edge를 돌면서 outdegree와 indegree의 수를 저장함
2. outdegree가 2이상이고(조건에 그래프 개수가 2개이상이라 함) indegree가 0인걸 찾으면 됨

생성된 정점의 특징이 명확해서 생각보다 찾기 쉬웠는데 그래프 개수가 2개 이상이라는 조건을 못봄...   
생성된 정점의 특징은 아래와 같음
- 도넛, 막대 그래프에선 outdegree가 2이상인 것이 없음   
- 8자 그래프에선 outdegree가 2인것이 있지만 indegree가 존재함
- 따라서 생성된 정점은 outdegree가 2이상이고 indegree가 없는 노드임

생성된 정점을 찾았다면 그래프마다 dfs를 돌면 됨

이때 무슨 그래프인지 판단을 해야함   
- 도넛 그래프 : 방문했던 노드를 다시 방문하면 도넛그래프
- 막대 그래프 : outdegree가 없는 노드를 만나면 막대그래프
- 8자 그래프 : outdegree가 2인 노드를 만나면 8자 그래프

처음에 그럼 8자 dfs를 돌면 도넛 그래프가 나올 수 있는거 아닌가? 라는 생각을 함    
하지만 그럴일이 없음

도넛그래프가 될려면 방문했던 노드를 방문해야 함    
하지만 그전에 무조건 outdegree가 2이상인 그래프를 만나게 되어있음

8자 그래프는 도넛그래프를 두개 합친것이기 때문에 한바퀴 돌기전에 무조건 겹치는 노드를 만남

## 코드
```cpp
#include <string>
#include <vector>
using namespace std;

int indegree[1000000] = {0, };
int outdegree[1000000] = {0, };
bool visited[1000000] = {false, };
vector<int> v[1000000];
vector<int> answer;

void dfs(int node){
    if(visited[node]) {answer[1]++; return;}
    if(outdegree[node] == 2) {answer[3]++; return;}
    if(outdegree[node] == 0) {answer[2]++; return;}
    
    visited[node] = true;
    for(auto child : v[node]){
        dfs(child);
    }
}

vector<int> solution(vector<vector<int>> edges) {
    answer.resize(4, 0);
    
    for(int i=0; i<edges.size(); i++){
        v[edges[i][0]].push_back(edges[i][1]);
        outdegree[edges[i][0]]++;
        indegree[edges[i][1]]++;
    }
    
    for(int i=0; i<1000000; i++){
        if(indegree[i] == 0 && outdegree[i] >= 2){
            answer[0] = i;
            break;
        }
    }
    
    for(auto child : v[answer[0]]) dfs(child);
    
    return answer;
}
```
