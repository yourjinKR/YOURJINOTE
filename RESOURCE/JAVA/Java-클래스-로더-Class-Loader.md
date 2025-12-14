# 클래스 로더란?

클래스 로더는 런타임 중에 동적으로 Java 클래스를 JVM의 메소드 영역에 로드하는 역할을 수행한다.  
클래스 로더에는 **로딩, 링크, 초기화** 단계로 나뉘어져 있다.

## 로딩

자바 바이트 코드를 메서드 영역에 저장
각 자바 바이트 코드는 JVM에 의해 메서드 영역에 다음 정보들을 저장한다.

- 로드된 클래스를 비롯한 그의 부모 클래스 정보
- 클래스 파일이나 Class, Interface, Enum의 관련 여부
- 변수나 메소드 등의 정보

## 링크
링크는 아래와 같이 세가지 단계를 거친다.
### 검증 (Vertification)
읽어 들인 클래스가 자바 언어 명세 및 JVM 명세에 명시된 대로 잘 구성되어 있는지 검사한다.
### 준비 (Pereparation)
클래스가 필요로 하는 메모리를 할당하고, 클래스에서 정의된 필드, 메소드, 인터페이스를 나타내는 데이터 구조를 준비한다. 
### 분석 (Resolution)
심볼릭 메모리 레퍼런스를 메소드 영역에 있는 실제 레퍼런스로 교체한다.


> **심볼릭 레퍼런스란?**  
> 프로그램 내에서 클래스, 메서드, 필드 등을 직접적으로 식별하는 데  사용되는 이름이나 식별자

## 초기화
정적 변수에 실제 값을 할당하고 클래스에 정의된 정적 블록을 실행한다.

# 클래스 로더의 종류

- Bootstrap Class Loader (Primitive Class Loader)
- Platform Class Loader (Extension Class Loader)
- System Class Loader (Application Class Loader)

![Pasted image 20251127174301](../../GALLERY/Pasted%20image%2020251127174301.png)
## 부트 스트랩 클래스 로더 (Bootstrap Class Loader)

- JVM 시작 시 **가장 최초로 실행되는 클래스 로더**이다.  
- 부트스트랩 클래스 로더는 자바 클래스를 로드하는 것이 아닌, 자바 클래스를 로드할 수 있는 자바 자체의 클래스 로더와 **최소한의 자바 클래스만을  로드**한다.

> java.lang.Object, Class, ClassLoader만을 로드합니다.

- 부트 클래스 로더는 상위 클래스 로더 없이 **독립적**으로 작동한다.

> Java 8 이전 버전에서는 rt.jar에서 핵심 Java 파일을 로드했습니다.  
> 하지만 Java 9부터는 Java 런타임 이미지(JRT)에서 핵심 Java 파일을 로드합니다.

## 플랫폼 클래스 로더 (Platform Class Loader)

- JDK의 모듈 시스템에서 플랫폼 별 확장 자바 클래스들을 불러온다.
- 부트 스트랩 클래스 로더를 부모로 갖는다.

> Java 8 이전 버전에서는 URLClassLoader를 상속하며, `jre/lib/ext` 내 모든 클래스를 로드한다.  
> 이후로는 Platform Loader로 변경되었으며, URLClassLoader가 아닌 BuiltinClassLoader를 상속한다. Inner Static 클래스로 구현되어 있다.

## 시스템 클래스 로더 (System Class Loader)

- 애플리케이션 클래스 로더라고도 하며, 애플리케이션의 클래스 경로에서 클래스를 로드한다.
- 자바 프로그램 실행 시 지정한 Classpath에 있는 클래스 파일 혹은 jar에 속한 클래스들을 로드한다. 
- 플랫폼 클래스 로더를 부모로 갖는다.

# 클래스 로더의 동작 방식

![[Pasted image 20251214111251.png]]

다음과 같은 방식으로 로드를 수행한다.

1. JVM의 메소드 영역에 클래스가 로드되어 있는지 확인한다. 만일 로드되어 있는 경우 해당 클래스를 사용한다.
2. 메소드 영역에 클래스가 로드되어 있지 않을 경우, 시스템 클래스 로더에 클래스 로드를 요청한다.
3. 시스템 클래스 로더는 플랫폼 클래스 로더에 요청을 위임한다.
4. 플랫폼 클래스 로더는 부트스트랩 클래스 로더에 요청을 위임한다.
5. 부트스트랩 클래스 로더는 부트스트랩 Classpath(JDK/JRE/LIB)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 플랫폼 클래스 로더에게 요청을 넘긴다.
6. 플랫폼 클래스 로더는 확장 Classpath(JDK/JRE/LIB/EXT)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않을 경우 시스템 클래스 로더에게 요청을 넘긴다.
7. 시스템 클래스 로더는 시스템 Classpath에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 `ClassNotFoundException`을 발생시킨다.

> 하위 클래스부터 요청을 시작으로 해당 요청을 상위 클래스 로더에게 위임한다.  
> 즉, 실질적인 실행 순서는 상위 클래스 로더부터 실행된다.

# 클래스 로더의 기능적 원칙

### 위임 원칙

- 클래스 로더는 클래스 또는 리소스를 찾기 위해 요청을 받았을 때, 상위 클래스 로더에게 책임을 위임하는 위임 모델을 따른다.
- 위에서 설명한 클래스 로더 동작 방식을 보면, 위임 법칙을 따른다는 것을 확인할 수 있다.

### 가시 범위 원칙

- 하위 클래스 로더는 상위 클래스 로더가 로드한 클래스*를 볼 수 있지만, 반대로 상위 클래스 로더는 하위 클래스 로더가 로드한 클래스를 알 수 없다.
- 이로 인해 java.lang.Object 클래스 등 상위 클래스 로더에서 로드한 클래스도 하위 클래스 로더인 시스템 클래스 로더 등에서 사용할 수 있다.

> 이를 통해 캡슐화를 보장하고 서로 다른 ClassLoader에 의해 로드된 클래스 간의 충돌을 방지한다.

### 유일성의 원칙

- 클래스 로더는 클래스가 한번만 로드되도록 하여 고유성을 유지한다.
- 상위 클래스 로더가 클래스를 찾을 수 없는 경우에만 현재 인스턴스가 해당 클래스를 로드하려고 시도한다.

## `java.lang.ClassLoader`의 메서드

메서스에 대한 정보는[여기](https://www.geeksforgeeks.org/java/classloader-in-java/)를 참고

# 동적 클래스 로딩
자바의 클래스 로딩은 클래스 참조 시점에 JVM에 코드가 링크되고, 실제 런타임 시점에 로딩되는 동적 로딩을 거친다. 런타임에 동적으로 클래스를 로딩한다는 것은 **JVM이 미리 모든 클래스에 대한 정보를 메소드 영역에 로딩하지 않는다**는 것을 의미한다.

동적으로 클래스를 로딩하는 방식은 두 가지가 있다.
## 로드 타임 동적 로딩 (Load-time Dynamic Loading)
하나의 클래스를 로딩하는 과정에서 동적으로 다른 클래스를 로딩하는 것을 로드 타임 동적 로딩이라고 한다.
### 예시 코드
```java
public class HelloWorld { 
	public static void main(String[] args) { 
		System.out.println("안녕하세요!"); 
	} 
}
```
위 코드의 경우 다음*과 같이 동작한다.

1. JVM이 시작되고 부트스트랩 클래스 로더가 생성된 후에 모든 클래스가 상속받고 있는 Object 클래스를 읽어온다.
2. 클래스 로더는 명령 행에서 지정한 HelloWorld 클래스를 로딩하기 위해, HelloWorld.class 파일을 읽는다.
3. HelloWorld 클래스를 로딩하는 과정에서 필요한 클래스인 java.lang.String과 java.lang.System을 로딩한다.
*
## 런타임 동적 로딩 (Run-time Dynamic Loading)
클래스를 로딩할 때가 아닌, 코드를 실행하는 순간에 클래스를 로딩하는 것을 런타임 동적 로딩이라고 한다.
### 예시 코드
```java
public class RuntimeLoading { 
	public static void main(String[] args) { 
		try { 
			Class cls = Class.forName(args[0]); 
			Object obj = cls.newInstance(); 
			Runnable r = (Runnable) obj; 
			r.run(); 
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
위 코드에서 `Class.forName(className)` 은 파라미터로 받은 className에 해당하는 클래스를 로딩한 후에 객체를 반환하며, 다음과 같이 동작한다.

1. `Class.forName()` 메소드가 실행되기 전까지는 RuntimeLoading 클래스에서 어떤 클래스를 참조하는지 알 수 없다.
2. 따라서 RuntimeLoading 클래스를 로딩할 때는 어떤 클래스도 읽어오지 않고, RuntimeLoading 클래스의 `main()` 메소드가 실행되고 `Class.forName(args[0])` 를 호출하는 순간에 비로소 `args[0]` 에 해당하는 클래스를 로딩한다.



# 출처
[geeksforgeeks](https://www.geeksforgeeks.org/java/classloader-in-java/)  
[f-lab](https://f-lab.kr/insight/understanding-jvm-class-loader-20250529?gad_source=1&gad_campaignid=22368870602&gbraid=0AAAAACGgUFdfKzkzzvnlSSkanha9oUlBE&gclid=CjwKCAiA86_JBhAIEiwA4i9JuynLqNNXbnPWq0WQA474xiQxdWCQzbx4TDzy44KdlxbtO742gca9RhoCyYgQAvD_BwE)  
[티스토리-JVM의-클래스-로더란?](https://steady-coding.tistory.com/593#google_vignette)  

> 클래스의 로딩, 초기화 시점에 대한 이야기와 예상 면접 질문 및 답변은 링크를 참고