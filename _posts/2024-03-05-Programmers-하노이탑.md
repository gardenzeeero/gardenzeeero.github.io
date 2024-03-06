---
title: "[C++] Programmers: 하노이탑"
date: 2024-03-05 +09:00
categories:
  - Study
  - Programmers
tags:
  - 프로그래머스
  - 재귀
---
[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/12946)

## 알고리즘 분류
재귀

## 알게된 점
하노이탑 맨날 안다고 생각했는데 코드를 막상 짤려고 하니 헷갈렸음

그래도 뭐 금방 풀어서 낫밷

## 나의 접근 방법
하노이야 뭐 재귀로서 너무나 유명한 문제라 처음부터 재귀로 접근함

3개짜리 탑을 1->3 옮기는 건 아래의 단계로 이루어짐
1. 2개짜리 탑을 1->2 옮김
1. 가장 큰 원판을 1->3 옮김
2. 2개짜리 탑을 2->3 옮김

4개짜리 탑을 1->3 옮기는 것도 동일함
1. 3개짜리 탑을 1->2 옮김
2. 가장 큰 원판을 1->3 옮김
3. 3개짜리 탑을 2->3 옮김

해당 과정을 코드로 옮기면 끝임

주석을 보며 좀 더 이해가 쉬움
## 코드
```cpp
#include <string>
#include <vector>
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> answer;

void hanoi(int n, int start, int with, int end){
    vector<int> temp;
    
    if(n != 1){
        hanoi(n-1, start, end, with);   //n-1개의 탑을 start에서 end를 통해 with로 옮김
        temp.push_back(start);
        temp.push_back(end);
        answer.push_back(temp);         //n번째 원판을 start에서 end로 옮김
        hanoi(n-1, with, start, end);   //n-1개의 탑을 with에서 start를 통해 end로 옮김
        // 해당 과정을 거치면 n개의 탑이 start에서 end로 옮겨짐
    }else{
        temp.push_back(start);
        temp.push_back(end);
        answer.push_back(temp);
    }
}

vector<vector<int>> solution(int n) {
    
    hanoi(n, 1, 2, 3);
    return answer;
}
```
