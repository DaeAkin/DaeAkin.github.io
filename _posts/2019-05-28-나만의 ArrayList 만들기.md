---
layout: post
title: 나만의 ArrayList 만들어보기
categories: [자료구조]
comments: true
image:
feature: ArrayList.png
---
# 나만의 ArrayList 만들어보기

프로그래밍 하다가 ArrayList를 많이 사용해 보셨을텐데요

우리가 사용하고 있는 ArrayList가 어떻게 만들어졌는지

만들어봄으로써 그 구조를 파악해보도록 하겠습니다. 

## ArrayList란?

가장 기본적인 자료구조이며, 논리적 저장 순서와 물리적 저장 순서가 일치합니다. 

### 장점

그래서 `인덱스` 로 원하는 배열에 접근할 수 있어, 값을 빠르게 찾을 수 있는 장점이 있습니다. 

만약 찾고자 하는 값의 인덱스를 알고 있으면 시간복잡도는 `O(1)`이 됩니다. 

### 단점

하지만 삭제 또는 삽입의 과정에서 <u>배열 마지막을 제외하고</u> 값을 삭제하거나 , 삽입한다면 

다른 자료구조보다 시간이 많이 걸립니다.

예를 들어 다음과 같이 ArrayList가 있다고 가정해보겠습니다.

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/arraylist/image1.jpeg?raw=true){: width="50%" height="50%" float="left"}

여기서 2번째 방에 있는 1의 값을 삭제해보겠습니다.

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/arraylist/image2.jpeg?raw=true){: width="50%" height="50%" float="left"}

이렇게 삭제하게 되면 배열의 연속적인 특징이 깨지게 됩니다. 

즉 빈 공간이 생기게 됩니다. 

이 빈 공간을 없애려면 어떻게 하면 될까요?

<u>삭제된 해당 index부터 한칸 씩 앞으로 땡겨오면 됩니다.</u>

이렇게 말이죠!

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/arraylist/image3.gif?raw=true){: width="50%" height="50%" float="left"}

하지만 이렇게 `shift` 해주는 비용은 생각보다 작진 않습니다.

삭제 기능에 대한 시간 복잡도는 `O(n)` 을 갖게 되며, worst case 에도 `O(n)` 을 갖게 됩니다. 

삽입의 경우도 같은데요, 중간에 값을 넣고 한칸씩 밀어내는 식으로 작동합니다.

## 나만의 ArrayList 만들기 

저는 자바에 익숙하기 때문에 자바로 만들겠습니다. 

자바의 기본 ArrayList는 메소드가 엄청 많은데요, 그중에서 6개만 만들어보도록 하겠습니다. 

- **boolean add(E element)** : 새로운 요소를 추가하는 함수입니다.
- **E get(int index)** : 해당 index의 요소를 가져오고 반환합니다.
- **E set(int index,E element)** : 해당 index의 요소를 element 값으로 변경하고 반환합니다.
- **E remove(int index)** : 해당 index의 요소를 삭제하고 반환합니다.
- **int indexOf(Object element)** : 해당 element의 요소가 몇 번째 있는지 반환 합니다.

## 변수 선언하기

ArrayList는 두 개의 변수가 있습니다.

```java
public class MyArrayList<E>{
	// 배열의 크기
	private int size;
	// 배열
	private E[] array;
 
  ...
}
```

현재 배열이 어디까지 만들었는지 확인하는 `size` 변수와 

미리 배열을 생성해 놓는 `array` 변수 입니다. 

## 생성자 만들기

```java
public class MyArrayList<E>{
	// 배열의 크기
	private int size;
	// 배열
	private E[] array;
	
	public MyArrayList() {
		 size = 0;
		 array = (E[]) new Object[10];
	}
```

처음 array 배열을 10칸을 할당해 줍니다. 

<u>배열의 몇칸을 할당했는데 체크</u>해주는 `size` 변수도 0으로 초기화 해줍니다.

## Add 함수 만들기

이제 본격적으로 ArrayList가 사용하는 함수를 만들어보겠습니다. 

먼저 Add함수를 만들겠습니다.

```java
public boolean add(E element) {
	// 배열의크기가 size보다 커지면 배열을 2배로 늘려준다.
	if(size >= array.length) {
		// 원래 배열의 길이 X 2 만큼 새로운 배열을 생성한 후
		E[] bigger = (E[]) new Object[array.length * 2];
		// 원래 데이터를 새로만든 배열에 복사를 해준다 .
		System.arraycopy(array, 0, bigger, 0, array.length);
		//array 에게 bigger를 할당
		array = bigger;
	}
	array[size] = element;
	size++;
	// 항상 True를 반환하는 이유는 ? 
	return true;
}
```

만약 3를 집어넣는다고 가정했을 때, 

현재 `array.length` 값과 `size` 가 같으면 배열이 가득찼다는 의미이므로,

배열의 길이를 두배로 늘려줘야 합니다.

두배를 늘려주고, 원래있던 배열의 값들도 같이 복사해줘야 하는데, `arraycopy()` 메소드를 사용하시면 됩니다. 

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/arraylist/image4.jpeg?raw=true){: width="50%" height="50%" float="left"}

현재 제가만든 add함수는 항상 `True` 값을 리턴하게 됩니다.


## Get 함수 만들기

`Add()` 함수를 만들었으니, 해당 배열에 접근해서 값을 가져오는 함수를 작성하겠습니다.

Random Access는 ArrayList의 장점으로 접근하고자 하는 index의 번호를 알고 있으면, 

빠르게 접근할 수 있습니다. 이때 시간복잡도는 `O(1)` 이 됩니다.

```java
// ArrayList에 접근하기
public E get(int index) {
  // 찾으려는 요소의 index가 0보다 적거나, 배열을 넘으면 예외를 던진다.
  if(index<0 || index>=size) {
    throw new IndexOutOfBoundsException();
  }
  return array[index];
}
```

만약 index 값이 <u>마이너스값</u>이거나 범위를 넘어선다면,

예외를 던지도록 했습니다.

## Set함수 만들기

`Set()` 함수는 해당 index에 접근 한 후, 값을 변경하는 함수 입니다.

리턴 값으로는 원래 존재하던 값을 리턴해줍니다.

```java
//해당 요소의 ArrayList의 값을 변경하고 원래 있던 값을 return 하기.
public E set(int index,E element) {
	// 유효한 index인지 확인하기 위해서 만들어뒀던 get() 함수를 호출해주자.
	// 바뀐 요소를 리턴해주기 위해서도 get()을 사용했다.
	E old = get(index);
	array[index] = element;
	return old;
}
```
여기서 앞전에 만들었던 `get()` 함수를 재사용 했습니다.

`get()` 함수를 이용함으로써 올바른 index 범위 인지 확인도 해주고

원래 존재하던 값을 뽑아오는 역할도 해줍니다.

## indexOf함수 만들기

해당 값이 ArrayList에 존재하는지 검사해주는 `indexOf()` 함수를 만들겠습니다. 

```java
// 해당 element가 배열에 존재하는지
public int indexOf(Object element) {
	for(int i=0; i<size; i++) {
		if(element.equals(array[i]))
			return i;
	}
	return -1;
}
```

`indexOf()` 함수는 운이 좋으면 대상 객체를 단번에 찾아서 반환하고, 

운이 없다면 모든 배열을 탐색해야 합니다. 

평균적으로 size의 절반을 테스트 하게 됩니다. 

시간복잡도는 `O(n)` 을 갖게 됩니다.

## remove함수 만들기

remove는 간단합니다. 해당 index를 접근하여, 그 요소를 삭제한 후,

삭제한 요소 값을 리턴해주면 됩니다.

만약 중간에 있는 값을 삭제했다면 빈공간이 생기게 됩니다. 

이는 배열의 연속적인 특징이 깨지게 됩니다.

이런 현상을 막기위해서 삭제한 index로부터 뒤에 요소들을 앞으로 땡겨와야 합니다.

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/arraylist/image3.gif?raw=true){: width="50%" height="50%" float="left"}

코드로 짜면 이렇게 됩니다.

```java
public E remove(int index) {
	//여기서 또한 유효한 index인지 확인하기 위해 get()함수를 호출해주자.
	// 삭제된 요소를 리턴해주기 위해서도 get()을 사용했다.
	E removed = get(index);
	for(int i = index; i<size-1; i++) {
		// 요소들을 한칸씩 땡겨주기.
		array[i] = array[i+1];
	}
	//배열 크기 1 감소
	size--;
	return removed;
}
```
remove는 맨 끝에 삭제하게되면 `shift` 할 비용이 없으므로 시간복잡도가 `O(1)` 을 가집니다.

맨 앞을 삭제하게 되면 전부다 `shift` 하게 되므로 시간복잡도가 `O(n)` 을 가집니다.

평균적으로 `O(n)` 을 갖게 됩니다.

## toString 구현하기

이제 만들어진 ArrayList가 정상적으로 작동하는지 출력하기 위해 

toString함수를 구현하도록 하겠습니다.

```java
@Override
public String toString() {
  //StringBuffer 와 StringBuilder 와 new String 와 String = ""의 차이점? 
  StringBuilder sb = new StringBuilder();
  sb.append("[");
  for(int i=0; i<size; i++) {
    sb.append(array[i]);
    //마지막 요소에는 콤마(,)를 제외해준다.
    if(i != size-1)
    sb.append(",");
  }
  sb.append("]");

  return sb.toString();
}
```

여기서 StringBuilder를 사용했는데, 

StringBuffer / StringBuilder 와 차이는 무엇이고,

String 객체를 만들 때 new String 과 String = ""의 차이는 무엇일까요?

추후에 포스팅으로 알려드리도록 하겠습니다.

## 테스트

```java
public class Test {
	
	public static void main(String[] args) {
		MyArrayList<Integer> list = new MyArrayList<>();
		
		list.add(0);
		list.add(1);
		list.add(2);
		list.add(3);
		list.add(4);
		
		list.remove(3);
		
		System.out.println("1의 위치는? :" +list.indexOf(1));
		
		list.set(0, 10);
		
		System.out.println(list.toString());
		
	}
}
```

### Output

```java
1의 위치는? :1
[10,1,2,4]
```

## 마무리

지금까지 자료구조의 기본인 ArrayList에 대해서 알아보았습니다. 

ArrayList는 자료검색에는 빠르지만, 잦은 삽입과 삭제가 이루어진다면 

시간복잡도는 `O(n)` 이기 때문에 느릴수 밖에 없습니다.

만약 빠른 자료구조를 찾고싶다면 LinkedList 자료구조를 사용하시는걸 추천드립니다. 

LinkedList는 다음 포스팅에서 다뤄보겠습니다.

## 전체코드


```java
package datastructure;

import java.util.List;

public class MyArrayList<E>{
	// 배열의 크기
	private int size;
	// 배열
	private E[] array;
	
	public MyArrayList() {
		 size = 0;
		 array = (E[]) new Object[10];
	}
	
	public boolean add(E element) {
		// 배열의크기가 size보다 커지면 배열을 2배로 늘려준다.
		if(size >= array.length) {
			// 원래 배열의 길이 X 2 만큼 새로운 배열을 생성한 후
			E[] bigger = (E[]) new Object[array.length * 2];
			// 원래 데이터를 새로만든 배열에 복사를 해준다 .
			System.arraycopy(array, 0, bigger, 0, array.length);
			//array 에게 bigger를 할당
			array = bigger;
		}
		array[size] = element;
		size++;
		// 항상 True를 반환하는 이유는 ? 
		return true;
	}
	
	// ArrayList에 접근하기
	public E get(int index) {
		// 찾으려는 요소의 index가 0보다 적거나, 배열을 넘으면 예외를 던진다.
		if(index<0 || index>=size) {
			throw new IndexOutOfBoundsException();
		}
		return array[index];
	}
	
	//해당 요소의 ArrayList의 값을 변경하고 원래 있던 값을 return 하기.
	public E set(int index,E element) {
		// 유효한 index인지 확인하기 위해서 만들어뒀던 get() 함수를 호출해주자.
		// 바뀐 요소를 리턴해주기 위해서도 get()을 사용했다.
		E old = get(index);
		array[index] = element;
		return old;
	}
	

	// 해당 element가 배열에 존재하는지
	public int indexOf(Object element) {
		for(int i=0; i<size; i++) {
			if(element.equals(array[i]))
				return i;
		}
		return -1;
	}
	
	// 
	public E remove(int index) {
		//여기서 또한 유효한 index인지 확인하기 위해 get()함수를 호출해주자.
		// 삭제된 요소를 리턴해주기 위해서도 get()을 사용했다.
		E removed = get(index);
		for(int i = index; i<size-1; i++) {
			// 요소들을 한칸씩 땡겨주기.
			array[i] = array[i+1];
		}
		//배열 크기 1 감소
		size--;
		return removed;
	}
	
	@Override
	public String toString() {
		//StringBuffer 와 StringBuilder 와 new String 와 String = ""의 차이점? 
		StringBuilder sb = new StringBuilder();
		sb.append("[");
		for(int i=0; i<size; i++) {
			sb.append(array[i]);
			//마지막 요소에는 콤마(,)를 제외해준다.
			if(i != size-1)
			sb.append(",");
		}
		sb.append("]");
		
		return sb.toString();
	}
}
```

