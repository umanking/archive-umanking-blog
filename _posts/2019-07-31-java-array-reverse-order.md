---
layout: post
title: "[Java]  배열 reverseOrder 하는 방법" 
categoires: [java]
date: 2019-07-31 12:28:33
tags: 
- Java
redirect_from: 
- 2019/07/31/java-array-reverse-order/
---

> Java 배열을 reverOrder정렬 하는 방법



```java
//기본 정렬
private void arraySort(){
  int[] intArray = new int[]{5,4,2,1,3};
	
  //순차 정렬
  Arrays.sort(intArray);
  System.out.println(Arrays.toString(intArray)); // [1,2,3,4,5]
  
  // reverse 정렬
  Arrays.sort(intArray, Collections.reverseOrder()); //컴파일 에러
  System.out.println(Arrays.toString(intArray)); 
}
```

알고리즘 문제를 풀다가, int[] 배열을 reverse 정렬해야하는 상황이 있었는데, 당연히 10번라인 처럼 코딩하면 될줄 알았는데, 다음과 같이 컴파일 에러가 발생했다. 



![](/images/reverseOrder.png)

인텔리J의 도움을 받아서 해석해보니,  sort 메서드의 첫번째 인자가 T[] 이걸 필요로 하고, 발견된건 int[] 타입이라서 그런다고 한다. 해결책을 찾아보니

int[] -> Integer[] 로 변환 시키면, 해당 함수를 사용할 수 있다.

```java
Integer integerArray = Arrays.stream(intArray).boxed().toArray(Integer::new);

Arrays.sort(integerArray, Collections.reverseOrder());
System.out.println(Arrays.toString(intArray));  [5,4,3,2,1]
```

