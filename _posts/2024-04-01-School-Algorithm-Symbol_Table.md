---
title: "[Java] Symbol Table"
date: 2024-04-01 +09:00
categories:
  - School
  - DataStructure
tags:
  - 자료구조
---
## Symbol Table
---
(Key, Value) 쌍의 모임이다.  
특정 키와 그 키에 해당되는 값의 쌍을 삽입한다.  
키가 주어질 때, 관련된 값을 검색한다.

## Symbol Table API
---
![](images/2024-03-17-School-Algorithm-Symbol_Table.png)

1. **Generics**를 지원해야 한다.
	- Key는 **Comparable** 인터페이스를 구현해야 한다.(이진검색)
2. 중복 Key는 **허용하지 않는다.**
	- get(key1): key1이 없으면, **null 반환**
3. Key와 Value는 **null이 될 수 없다.**
4. 테이블에서 Key, Value를 삭제하는 경우 **실제로 Key를 검색**해 **삭제**해야 한다.


## 참고자료
---
영남대학교 조행래 교수님 - 알고리즘 강의자료