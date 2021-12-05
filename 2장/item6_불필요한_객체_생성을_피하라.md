# 불필요한 객체 생성을 피하라

### 극단적인 예?
s1 은 실행될 때마다 String 인스턴스를 새로 만든다.   
반면, s2 는 하나의 String 인스턴스를 사용하며, 같은 가상 머신 안에서 동일한 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.
``` java
String s1 = new String("bikini");   // 따라하지 말 것
String s2 = "bikini";
```

### 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피하자
* 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다.   
* b1 의 생성자를 대신하여 b2의 팩터리 메서드를 사용하면 불필요한 객체 생성을 피할 수 있다. (b1 의 경우 deprecated 로 표시된다.) 
``` java
boolean b1 = new Boolean("true");
boolean b2 = Boolean.valueOf("true");
```

### 생성 비용이 '아주 비싼' 객체가 반복해서 필요하다면 캐싱하여 재사용하길 권한다.
String.matches 가 내부에서 만드는 정규표현식용 Pattern 인스턴스는, 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다.  
* Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 인스턴스 생성 비용이 높다.

#### 개선 전
```java
static boolean isRomanNumeral(String s) {
    return s.matches("...");
}
``` 

#### **개선 후: 비싼 객체(Pattern) 을 재사용해 성능을 개선**
정규표현식을 표현하는 (불변인) Pattern 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해두고, 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용한다.
```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("...");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

### 의도치 않은 오토박싱 사용으로 인한 성능 저하
* 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.
* 의미상으로는 별다를 것 없어 보이지만 성능에서는 그렇지 않다.
* 아래 코드는 sum 변수를 long 이 아닌 Long 으로 선언하여 불필요한 Long 인스턴스를 약 2^31 개 생성한 경우인데, sum 의 타입을 long 으로만 변경하여도 성능이 개선된다.
```java
private static long sum() {
    Long sum = 0l;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```


### 객체 생성은 무조건 피해야 할까?
* 요즘의 JVM 에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않는다.  
  (프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다.)
* 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 자체적인 객체 풀(pool) 을 만들지는 말자.   
  - 물론, 데이터베이스 연결 같은 경우 생성 비용이 비싸니 재사용하는 편이 낫다.  