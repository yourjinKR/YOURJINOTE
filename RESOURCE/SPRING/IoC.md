# IoC (Inversion of Control)

IoC, 제어의 역전은 프로그램의 제어 흐름을 프로그래머가 직접 관리하는 것이 아닌 **프레임워크에서 프로그래머가 작성한 코드를 호출하고 흐름을 제어**하는 것이다.

즉, 객체의 생명주기를 개발자가 직접 다루는 것이 아닌 스프링 프레임워크가 이를 직접 관리한다는 것이다.

프레임워크가 요구하는대로 객체를 생성하면, 프레임워크가 해당 객체들을 가져다가 알아서 생성하고, 메소드를 호출하고, 소멸시킨다.

> 우리가 Controller, Service, Repository를 구현하지만 어느 타이밍에 어떤 녀석이 호출 되는 것은 우리가 정하지 않았다. 즉, 이러한 흐름은 Spring이 제어한다는 것이다.

IoC 이점 → 개발자가 핵심 비지니스 로직 개발에 더 집중할 수 있게 된다.

## IoC와 DI

[DI](RESOURCE/CS/DI-의존성%20주입.md)(Dependency Injection), 의존성 주입은 **IoC 원칙을 실현하기 위한 여러 디자인 패턴 중 하나**이다.  
IoC와 DI는 모두 객체간의 결합을 느슨하게 만들어 유연한 확장성을 지닌 코드를 작성하게 만든다.  

자세한 내용은 [해당 글](Spring-DI.md)을 참고

## IoC Container

Spring Framework가 객체를 관리하는 컨테이너이다.  
객체의 생성, 구성 그리고 전체 생명주기를 관리하는 역할을 수행하며 Spring Container라고도 부른다.

> Spring Container는 보통 Spring의 IoC 컨테이너를 가리키는 비공식·포괄적 용어이다.

- **Bean 생성** : 설정 정보를 바탕으로 객체를 생성
- **의존성 주입** : 객체 간의 관계를 파악하여 필요한 객체를 매핑
- **생명주기 관리** : 

### 구현 방식

Spring은 컨테이너를 추상화하기 위해 두 가지 주요 인터페이스를 제공한다.

- `BeanFactory` : 구성 프레임워크와 기본 기능 제공
- `ApplicationContext` : 엔터프라이즈급 편의 기능을 추가로 제공

#### BeanFactory

- Spring 설정 프레임워크의 가장 기본적인 기능을 제공합니다.
- **Lazy Loading** 전략을 사용하여 빈(Bean)이 요청될 때만 인스턴스화합니다.
- 리소스가 제한적인 모바일 기기나 경량 애플리케이션에 적합합니다.

#### ApplicationContext

- `BeanFactory`의 모든 기능을 포함하며, 엔터프라이즈 기능에 특화된 하위 인터페이스입니다.
- **Eager Loading** 전략을 사용하여 컨테이너 기동 시 **싱글톤 빈을 미리 생성**합니다.
- **주요 부가 기능**
    - `MessageSource`: 국제화(i18n) 지원
    - `ApplicationEventPublisher`: 이벤트 발행 및 구독 모델 지원
    - `ResourceLoader`: 리소스(파일, URL 등)에 대한 통일된 접근 방식 제공
    - AOP(Aspect-Oriented Programming)와의 긴밀한 통합


### Configuration Metadata

Spring 컨테이너는 사용자가 제공하는 Configuration Metadata (설정 메타데이터)를 참고하여 [Spring Bean](Spring%20Bean.md) 관리한다.  해당 메타 데이터는 컨테이너에게 "애플리케이션의 객체를 어떻게 인스턴스화 하고, 구성하고, 조립할 것인가"를 지시한다.

Spring에서는 세 가지 주요 설정 방식을 지원한다.

#### XML-based configuration

`<bean>` 태그를 직접 사용하는 방식이다.  
`application-config.xml`을 `resource` 아래에 만들어 직접 관리한다.

```xml
<bean id="jwtUtil" class="org.fitsync.util.JwtUtil" />
<bean id="memberService" class="org.fitsync.service.MemberServiceImple"/>
<bean id="reportService" class="org.fitsync.service.ReportServiceImple"/>
<bean id="authTokenFilter" class="org.fitsync.filter.AuthTokenFilter">
```

이전 프로젝트에서 사용했던 레거시 방식에 대한 설명은 [해당 글](RESOURCE/SPRING/Spring%20MVC%20Legacy에서%20Bean을%20관리하는%20방법.md)을 참고

#### Java-based configuration

해당 방식은 Bean 의존관계를 수동으로 직접 등록하고자 할때 사용하는 방식이다.

- 클래스 생성 후 `@Configuration` 명시
- 각 메서드에 `@Bean` 명시

```java
@Configuration
public class AppConfig {

    @Bean
    public Item item1() {
        return new ItemImpl1();
    }

    @Bean
    public Store store() {
        return new Store(item1());
    }
}
```

#### Annotation-based configuration

이미 스프링에서 제공되는 어노테이션을 통해 자동으로 의존관계를 등록하는 방식이다.

- `@Controller` : 해당 클래스가 Presentation Layer 에서 컨트롤러임을 명시함.
- `@Service` : 해당 클래스가 Application Layer 에서 Service 임을 명시함.
- `@Repository` : 해당 클래스가 Persistence Layer 에서 DAO 임을 명시한다.
- `@Component` : 위 3가지 경우 이외에 Bean 으로 등록하고 싶은 클래스에 명시한다.

위 어노테이션을 붙이면 스프링이 `Component Scan`을 진행하여 각 객체들간의 의존관계를 읽는다.  

##### 내부 로직

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Component  
public @interface Service { }
```

해당 인터페이스 내부에는 `@Component`를 포함하고 있다.

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),  
       @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })  
public @interface SpringBootApplication { }
```

이러한 컴포넌트들은 `@ComponentScan`에 의해 스캔된다.

