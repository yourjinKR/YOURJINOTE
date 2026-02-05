## Comparable과 Comparator의 차이를 설명해주세요

 `Comparable`과 `Comparator`는 모두 [객체의 정렬 기준을 정의](../../../RESOURCE/JAVA/Java-객체비교.md)하는 데 사용되는 인터페이스입니다.  
 가장 큰 차이점은 `Comparable`은 객체의 **'기본 정렬 기준(Natural Ordering)'을 정의**하여 해당 클래스 내부에서 구현하는 반면, `Comparator`는 **'특정한 정렬 기준'을 별도로 정의**할 때 사용한다는 점입니다.  
 구현하는 메서드 또한 `Comparable`은 `compareTo(T o)`를, `Comparator`는 `compare(T o1, T o2)`를 사용합니다.