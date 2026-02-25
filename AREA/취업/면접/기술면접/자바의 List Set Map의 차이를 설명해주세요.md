## 자바의 List Set Map의 차이를 설명해주세요

자바의 **컬렉션 프레임워크의 주요 인터페이스**인 List, Set, Map의 차이점은 **데이터 순서 유지**와 **중복 허용 여부**에 차이가 있습니다.

[List](RESOURCE/JAVA/Java-List.md)는 데이터의 저장 순서를 유지하며 중복 저장을 허용합니다.  
인덱스로 데이터를 접근하며, 대표적으로 `ArrayList`와 `LinkedList`가 있습니다.  

[Set](RESOURCE/JAVA/Java-Set.md)는 순서를 보장하지 않으며 데이터의 중복을 허용하지 않습니다.  
중복을 제거할때 주로 사용하며 대표적으로 `HashSet`과 순서를 보장하는 `TreeSet` 등이 있습니다.

[Map](RESOURCE/JAVA/Java-Map.md)은 key와 value의 쌍으로 데이터를 관리합니다. Key는 중복될 수 없지만 Value는 중복이 가능합니다.  
키를 통해 빠른 데이터 탐색을 할 때 주로 `HashMap`이 가장 대표적으로 사용됩니다.   