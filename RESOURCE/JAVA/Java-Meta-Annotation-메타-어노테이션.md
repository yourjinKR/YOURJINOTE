# 메타 어노테이션

[어노테이션](Java-Annotation-어노테이션.md)을 정의할 때 사용하는 어노테이션이다.  

> 데이터와 메타 데이터의 관계라고 생각하자.

### @Target

어노테이션을 적용할 수 있는 대상의 지정에 사용

### @Retention

어노테이션이 유지되는 기간을 지정

### @Document

어노테이션 정보가 javadoc으로 작성한 문서에 포함

### @Inherited

어노테이션이 자손 클래스에 상속되도록 한다.  

### @Repeatable

하나의 대상에 같은 어노테이션을 여러 번 붙게 할 수 있도록 설정

```java
@Repeatable(ToDodos.class)  
@interface Todo{  
    String value();  
}

// Todo 배열을 갖고 있는 인터페이스를 선언
@interface ToDodos {  
    Todo[] value();  
}
```

### @Native

네이티브 메서드에 의해 참조되는 상수 필드임을 알리는 어노테이션

## 다시 점검할 부분

- `@Retention` 어노테이션의 유지 정책 원리에 대한 이해 
- 직접 커스텀 어노테이션을 만들어 보기
- 