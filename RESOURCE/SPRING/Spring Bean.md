# Spring Bean

Spring IoC Container에 의해 인스턴스화 되고 설정되고 조립되어 관리되는 객체를 말한다.

## Bean Scope

컨테이너에서 Bean의 생명주기 및 범위를 정하는 방식이다.  
Spring에서 지원하는 스코프는 다음과 같다.  

- singletone
- prototype
- request
- session
- application
- websocket

### Singleton (싱글톤) 스코프

- 스프링 IoC 컨테이너당 **단 하나의 인스턴스**만 생성
- 별도의 설정이 없다면 해당 방식으로 모든 빈은 싱글톤으로 생성하여 **메모리를 절약**
- **상태를 공유**하기 때문에 동시성 문제에 주의

> 일치 여부는 해당 빈 정의와 일치하는 ID를 가진 빈에 대한 모든 요청은 스프링 컨테이너에 의해 특정 빈인스턴스 하나만 반환한다.

![Pasted image 20260303203750](../../GALLERY/Pasted%20image%2020260303203750.png)

#### xml

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

### Prototype

- 특정 빈에 대한 요청이 있을 때마다 새로운 인스턴스를 생성하여 반환
	- `getBean()` 호출 시
	- 의존성 주입 시
- 다른 스코프와 달리 **라이프사이클을 관리하지 않음**
	- 클라이언트 코드는 프로토타입 스코프 객체를 정리하고 프로토타입 빈이 보유한 비용이 많이 드는 리소스를 해제할 필요가 있음
	- 

스프링 컨테이너는 프로토타입 빈을 생성하고 의존성을 주입한 까지만 관리하며, 그 이후의 관리(소멸 등)는 빈을 받아간 클라이언트가 책임져야 합니다.

> 상태를 저장하는 빈에는 프로토타입 스코프를, 상태를 저장하지 않는 빈에는 싱글턴 스코프를 사용하는 것이 좋다.

![Pasted image 20260303203757](../../GALLERY/Pasted%20image%2020260303203757.png)

#### xml

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

### Web

- 스프링 웹 환경(`WebApplicationContext`)에서만 사용할 수 있는 특수 스코프
- `request`: 각각의 HTTP 요청마다 새로운 빈을 만듭니다.
- `session`: HTTP 세션(사용자 로그인 상태 등)마다 하나의 빈을 만듭니다.

> TODO : 더 자세히 알아보기


## Bean Lifecycle

### 흐름 요약

객체 생성 → 의존성 주입 →  초기화 콜백 → 사용 →  소멸 전 콜백

#### 세부 흐름 (커스텀 환경)

1. 빈 정의(BeanDefinition) 로딩
2. 빈 인스턴스 생성 (new)
3. 의존성 주입(DI)
4. _Aware 계열 콜백_ (컨테이너 정보 주입)
5. **BeanPostProcessor의 before 초기화 처리**
6. **초기화(Init)**: `@PostConstruct` → `InitializingBean#afterPropertiesSet()` → `init-method` (설정에 따라 순서/여부가 달라짐)
7. **BeanPostProcessor의 after 초기화 처리**
8. 사용(ready)
**(컨테이너 종료)**  
9. **소멸(Destroy)**: `@PreDestroy` → `DisposableBean#destroy()` → `destroy-method`

| **단계**                      | **주요 작업**   | **특징**                                       |
| --------------------------- | ----------- | -------------------------------------------- |
| **Instantiate**             | 빈 객체 생성     | `new` 키워드 단계                                 |
| **Populate Properties**     | 의존성 주입 (DI) | `@Autowired`, Setter 주입 등                    |
| **Aware Callbacks**         | 인프라 정보 주입   | `BeanNameAware`, `ApplicationContextAware` 등 |
| **Post-processor (Before)** | 초기화 전 처리    | `@PostConstruct` 처리 등                        |
| **Initialization**          | 커스텀 초기화     | `afterPropertiesSet`, `init-method`          |
| **Post-processor (After)**  | 초기화 후 처리    | **AOP 프록시 생성** 핵심 단계                         |
| **Ready**                   | 사용 가능       | 애플리케이션 로직 수행                                 |
| **Pre-Destroy / Destroy**   | 자원 정리       | `@PreDestroy`, `destroy-method`              |

### 생성 및 주입

- Spring은 [빈 정의](Bean%20Definition.md)를 바탕으로 객체를 생성
- 생성 후에 `@Autowired` / 생성자 주입 / setter 주입 등으로 **DI가 완료**
- 이 시점 이후부터 “의존성이 모두 채워진 상태”가 된다

### Aware 콜백

빈이 원하면 Spring이 컨테이너 정보를 주입해주는 단계

- `BeanNameAware`: “내 빈 이름이 뭔지”
- `ApplicationContextAware`: “ApplicationContext를 달라”
- `EnvironmentAware`: “프로퍼티/환경 정보를 달라”
    
### BeanPostProcessor (초기화 전/후 가로채기)

- 빈 초기화 **전, 후**에 가로채서 추가 작업 수행
- 대표적으로 AOP 프록시 생성(`@Transactional`), `@Autowired` 처리, `@Async` 프록시 등 많은 기능이 이 레벨에서 발생

### 초기화

- **DI가 끝난 뒤, 빈이 사용 가능 상태가 되기 위한 준비**를 하는 구간
- DB 커넥션 테스트, 캐시 로딩, 외부 클라이언트 초기화 같은 작업 수행

#### 초기화 방식

- `@PostConstruct`
- `InitializingBean#afterPropertiesSet()`
- `init-method` (XML 또는 `@Bean(initMethod="...")`)

### 소멸

컨테이너가 내려갈 때 “자원 해제”를 하는 단계

- 스레드 풀 종료
- 커넥션/소켓 닫기
- 파일 핸들 정리
- 임시 리소스 제거

#### 소멸 방식

- `@PreDestroy`
- `DisposableBean#destroy()`
- `destroy-method` (`@Bean(destroyMethod="...")`)

### 생명주기 제어 (초기화/소멸) 방식 3가지

##### 어노테이션 방식 (`@PostConstruct`, `@PreDestroy`)

- 가장 **권장되는 최신 방식**
- 의존성 주입이 완료된 직후 실행할 메서드에 `@PostConstruct`를, 컨테이너가 종료되어 빈이 소멸되기 직전 실행할 메서드에 `@PreDestroy`를 명시
- 자바 표준(JSR-250)이므로 스프링 외의 컨테이너에서도 동작하며 코드가 간결

```java
@PostConstruct  
public void initAnnotation() {  
    System.out.println("[5-1] @PostConstruct 호출");  
}

@PreDestroy  
public void destroyAnnotation() {  
    System.out.println("[9-1] @PreDestroy 호출");  
}
```

##### 인터페이스 방식  (`InitializingBean`, `DisposableBean`)

- 스프링 초기에 사용하던 방식
- 빈이 스프링 전용 인터페이스를 상속받아야 하므로, **객체가 스프링 프레임워크에 강하게 결합**(Coupling)된다는 단점이 있어 **현재는 잘 사용하지 않음**

```java
public class LifeCycleBean implements InitializingBean, DisposableBean {

	@Override  
	public void afterPropertiesSet() throws Exception {  
	    System.out.println("[5-2] InitializingBean.afterPropertiesSet() 호출");  
	}
	
	@Override  
	public void destroy() throws Exception {  
	    System.out.println("[9-2] DisposableBean.destroy() 호출");  
	}
	
}
```

##### 사용자 지정 초기화/소멸 메서드 (`@Bean(initMethod, destroyMethod)`)

- 외부 라이브러리처럼 **내가 코드를 직접 수정할 수 없는 클래스**에 초기화/소멸 작업을 지시해야 할 때 사용
- 예를 들어 `@Bean(initMethod = "start", destroyMethod = "close")` 형태로 설정 파일에 명시

```java
@Configuration  
public class LifeCycleConfig {  
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")  
    public LifeCycleBean lifeCycleBean() {  
        return new LifeCycleBean();  
    }
}
```


### Scope에 따른 생명주기

- **singleton(기본)**
    
    - 컨테이너 시작 시점에 생성되는 경우가 많고(기본 eager),
    - 컨테이너 종료 시 destroy가 호출
        
- **prototype**
    
    - 요청할 때마다 새로 생성
    - **소멸(destroy)은 Spring이 자동 호출하지 않습니다.**  
        → 즉, prototype 빈이 잡은 자원 해제는 개발자가 직접 수행
        
- **request / session**(웹 스코프)
    
    - HTTP 요청/세션 생명주기에 묶여서 생성/소멸

# 출처

[Spring docs - Bean Overview](https://docs.spring.io/spring-framework/reference/core/beans/definition.html)  
[Spring docs - Bean Scope](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)