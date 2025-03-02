## try-finally보다는 try-with-resource를 사용하라

### 1. 자원 닫기
자바 라이브러리에는 `close 메서드`를 사용하여 직접 닫아줘야하는 자원이 많다.   
예) InputStream, OutputStream, java.sql.Connection 등 ... 

하지만, 자원 닫기는 개발자가 놓치기 쉬워 **예측할 수 없는 성능 문제**로 이어질 수 있다.

이를 막기 위해 `finalizer`를 사용하기도 하지만, 그리 믿음직스럽진 못하다.

<br/>

### 2. try-finally

자원을 안전하게 닫기 위해 전통적으로 **try-finally** 구문이 사용되었다. 하지만 여러 개의 자원을 사용할 경우, 코드가 복잡해지고 디버깅이 어려워질 수 있다.   

**자원이 하나일 때: try-finally 구문**
```java
static String firstLineOfFile(String path) {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

위 코드는 `BufferedReader`를 생성한 후, `try-finally` 구문을 사용해 파일을 읽고 닫는 방식이다. `finally` 블록에서 `close() 메서드`를 사용하기 때문에 예외가 발생하더라도 자원이 안전하게 해제된다. 


<br/>

**자원이 두 개일떄: try-finally 구문**
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOuputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
위 코드에서는 `InputStream`과 `OutputStream` 두 개의 자원을 사용한다. 각각의 자원을 안전하게 닫기 위해 `try-finally` 블록을 중첩해서 사용하고 있다. 

그러나 이러한 방식에는 두 가지 단점이 존재한다. 

1. **코드의 복잡성 증가**: 자원이 많아질수록 `try-finally` 블록이 중첩되어 코드의 가독성이 떨어진다. 예를 들어, 자원이 10개라면 `try-fianlly` 블록이 10번 중첩되어 코드가 매우 복잡해질 것이다. 

2. **디버깅 어려움** : 예외가 발생했을 때, 어떤 자원에서 문제가 발생했는지 파악하기 어려울 수 있다.   

`firstLineOfFile` 메서드를 살펴보자. 만약 기기에서 물리적인 문제가 발생하면, `br.readLine()`에서 첫 번째 예외가 발생하고, 이후 `br.close()`에서도 예외가 발생할 수 있다. 하지만 스택 추적 내역에서는 두 번째 예외만 기록되므로, 첫 번째 예외에 대한 정보는 남지 않아 디버깅이 매우 어려워질 수 있다.  

<br/>

### 3. try-with-resources
위에서 언급한 문제를 해결하기 위해 Java 7부터 **try-with-resources** 문법이 도입되었다. 이 문법을 사용하면 `AutoCloseable`을 구현한 객체를 `try` 블록에서 선언하고, `finallly` 없이 자동으로 자원이 해제되도록 할 수 있다. 

`AutoCloseable`을 구현한 객체는 다음과 같다.
- BufferedReader, FileReader, FileWriter, InputStream, OutputStream, ... 

<br/>

**자원이 하나일 때: try-with-resources구문**
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
            new FileReader(path))){
        return br.readLine();
    }
}
```
위 코드는 `BufferedReader` 객체를 `try` 블록에서 선엄함으로써, `finally` 없이도 `close()` 가 자동으로 호출된다. 

<br/>

**자원이 두 개일떄: try-with-resources 구문**
```java
static void copy(String src, String dst) throws IOException { 
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOuputStream(dst)){
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0)
                 out.write(buf, 0, n);
    }
}
```
여러 개의 자원을 사용할 경우 `try` 괄호 안에 `;` 를 이용해 여러 개의 자원을 선언할 수 있으며, 모든 자원이 자동으로 해제된다. 

<br/>

**try-with-resources 구문의 장점**

1. **코드의 가독성 증가**: `try-finally` 와 달리 중첩된 블록이 없어 코드가 간결해진다.
2. **디버깅 용이**: 예외 발생 시, `close()`에서 발생한 예외는 숨겨지고, 원래 발생한 예외가 기록되어 확인할 수 있다. 

`firstLineOfFile` 메서드에서 `br.readLine()`과 `close` 에서 모두 예외가 발생한다면, close에서 발생한 예외는 숨겨지고 **br.readLine()에서 발생한 예외가 기록된다.** 즉, 개발자에게 보여줄 예외 하나만 보존되고 다른 예외는 숨겨진다. 

또한, 숨겨진 예외도 스택 추적 내역에 `suppressed` 라는 꼬리표를 달고 출력되며, Java 7에서 `getSuppressed` 메서드를 이용하면 프로그램 코드에서 가져올 수 있다. 

<br/>

**try-with-resources에서의 catch절 사용**  

catch절을 사용하여 try문을 중첩하지 않고도 다수의 예외를 처리할 수 있다. 

```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(
            new FileReader(path))) {
                return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```
파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환하는 코드이다. 

<br/>


## ✨ 핵심 정리 
> 꼭 회수해야하는 자원을 다룰 때는 **try-with-resources**를 사용하자.   
> 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨~씬 유용하다. 👍  
> try-finally로 작성하면 코드가 지저분해지는 경우라도,  
**try-with-resources**로는 정확하고 쉽게 자원을 회수할 수 있다! 