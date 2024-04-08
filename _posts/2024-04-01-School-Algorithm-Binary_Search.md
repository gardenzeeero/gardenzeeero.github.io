---
title: "[Java] 연결리스트 이용 순차검색, 배열 이용 이진검색"
date: 2024-04-01 +09:00
categories:
  - School
  - DataStructure
tags:
  - 자료구조
---
## 연결리스트 이용 순차검색
---
(Key, Value)로 구성된 **연결리스트**를 만든다.

1. 검색
	- **모든 Key를 스캔**하면서 일치여부 확인
2. 삽입
	- **모든 Key를 스캔**하여 일치여부를 확인 후 **없다면 맨 앞에 삽입**

### 구현
---
node를 Key와 Value로 구성

```java
class Node<K, V>{  
    K key; V value;  
    Node<K, V> next;  
  
    public Node(K key, V value, Node<K, V> next) {  
        this.key = key;  
        this.value = value;  
        this.next = next;  
    }  
}
```

### SequentialSearchST 구현
---

```java
public class SequentialSearchST <K,V>{  
    private Node<K, V> first;  
    int N;  
  
    public V get(K key){  
        for(Node<K, V> x = first; x != null; x = x.next){  
            if(key.equals(x.key)) return x.value;  
        }  
        return null;    //못찾은 경우  
    }  
  
    public void put(K key, V value){  
        for(Node<K, V> x = first; x != null; x = x.next){  
            if(key.equals(x.key)){  
                x.value = value;  
                return;  
            }  
        }  
        first = new Node<K, V>(key, value, first);  
        N++;  
    }  
  
    public void delete(K key){  
        if(key.equals(first.key)){  
            first = first.next; N--;  
            return;  
        }  
        for(Node<K, V> x = first; x.next != null; x = x.next){  
            if(key.equals(x.next.key)){  
                x.next = x.next.next; N--;  
                return;  
            }  
        }  
    }  
  
    public Iterable<K> keys(){  
        ArrayList<K> keyList = new ArrayList<K>(N);  
        for(Node<K, V> x = first; x != null; x = x.next){  
            keyList.add(x.key);  
        }  
        return keyList;  
    }  
  
    public boolean contains(K key){return get(key) != null; }  
    public boolean isEmpty(){ return N == 0; }  
    public int size(){ return N; }  
}
```

1. get
	- **first부터** 넘어가면서 Key와 **equals 비교**
2. put
	- first부터 넘어가면서 Key와 equals 비교 후 **있다면 value 변경
	- 없다면 first를 **next 매개변수**로 넘겨주고 first를 **새로만든 Node로 지정**
3. delete
	- **first 노드**를 비교 후 Key가 같다면 **first = first.next**로 변경 후 N--
	- 다르다면 **first.next부터** Key 비교, 같다면 **x.next = x.next.next** 후 N--
4. keys
	- Iterable을 구현한 **ArrayList를 반환**
5. contains
	- get을 이용해 **null이 반환되면** false 반환
6. isEmpty, size
	- N을 이용
