# 우선 순위 큐

우선 순위 큐는 특별한 유형의 [[Queue|큐]]이다.  
각 큐의 항목에는 우선순위라는 추가 정보가 있으며 일반 큐와는 달리 선입 선출 방식이 아닌 우선순위에 따라 항목이 제거된다.

```java
Queue<Integer> queue = new PriorityQueue<>();  
  
queue.add(5);  
queue.add(1);  
queue.add(2);  
queue.add(4);  
queue.add(3);  
  
System.out.println(queue.poll()); // 1  
System.out.println(queue.poll()); // 2  
System.out.println(queue.poll()); // 3  
System.out.println(queue.poll()); // 4  
System.out.println(queue.poll()); // 5 
System.out.println(queue.poll()); // null
```