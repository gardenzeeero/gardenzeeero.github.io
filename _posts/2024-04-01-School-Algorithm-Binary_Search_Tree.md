---
title: "[Java] 이진검색트리 (Binary Search Tree)"
date: 2024-04-01 +09:00
categories:
  - School
  - DataStructure
tags:
  - 자료구조
---
## Node
---
**Ordered operation과 Balanced Tree**를 위한 추가 구조를 선언해야한다.

```java
class Node<K, V>{  
    K key; V value;  
    int N;      //자손노드 + 1    int aux;    //AVL, RB 트리에 사용  
    Node<K, V> left, right, parent;  
  
    public Node(K key, V value, int n) {  
        this.key = key; this.value = value;  
        this.N = 1;  
    }  
  
    public int getAux() { return aux; }  
    public void setAux(int aux) { this.aux = aux; }  
}
```


## BST
---
Root 노드에 대한 참조를 하나 선언해줘야 한다.

![](images/2024-04-01-School-Algorithm-Binary_Search_Tree.png)

get, put, keys, delete를 구현해야 한다.    
여기서 당연한 특징이 있는데 해당 메서드를 구현하는데 **필요한 다른 메서드**는 **private나 protected**로 만든다.

### treeSearch
---
우선 **get과 put을 쉽게 구현**하기 위해 treeSearch 메서드를 구현해야한다.

이 메서드는 해당 **키를 갖는 노드 or 순회의 마지막 노드**를 반환한다.
(키를 갖는 노드 - get, 순회의 마지막 노드 - put)

#### 구현
**루트 노드**부터 시작해 `compareTo` 메서드를 이용해 **내려갈 수 있을 때 까지** 내려간다.
더이상 못 내려가면 **해당 노드를 반환**한다.

```java
protected Node<K, V> treeSearch(K key){  
    Node<K, V> x = root;  
      
    while(true){  
        int cmp = key.compareTo(x.key);  
        if(cmp == 0) return x;  
        else if(cmp < 0){  
            if(x.left == null) return x;  
            x = x.left;  
        }else{  
            if(x.right == null) return x;  
            x = x.right;  
        }  
    }  
}
```

### get
---
트리에 있는 key의 value를 반환받는 메서드이다.  
treeSearch를 활용해 구현한다.   
#### 구현
**treeSearch**를 통해 얻는 **Node의 key**와 **매개변수 key**를 비교한다.
- **같다면** 트리에 해당 key가 **있음** -> **value** 반환
- **다르다면** 트리에 해당 key가 **없음** -> **null** 반환
```java
public V get(K key){  
    if(root == null) return null;  
  
    Node<K, V> x = treeSearch(key);  
    if(key.equals(x.key)) return x.value;  
    else return null;  
}
```

### put
---
트리에 새로운 노드를 추가하는 메서드이다.   
treeSearch를 활용해 구현한다.   
#### 구현
**treeSearch**를 통해 얻는 **Node의 key**와 **매개변수 key**를 비교
- **같다면** 트리에 해당 key가 **있음** -> **value** 변경
- **다르다면** 트리에 해당 key가 없음 -> 새로운 노드 **추가**
	- 0보다 **작다면** 해당 노드의 **왼쪽**에 새로운 노드 추가
	- 0보다 **크다면** 해당 노드의 **오른쪽**에 새로운 노드 추가
	- 새로운 노드 **추가 후 새로운 부모 지정**
	- **조상 노드 방문**하며 **N을 1**씩 증가

```java
public void put(K key, V value){  
    if(root == null){ root = new Node<K, V>(key, value); return;}  
    Node<K, V> x = treeSearch(key);  
    int cmp = key.compareTo(x.key);  
    if(cmp == 0) x.value = value;  
    else{  
        Node<K, V> newNode = new Node<K, V>(key, value);  
        if(cmp < 0) x.left = newNode;  
        else x.right = newNode;  
        newNode.parent = x;  
        rebalanceInsert(newNode);  
    }  
}  
  
protected void rebalanceInsert(Node<K, V> newNode){  
    resetSize(newNode.parent, 1);  
}  
  
protected void resetSize(Node<K, V> x, int value){  
    while(x != null){  
        x.N += value;  
        x = x.parent;  
    }  
}
```

### keys
---
트리의 **정렬된 key**들을 리스트로 반환한다.   
이진탐색트리를 **Inorder 순회**하면 정렬된 형태로 나온다는 것을 이용한다.

#### 구현
```java
public Iterable<K> keys(){  
    if(root == null) return null;  
    ArrayList<K> list = new ArrayList<>();  
    inorder(root, list);  
    return list;  
}  
  
private void inorder(Node<K, V> x, ArrayList<K> list){  
    if(x != null){  
        inorder(x.left, list);  
        list.add(x.key);  
        inorder(x.right, list);  
    }  
}
```


### delete
---
delete가 BST의 구현부에서 가장 어렵다.   
경우의 수가 많기 때문이다.

지우려는 노드는 세가지로 나뉜다.
1. **리프노드인 루트노드**
	- 루트를 지우고 끝낸다.
2. **자식이 1개인 루트노드**
	- 자식을 루트노드로 바꾼 후 parent를 null로 조정한다.
3. **자식이 2개인 모든노드**
	- **다음에 올 노드**(y)를 찾아서 **지울 노드 자리에 넣는다.**
	- 기존 y의 자식들(오른쪽에 존재)을 **y의 parent에 연결**시켜준다.
	- 이 경우 노드를 지우고 연결하는 것이 아닌 **다음에 올 노드를 가져와 덮는 것**이다.
4. **자식이1개 이하이고 루트가 아닌 노드**
	- 부모에 자식을 붙여줌

1, 2, 4번의 경우 이해가 쉽다.   
3번의 경우가 이해가 어려울 수 있는데 그림을 통해 보면 쉽다.

![](images/2024-04-01-School-Algorithm-Binary_Search_Tree-1.png)

#### 구현
```java
public void delete(K key){  
    if(root == null) return;  
    Node<K, V> x, y, p;  
    x = treeSearch(key);  
  
    if(key.equals(x.key)) return;  
  
    if(x == root || isTwoNode(x)){  
        if(isLeaf(x)){  //1번 : 리프노드인 루트노드  
            root = null; return;  
        }  
        else if(!isTwoNode(x)){ //2번 : 자식이 1개인 루트노드  
            root = (x.right == null) ? x.left : x.right;  
            root.parent = null;  
            return;  
        }else{ //3번 : 자식이 2개인 모든노드  
            y = min(x.right); //x 다음에 올 노드 찾아서 x 자리로 복사  
            x.key = y.key; x.value = y.value;  
            p = y.parent;  
            relink(p, y.right, y == p.left); //y의 자식(무조건 1개또는 없음)을 relink            rebalanceDelete(p, y);  
        }  
    }else{ // 4번 : 자식이 1개 이하 이고 루트가 아닌 노드  
        p = x.parent;  
        if(x.right == null){  
            relink(p, x.left, x == p.left);  
        }else if(x.left == null){  
            relink(p, x.right, x == p.left);  
        }  
        rebalanceDelete(p, x);  
    }  
}  
  
protected boolean isTwoNode(Node<K, V> x){  
    return x.left != null && x.right != null;  
}  
protected boolean isLeaf(Node<K, V> x){  
    return x.left == null && x.right == null;  
}  
protected Node<K, V> min(Node<K, V> x){  
    while(x.left != null) x = x.left;  
    return x;  
}  
protected void relink(Node<K, V> parent, Node<K, V> child, boolean makeLeft){  
    if(child != null) child.parent = parent;  
    if(makeLeft) parent.left = child;  
    else parent.right = child;  
}  
protected void rebalanceDelete(Node<K, V> p, Node<K, V> deleted){
	resetSize(p, -1);
}
```