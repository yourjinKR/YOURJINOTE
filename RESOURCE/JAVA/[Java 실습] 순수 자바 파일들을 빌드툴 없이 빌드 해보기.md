# 순수 Java 프로젝트 수동 빌드 실습 (빌드 툴 없이)

## 1. 개요

이 문서는 Maven이나 Gradle 같은 빌드 툴 없이, 순수하게 `javac`와 `java` 명령어만으로 Java 프로젝트를 빌드하고 실행하는 과정을 설명한다.  
모든 과정은 Bash 환경에서 수행된다.

## 2. 디렉터리 구조 생성

```bash
mkdir -p pure-java-build/src/app/util
mkdir -p pure-java-build/out
cd pure-java-build
```

최종 구조 예시:

```
pure-java-build/
 ├── src/
 │   └── app/
 │       ├── Main.java
 │       └── util/
 │           └── Printer.java
 ├── out/
 └── sources.txt
```

## 3. 소스 코드 작성

### src/app/util/Printer.java

```java
package app.util;

public class Printer {
    public static void print(String message) {
        System.out.println("[Printer] " + message);
    }
}
```

### src/app/Main.java

```java
package app;

import app.util.Printer;

public class Main {
    public static void main(String[] args) {
        Printer.print("Hello, Pure Java Build!");
    }
}
```

## 4. 소스 목록 파일 생성

```bash
find src -name "*.java" > sources.txt
```

`sources.txt` 내용 예시:

```
src/app/Main.java
src/app/util/Printer.java
```

## 5. 컴파일

```bash
javac -d out @sources.txt
```

설명:
- `-d out` : 컴파일 결과(.class 파일)를 out 디렉터리에 저장
- `@sources.txt` : 소스 파일 목록을 한 번에 읽어 컴파일
    

컴파일 결과 확인:
```bash
find out -type f
```

예시 출력:
```
out/app/Main.class
out/app/util/Printer.class
```

## 6. 실행

```bash
java -cp out app.Main
```

예상 출력:
```
[Printer] Hello, Pure Java Build!
```

## 7. 빌드 및 실행 스크립트 (선택 사항)
반복적인 명령 실행을 줄이기 위해 간단한 빌드 스크립트를 만들 수 있다.
### build.sh

```bash
#!/bin/bash
set -e

echo "[1] 소스 목록 생성 중..."
find src -name "*.java" > sources.txt

echo "[2] 컴파일 중..."
javac -d out @sources.txt

echo "[3] 실행 중..."
java -cp out app.Main
```

실행 권한 부여 및 실행:

```bash
chmod +x build.sh
./build.sh
```

## 8. 요약
| 단계  | 명령어                                     | 설명          |
| --- | --------------------------------------- | ----------- |
| 1   | `find src -name "*.java" > sources.txt` | 소스 파일 목록 생성 |
| 2   | `javac -d out @sources.txt`             | 소스 파일 컴파일   |
| 3   | `java -cp out app.Main`                 | 프로그램 실행     ||

## 9. 정리
- `javac`는 `.java` 파일을 `.class` 파일로 변환하는 컴파일러이다.
- `java` 명령은 지정한 클래스 경로(`-cp`)에서 main 메서드를 포함한 클래스를 실행한다.
- `@sources.txt`를 이용하면 여러 파일을 한 번에 컴파일할 수 있다.
- Maven이나 Gradle 같은 빌드 툴은 결국 이 과정을 자동화한 도구에 불과하다.
    


