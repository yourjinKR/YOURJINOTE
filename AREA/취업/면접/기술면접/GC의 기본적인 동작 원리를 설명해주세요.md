# GC의 기본적인 동작 원리를 설명해주세요

[GC](../../../../RESOURCE/JAVA/Java-가비지-컬렉션-Garbage-Collection.md)의 기본적으로 **도달 가능성**을 기준으로 사용 중인 객체와 가비지를 구분합니다.  
이를 실제 구현한 대표적인 알고리즘이 **Mark And Sweep**으로, 루트에서 연결된 객체들을 마킹하고 연결되지 않은 객체들의 메모리를 쓸어버리는 방식입니다.

성능을 최적화하기 위해 현대의 JVM은 힙 메모리를 Young Generation과 Old Generation으로 관리하는 세대별 가비지 컬렉션 기법을 사용합니다.