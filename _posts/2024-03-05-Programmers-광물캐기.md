---
title: "[C++] Programmers: 광물캐기"
date: 2024-03-05 +09:00
categories:
  - Study
  - Programmers
tags:
  - 프로그래머스
  - 브루트포스
---
[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/172927?language=cpp)

## 알고리즘 분류
브루트 포스, DFS

## 알게된 점
프로그래머스는 입력을 매개변수로 전달해줘서 백준 풀때랑 느낌이 좀 달랐음

자동완성도 없다보니 확실하게 알고 써야해서 그게 좀 귀찮았음

처음에 바보같이 C로 풀다가 25분가량 날려먹어서 좀 아쉬웠음...

## 나의 접근 방법
처음에 그냥 다 해보면 되는거 아닌가 라는 생각이 들었음   
브루트포스로 풀려면 시간복잡도를 계산해봐야 함

minerals의 길이가 최대 50이기 때문에 곡괭이를 많이써도 10개가 최대임

그럼 길이가 10인 서로다른 곡괭이 순열의 개수를 구하면 됨    
그럼 얼마인가? 3<sup>5</sup> * 2<sup>5</sup> 임 대략 한 15000정도 됨

하나의 순열에 대해서 피로도를 계산할 때 모든 mineral을 탐색해야하니 50을 곱해줘야함    
그래도 3<sup>5</sup> * 2<sup>5</sup> * 50 이기에 시간은 충분함

코드의 흐름은 아래와 같음
1. 각 곡괭이의 개수만큼 v에 push 해줌 (v : dia, dia, stone, stone, stone)
2. next_permutation을 이용해 순열을 바꿔주며 find 함수에 진입함
3. 곡괭이의 종류를 판단해 5개씩 광물을 캐줌 (종류에 따라 피로도 계산)
	- 광물이 5개보다 적게 남았을 수 있으니 매번 체크해줌
4. 곡괭이를 다썼거나 광물을 다 캤다면 반복문을 탈출함
5. min값을 비교해 업데이트 함

코드를 주석과 보면 더 이해가 편함

## 코드
```cpp
#include <string>
#include <vector>
#include <bits/stdc++.h>
using namespace std;

vector<char> v;
int minPiro = 10000000;
int picks_size = 0;

//매번 minerals 값의 복사를 막기 위해 &(참조)사용
void find(vector<string> &minerals){
    int temp = 0;
    int gockIdx = 0, mineralIdx = 0;
    while(true){
	    //곡괭이를 다썻거나 광물을 다 캤다면 break;
        if(gockIdx >= picks_size || mineralIdx >= minerals.size()) break;
        
        if(v[gockIdx] == 'd'){    //다이아 곡괭이인 경우
            for(int i=mineralIdx; i<mineralIdx+5; i++){
                if(i >= minerals.size()) break;    //광물이 5개 이하인 경우를 체크
                temp++;
            }
        }else if(v[gockIdx] == 'i'){    //철 곡괭이인 경우
            for(int i=mineralIdx; i<mineralIdx+5; i++){
                if(i >= minerals.size()) break;    //광물이 5개 이하인 경우를 체크
                if(minerals[i] == "diamond") temp += 5;
                else temp++;
            }
        }else{    //돌 곡괭이 인 경우
            for(int i=mineralIdx; i<mineralIdx+5; i++){
                if(i >= minerals.size()) break;    //광물이 5개 이하인 경우를 체크
                if(minerals[i] == "diamond") temp += 25;
                else if(minerals[i] == "iron") temp += 5;
                else temp++;
            }
        }
        mineralIdx += 5;     //광물은 5개씩 증가(광물개수를 넘어가면 반복문 처음에서 break)
        gockIdx++;    //곡괭이는 하나씩 증가
    }
    minPiro = min(minPiro, temp);
}


int solution(vector<int> picks, vector<string> minerals) {
	//곡괭이 개수 만큼 d, i, s를 v에 저장
    for(int i=0; i<3; i++){
        for(int j=0; j<picks[i]; j++){
            if(i==0) v.push_back('d');
            else if(i==1) v.push_back('i');
            else v.push_back('s');
        }
    }
    picks_size = picks[0] + picks[1] + picks[2];

	//next_permutation은 다음 순열을 만들어주니 처음부터 뽑기위해 sort한번
    sort(v.begin(), v.end());
    do{
        find(minerals);
    }while(next_permutation(v.begin(), v.end()));
    
    return minPiro;
    
}
```
