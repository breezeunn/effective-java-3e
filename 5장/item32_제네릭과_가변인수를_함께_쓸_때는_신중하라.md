item32. 제네릭과 가변인수를 함께 쓸 때는 신중하라   

## 핵심 정리   
- 제네릭과 가변인수를 함께 쓸 때는 신중해야 한다.   
- 제네릭 varargs 매개변수는 타입 안전하지는 않지만, 허용된다.
- 메서드에 제네릭 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는 데 불편함이 없게끔 하자.


### 가변 인수
가변인수(varargs) 기능을 통해 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있다.
```java
static void dangerous(List<String>... stringLists)
```


### 힙 오염(heap pollution)에 대한 컴파일러의 경고
가변인수 메서드 호출 시 가변인수를 담기 위한 배열이 자동으로 만들어지는데, 이 배열이 클라이언트에 노출되어 매개변수화 타입의 변수가 타입이 다른 객체를 참조하게 된다면 힙 오염이 발생할 수 있다.
```java
Type safety: Potential heap pollution via varargs parameter stringLists
```

### 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.
이렇게 타입이 다른 객체를 참조하는 상황이 있다면 힙오염 뿐만 아니라 컴파일러가 자동 생성한 형변환이 실패할 수 있어, 제네릭 타입 시스템이 약속한 타입 안전성의 근간이 흔들려버린다. 이렇게 타입 안정성이 깨지는 경우가 발생하여 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

``` java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42); 
    Object[] objects = stringLists; 
    objects[0] = intList;               // 힙 오염 발생
    String s = stringLists[0].get(0);   // ClassCastException
}
```

``` java
Exception in thread "main" java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String (java.lang.Integer and java.lag (java.lang.Integer and java.lang.String are in module java.base of loader 'bootstrap')
        at Test.dangerous(Test.java:8)
```

#### 제네릭 배열을 프로그래머가 직접 생성하는 건 허용하지 않으면서 제네릭 varargs 매개변수를 받는 메서드를 선언할 수 있게 한 이유   
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 유용하기 때문에, 언어 설계자는 모순을 수용하기로 했다.
```java
List<String>[] errorLists = new List<String>[1]; // 오류!
static void dangerous(List<String>... stringLists) {    // 가능
    // TODO
}
```
- Arrays.asList, Collections.addAll, EnumSet.of 등 자바 라이브러리도 이런 메서드를 여럿 제공한다.

### @SafeVarargs 애너테이션: 메서드가 타입 안전함을 보장하는 장치
- 자바 7에서 추가되었다.
- 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있다.
- 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다.

### 안전한 메서드
1) 메서드가 이 배열에 아무것도 저장하지 않고(그 매개변수들을 덮어쓰지 않고)
2) 배열의 참조가 밖으로 노출되지 않는다면 (신뢰할 수 없는 코드가 배열에 접근할 수 없다면)
> 즉, varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.


### varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다
- 아래의 toArray는 가변인수로 넘어온 매개변수들을 배열에 담아 반환하는 제네릭 메서드이다.(별다른 경고 없이 컴파일되므로 위험한 메서드)
- toArray 가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다. 
   > 예를 들면, 아래 코드에서 만드는 배열의 타입은 Object[] 인데, Object[] 가 String[] attributes = 에 저장되는 과정에서 String[] 으로 형변환을 시도하는 중 실패하여 오류가 발생하게 된다.
- 예외
  1) @SafeVarargs 로 애노테이트 된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
  2) 그저 이 배열 내용의 일부 함수를 호출만 하는 (varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

```java
static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
            /*
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
            */
        }
        throw new AssertionError();
    }

    static <T> T[] toArray(T... args) {
        return args;    // 참조가 노출된다
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");
    }
```

```java
Exception in thread "main" java.lang.ClassCastException: class [Ljava.lang.Object; cannot be cast to class [Ljava.lang.String; ([Ljava.lang.Object; and [Ljava.lang.String; are in module java.base of loader 'bootstrap')
        at Sample322.main(Sample322.java:18)
```

### 제네릭 varargs 매개변수를 안전하게 사용하는 예   
임의 개수의 리스트를 인수로 받아, 받은 순서대로 그 안의 모든 원소를 하나의 리스트로 옮겨 담아 반환한다.
```java
static <T> List<T> flatten(List<List<? extends T>>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}

```