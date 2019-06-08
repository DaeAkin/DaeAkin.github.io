---
layout: post
title: 나만의 LinkedList 만들어보기
categories: [자료구조]
comments: true
image:
feature: linkedlist.png
---
# LinkedList 만들기

## LinkedList란

<u>ArrayList</u> 는 삽입과 삭제를 수행할 때 연속적인 특징을 깨뜨리지 않기 위해 배열을 Shift 해줘야 합니다. 

이 때 발생하는 시간복잡도는 **O(n)** 입니다.

그러나 LinkedLIst를 이용하게 되면, 시간복잡도를 O(1)만으로 끝낼 수 있기 때문에, 

ArrayLIst에 비해 삽입과 삭제가 월등히 빠르다고 할 수 있습니다.

## 구조

LinkedList의 구조는 다음과 같습니다.

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/linkedlist/image1.png?raw=true)

LinkedList는 내부적으로 Node라는 객체를 갖고 있습니다. 

이 Node들을 서로 연결해서 자료를 참조하는 방법 입니다.

Node의 next 변수는 자기자신의 다음으로 올 Node객체를 알고 있습니다. 

그러나 LinkedList 역시 문제가 있습니다.

ArrayList는 논리적 저장순서와 물리적 저장 순서가 일치하기 때문에 원하는 index 번호를 알면 

O(1)의 시간으로 해당 자료에 바로 참조 할 수 있습니다. 

그러나 LinkedLIst는 원하는 자료를 찾기 위해서는 처음 Node부터 검색해봐야 하기 때문에

O(n)의 시간만큼 걸립니다.



## 구현하기

클래스이름은 MyLinkedList로 하겠습니다. 

클래스를 하나 만들고 제너릭 타입을 선언해주세요.

```java
public class MySinglyLinkedList<E> {
}
```

Node 클래스를 내부에 하나 만들어주세요.

```java
private class Node {
  	// 값을 저장할 때 쓰일 변수
		Object data;
  	// 다음 노드를 가르킬 변수
		Node next;
		
		// 처음 LinkedList를 만들때 쓰일 생성자 
		public Node(Object data) {
			this.data = data;
			this.next = null;
		}
		
	}
```

LinkedList는 Node가 몇개 있는지 알려주는 <u>size</u> 변수와 

첫번째 노드를 가르킬 <u>head</u> 변수가 필요합니다.

```java
private int size;
private Node head;
```

### 지금까지의 작업

```java
public class MySinglyLinkedList<E> {
	
	private class Node {
		Object data;
		Node next;
		
		// 처음 LinkedList를 만들때 쓰일 생성자 
		public Node(Object data) {
			this.data = data;
			this.next = null;
		}
		
	}
	private int size;
	private Node head;
	
	//LinkedList의 생성자
	//처음 LinkedList를 만든다면 head는 비어 있다.
	public MySinglyLinkedList() {
		head = null;
		size = 0;
	}
```

## add함수 만들기

```java
public boolean add(E element) {
  // 첫번 째 노드이면,
  if(head == null) {
    head = new Node(element);
  } else {
  // 첫번째 노드부터 탐색
  Node node = head;
  // 헤더의 다음 노드를 계속 탐색하는데, 다음 노드가 null이 아닐 떄 까지 탐색한다.
  for( ; node.next != null; node = node.next) {

  }
  // 탐색을 다했으면 node 객체가 마지막 노드를 가지고 있게 된다.
  node.next = new Node(element);
  }

  size++;

  return true;
}
```

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/linkedlist/image2.png?raw=true)

`add()` 함수는 마지막 Node를 찾아 새로운 Node를 만들어 이어줘야 합니다. 

그러기 위해서는 처음 Node부터 Node.next가 null 을 만날 때 까지 탐색을 해야합니다.

탐색한 노드를 node 변수에 담고, node.next에 새로운 Node객체를 생성해주면 됩니다.

## get함수 만들기

```java
// 몇번째 노드를 가져오고 싶은지
public E get(int index) { 
   if(index < 0 || size <=index)
     throw new IndexOutOfBoundsException();

   Node node = getNode(index);
   return (E) node.data;
}

public Node getNode(int index) {
  Node node = head;

   // index 까지 node를 움직인다.
   for(int i=0; i<index; i++) {
     node = node.next;
   }

   return node;
}
```

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/linkedlist/image3.png?raw=true)

`get()` 함수는 해당 index 값 까지 node를 head부터 돌려서 해당 Node객체를 가져오면 됩니다.

이때 index의 값이 마이너스 값이거나 size보다 같거나크면 예외를 던져줍니다.

## set함수 만들기

```java
// n번째 있는 노드의 값 수정하고
// 원래 들어있던 값 리턴하기
public E set(int index,E element ) {
  //get() 호출하기
  E old = get(index);
  // 커서를 head에 맞춘다.
  Node node = head;
  // index까지 node를 움직인다.
  for(int i=0; i<index; i++) {
    node= node.next;
  }
  // node
  node.data = element;

  return old;
}
```

`set()` 함수는 방금만든 `get()` 함수와 비슷합니다. 

`get()` 함수처럼 index 만큼 for문을 돌려 해당 Node객체를 가르키고 

가르킨 객체의 data를 변경해주면 됩니다.

## indexOf함수 만들기 

```java
// 해당 요소가 몇번 째 노드에 있는지?
public int indexOf(E element) {
  Node node = head;
  int cnt =0;
  for(; node.next != null; node=node.next) {
    if(node.data.equals(element)) {
      return cnt;
    }
    cnt++;
  }
  return -1;
}
```

`indexOf()` 함수는 Node 객체에 해당 값이 존재하는지 판별하는 객체입니다. 

for문을 돌때마다 Node의 data에 접근하여 `equals` 함수로 비교를 해줍니다. 



## remove 함수

```java
//해당 index의 요소 삭제 중요!
public E remove(int index) {
  E element = get(index);

  Node preNode = getNode(index - 1);

  // 첫번째 노드이면?
  if (index == 0) {
    head = new Node(head.next);
  } else {
    preNode.next = preNode.next.next;
  }

  size--;
  return element;
}
```

`remove()` 함수는 조금 복잡합니다. 

#### 1) 첫번째 노드를 삭제한다면 첫번 째 노드의 다음 노드가 head가 됩니다. 

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/linkedlist/image4.png?raw=true)

#### 2) 중간에 노드를 삭제한다면, 이전 노드와 다음 노드를 서로 이어줘야 합니다. 

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/linkedlist/image5.png?raw=true)

## 코드

[LinkedLIst 구현](https://github.com/DaeAkin/baekjoon/blob/master/algorism/src/datastructure/MySinglyLinkedList.java)



## 마무리

| 구분               | LinkedList |
| ------------------ | ---------- |
| add(끝)            | n          |
| add(시작)          | 1          |
| add(일반적으로)    | n          |
| get/set            | n          |
| indexOf            | n          |
| remove(끝)         | n          |
| remove(시작)       | 1          |
| remove(일반적으로) | n          |



지금까지 단방향 LinkedLIst를 구현해보았습니다. 

다음시간에는 더 많은 장점을 가진 양방향 LinkedList로 찾아뵙겠습니다.
