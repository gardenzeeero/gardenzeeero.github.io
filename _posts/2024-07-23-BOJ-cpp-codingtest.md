---
title: "[C++] 코테용 cpp 사용법"
date: 2024-07-23 +09:00
categories:
  - Study
  - Baekjoon
tags:
  - cpp
pin: false
---

### set
Java의 TreeSet에 해당 (Red_black tree로 구현), 이진 검색트리가 필요할 때 사용
중복 허용 X
```cpp
set<int> s;
s.insert(10);

s.insert(10); s.insert(20);
cout << s.erase(10); //10 있으니까 지우고 1 반환
cout << s.erase(40); //40 없어서 0 반환

//있으면 해당 iterator 반환, 없으면 s.end() 반환
if(s.find(20) != s.end()) cout << "20 is found";

s.size();    //들어있는 수
s.count(50); //개수 세는건데 0 아니면 1

//정렬된 순서대로 나옴
for(auto e : s){
	cout << e << ' ';
}
//이 까지는 unordered_set이랑 동일

set<int>::iterator it1 = s.begin(); it1++;
auto it2 = prev(it1);               //it1 바로 전 원소 가리킴
it2 = next(it1);                    //it1 바로 다음 원소 가리킴

auto it3 = s.lower_bound(-20); //-20 보다 크거나 같은 수 중 처음나온거
auto it4 = s.upper_bound(-20); //-20 보다 큰 수 중 처음 나온거

//둘다  s.end()를 가리킬 수 있어서 조심해야함
```

### unordered_set
Java의 HashSet에 해당    
중복 허용 X   
```cpp
unordered_set<int> s;

s.insert(10); s.insert(20);
cout << s.erase(10); //10 있으니까 지우고 1 반환
cout << s.erase(40); //40 없어서 0 반환

//있으면 해당 iterator 반환, 없으면 s.end() 반환
if(s.find(20) != s.end()) cout << "20 is found";

s.size();    //들어있는 수
s.count(50); //개수 세는건데 0 아니면 1

// 무작위 순서로 막 나옴
for(auto e : s){
	cout << e << ' ';
}
```

### multiset
set이랑 큰 차이 없지만 중복 가능     
erase할 때 그냥 하면 다 지워지니까 find로 해야 하나만 지워짐    
이것도 lower_bound, upper_bound 쓸 때 조심해야함
```cpp
multiset<int> m;

m.erase(m.find(-10));
```
### unordered_multiset
unordered_set이랑 거의 동일 -> 중복된 원소가 여러개 들어감
메서드는 동일 근데 count랑 출력했을 때 값들이 달라지겠지   
erase할 때 조심해야함
```cpp
unordered_set<int> s;

s.insert(10); s.insert(10);
cout << s.erase(10); //모든 10을 지워버림

s.erase(s.find(10)); //이렇게 하면 처음 만난거만 지움
```

### map
Java의 TreeMap과 동일, 이진 탐색 트리   
다른건 다 unordered_map과 동일하고 출력시 정렬순서대로 출력

### unordered_map
Java의 HashMap이랑 동일, 중복키 허용 X
```cpp
unordered_map<string, int> m;

m["hi"] = 123;
m["bkd"] = 1000;

cout << m.size(); //개수 출력가능
m["hi"] = -1;     //hi의 값이 123 -> -1

//못찾으면 m.end()반환
if(m.find("hi") != m.end()) cout << "hi is found"

m.erase("hi");
m.clear();

//key = first, value = second
for(auto e : m){
	cout << e.first << " " << e.second;
}
```