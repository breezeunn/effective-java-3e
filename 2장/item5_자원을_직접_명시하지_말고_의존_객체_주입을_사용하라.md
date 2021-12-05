# 핵심 정리
* **필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자.**    
  - 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 
  - 이 자원들을 클래스가 직접 만들게 해서도 안 된다.    
* 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

## 잘못된 코드 1
* dictionary 가 final 로 정의되어 있어 유연하지 않고 테스트하기 어렵다.
``` java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // 객체 생성 방지
    
    public static boolean isValid(String world) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

## 잘못된 코드 2
* 싱글턴으로 구현되어 유연하지 않고 테스트하기 어렵다.
``` java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

## 잘못된 코드 1,2 수정
* 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주자   
   > SpellChecker 가 여러 사전을 활용할 수 있다

``` java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```