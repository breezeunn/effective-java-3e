item49.매개변수가 유효한지 검사하라

### 핵심 요약
1. 메서드나 생성자를 작성할 때 그 매개변수들에 어떤 제약이 있을지 생각해야 한다.
2. 제약을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야 한다.


#### 매개변수 검사를 제대로 하지 못하면 생길 수 있는 문제
1. 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.   
2. 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.   
3. 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와는 관련 없는 오류를 낼 수 있다.


#### 매개변수 제약 문서화
1. public 이나 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화 해야 한다. (@throws 자바독 태그 사용, item74 에서 상세히 다룰 예정)
2. 제약을 어겼을 때 발생하는 예외도 함께 기술한다.

``` java
/**
  * (현재 값 mod m) 값을 반환한다.
  * 이 메서드는 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
  * 
  * @param m 계수 (양수여야 한다.)
  * @return 현재 값 mod m
  * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
  */

  public BigInteger mod(BigInteger m) {
      if (m.signum() <= 0) 
          throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
      // 계산 수행 ..
  }

```

#### Objects.requireNonNull
자바 7에 추가된 java.util.Objects.requireNonNull 메서드를 통해 더 이상 null 검사를 수동으로 하지 않아도 된다.
``` java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}

public static <T> T requireNonNull(T obj, String message) {
    if (obj == null)
        throw new NullPointerException(message);
    return obj;
}
```

``` java
public static void main(String[] args) {
    Objects.requireNonNull(null, "전략");
}

Exception in thread "main" java.lang.NullPointerException: 전략        
        at java.base/java.util.Objects.requireNonNull(Objects.java:246)
        at Main.main(Main.java:5)
```

#### assert(단언문) 을 통한 매개변수 유효성 검증
- 공개되지 않은 메서드, 즉 public 이 아닌 메서드인 경우 메서드가 호출되는 상황을 통제하기 위해 사용할 수 있다.
- 실패 시 AssertionError 가 발생되며, 런타임에 아무런 효과나 성능 저하도 없다.  (단, 명령줄에서 -ea 혹은 00enableassertions 플래그 설정 시 런타임에 영향을 준다.)
- VS Code 에서 설정 시 launch.json 에 아래와 같이 추가
``` java
{
    "type": "java",
    "name": "Launch Main",
    // (...)
    "vmArgs": "-enableassertions"
},
```

```java
public static void main(String[] args) {
    //printMessage("hello world"); // OK
    printMessage(null);
}

private static void printMessage(String message) {
    assert message != null;
    // TODO 계산 수행
}

Exception in thread "main" java.lang.AssertionError
        at Main.printMessage(Main.java:8)
        at Main.main(Main.java:4)
```    

#### 메서드 실행 전 매개변수 유효성 검사 규칙의 예외
1. 유효성 검사 비용이 지나치게 높거나 실용적이지 않을때   
2. 혹은 계산 과정에서 암묵적으로 검사가 수행될 때
   > Collections.sort(List) 처럼 객체 리스트를 정렬하는 메서드가 있다고 가정하면, 리스트 안의 객체들은 모두 상호비교 될 수 있어야 한다. 상호 비교가 이루어질 수 없는 타입의 객체가 들어 있다면 그 객체와 비교할 때 ClassCastException 을 던질 것이다. 따라서 비교하기 아서 모든 객체가 상호 비교될 수 있는지 검사해봐야 별다른 실익이 없다.