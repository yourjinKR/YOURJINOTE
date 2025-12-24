# 템플릿 메소드 패턴

알고리즘의 전체 흐름을 상위 클래스에서 정의하고 세부 단계의 구현은 하위 클래스에서 결정하도록 하는 패턴이다.

- 공통적인 처리 흐름은 고정
- 변하는 부분만 서브 클래스만 확장

## 사용 시점

- 특정 알고리즘 구조를 유지해야 할 때
- 일부 메서드는 구현, 일부 메서드는 구현하지 않을 경우
- 여러 하위 클래스가 메서드 재정의 방식이 동일한 경우 (중복 코드가 많을 경우)

> 추상 메서드가 굉장히 많고 그 중 일부분은 정해진 순서대로 호출되어야 하는 클래스(혹은 인터페이스)가 있다고 할 때  해당 인터페이스를 구현하는데 있어서는 많은 노력이 필요할  것이다.  
> 이때 템플릿 역할을 하기 위한 메서드가 작성된 클래스를 구현하고 나머지 메서드는 해당 클래스를 추가적으로 확장(extends)하여 나머지 메서드를 재정의 하는 방식으로 사용할 수 있을 것이다.

## 장단점

- 중복 코드 제거
- 알고리즘 구조 일관성 유지
- 변경에 강함
- 상속에 의존하므로 유연성 제한
- 상위 클래스 변경 시 하위 클래스 영향 발생

## 핵심 구조

### Abstract Class

- 템플릿 메서드 정의
- 공통 로직 구현
- 추상 메서드 선언

> 템플릿 메서드는 보통 `final`로 선언하여 알고리즘의 구조 변경을 방지한다.  
> 즉, 상위 클래스가 흐름을 제어하는 것이다.

### Conctete Class

- 추상 메서드 구현
- 필요한 부분만 재정의

## 예제

```java
package org.example.selfmade.designPattern.templateMethod;  
  
public abstract class DataProcessor {  
  
    public final void process() {  
        readData();  
        processData();  
        saveData();  
    }  
  
    protected abstract void readData();  
    protected abstract void processData();  
  
    protected void saveData() {  
        System.out.println("데이터를 저장함");  
    }  
}
```

```java
package org.example.selfmade.designPattern.templateMethod;  
  
public class CsvDataProcessor extends DataProcessor {  
    @Override  
    protected void readData() {  
        System.out.println("csv 파일을 읽음");  
    }  
  
    @Override  
    protected void processData() {  
        System.out.println("csv 데이터를 처리함");  
    }  
}
```

```java
package org.example.selfmade.designPattern.templateMethod;  
  
public class JsonDataProcessor extends DataProcessor {  
    @Override  
    protected void readData() {  
        System.out.println("json 파일 읽음");  
    }  
  
    @Override  
    protected void processData() {  
        System.out.println("json 파일을 처리함");  
    }  
}
```

```java
package org.example.selfmade.designPattern.templateMethod;

import java.util.List;

public class TemplateMethodPatternEx {
    public static void main(String[] args) {
        List<DataProcessor> dataProcessors = List.of(
                new CsvDataProcessor(),
                new JsonDataProcessor()
        );

        dataProcessors.forEach(DataProcessor::process);

        /*

        csv 파일을 읽음
        csv 데이터를 처리함
        데이터를 저장함
        json 파일 읽음
        json 파일을 처리함
        데이터를 저장함

         */
    }
}
```

## 대표 예시

- Java의 [Abstract Collection](../JAVA/Java-Collection-Framework.md)
- Spring의 AbstractController
- JUnit의 `setUp()`, `tearDown()`

# 출처

https://refactoring.guru/ko/design-patterns/template-method  
[티스토리 - 템플릿 메소드 패턴에 대해](https://coding-factory.tistory.com/712)

