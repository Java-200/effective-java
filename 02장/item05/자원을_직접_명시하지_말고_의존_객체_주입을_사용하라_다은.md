## **자원을 직접 명시**

- dictionary를 선언하면서 초기화하는, KorDictionary 클래스의 인스턴스임을 직접 명시하는 것

```java
public class SpellChecker {
    private static Dictionary dictionary = new KorDictionary();
}
```
<br>

- 싱글턴과 유틸리티 클래스가 그런 경우
<br>

### **유틸리티 - 철자 검사기**

```java
public class SpellChecker {
    private static final Dictionary dictionary = new KorDictionary();
    private SpellChecker() {}
    public static boolean isValid(String word) {
        return dictionary.contains(word);
    }
    public static List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```
<br>

### **싱글톤 - 철자 검사기**

```java
public class SpellChecker {
    private final Dictionary dictionary = new KorDictionary();
    private SpellChecker() {}
    public static INSTANCE = new SpellChecker(...);
    public boolean isValid(String word) {
        return dictionary.contains(word);
    }
    public List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```
<br>

### **문제점**

- KorDictionary를 사용했지만 다른 언어의 사전이 필요하다면?  
    ⇒ 필요한 만큼 필드를 추가해야 함
    
- setDictionary() 같은 메서드로 변경하게 만든다면?
    ⇒ 멀티스레드 환경에서 사용할 수 없음
    
- 즉, 사용하는 자원에 따라 동작이 달라지는 경우 적합하지 않음
<br>

## 의존 주입 방식

- SpellChecker를 보면 인스턴스 생성 시 외부에서 필요한 자원을 넘겨주는 방식

```java
public class SpellChecker {
    private final Dictionary dictionary;
    public SpellChecker(Dictionary dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word) { 
        return dictionary.contains(word);
    }
    public List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```
<br>

### 생성자에 자원 팩토리 넘겨주기

- 팩토리: 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
- Supplier를 보면 와일드카드 타입으로 팩토리의 타입 매개변수를 제한함

```java
public class SpellChecker {
    private final Dictionary dictionary;
    public SpellChecker(Supplier<? extends Dictionary> dictionarySupplier) {
        this.dictionary = dictionarySupplier.get();
    }
    public boolean isValid(String word) {
        return dictionary.contains(word);
    }
    public List<String> suggestions(String typo) {
        return dictionary.closeWordsTo(typo);
    }
}
```
<br>

- 명시한 타입의 하위 타입이라면 원하는 팩토리를 넘길 수 있음

```java
SpellChecker korSpellChecker = new SpellChecker(KorDictionary::new);
SpellChecker engSpellChecker = new SpellChecker(EngDictionary::new);
```
<br>

- 의존성을 주입할 객체가 필요하고, 의존성이 많아지면 코드가 복잡해짐
- 그래서 Spring(@Autowired)과 같은 의존 주입 프레임워크를 사용하는 걸 추천

⇒ 클래스가 내부적으로 하나 이상의 자원에 의존하고 클래스 동작에 영향을 준다면? 싱글턴과 유틸리티 클래스가 아닌 의존 객체 주입을 사용
