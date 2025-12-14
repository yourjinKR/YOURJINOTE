# try-with-resource

### 기존 방식
사용 후 반납해주어야 하는 리소스들은 `Closable` 인터페이스를 구현하고 있으며 대표적으로 `null` 체킹 과정 후 직접 호출하는 과정을 거쳤다.

```java
public static void main(String args[]) throws IOException {
    FileInputStream is = null;
    BufferedInputStream bis = null;
    try {
        is = new FileInputStream("file.txt");
        bis = new BufferedInputStream(is);
        int data = -1;
        while((data = bis.read()) != -1){
            System.out.print((char) data);
        }
    } finally {
        // close resources
        if (is != null) is.close();
        if (bis != null) bis.close();
    }
}
```

- 자원 반납에 의해 코드가 복잡해짐
- 작업이 번거로움
- 실수로 자원을 반납하지 못하는 경우 발생
- 에러로 자원을 반납하지 못하는 경우 발생
- 에러 스택 트레이스가 누락되어 디버깅이 어려움

### try-with-resource

이를 해결하고자 `AutoClosable` 인터페이스를 구현하고 있는 자원에 대해 `try-with-resource` 문법을 추가하여 리소스 반납에 대한 코드를 보다 간결하게 작성할 수 있다.

> `AutoClosable`은 뒤늦게 추가된 인터페이스이지만 `Closable`의 자식 인터페이스가 아닌 부모 인터페이스이다.  

```java
public static void main(String args[]) throws IOException {
    try (FileInputStream is = new FileInputStream("file.txt"); BufferedInputStream bis = new BufferedInputStream(is)) {
        int data;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    }
}

```

- 코드를 간결하게 만들 수 있음
- 번거로운 자원 반납 작업을 하지 않아도 됨
- 실수로 자원을 반납하지 못하는 경우를 방지할 수 있음
- 에러로 자원을 반납하지 못하는 경우를 방지할 수 있음
- 모든 에러에 대한 스택 트레이스를 남길 수 있음


더 자세한 비교 예시 코드는 아래 블로그를 참고

# 출처
[티스토리 - 망나니 개발자](https://mangkyu.tistory.com/217)
