item32. 제네릭과 가변인수를 함께 쓸 때는 신중하라   

## 핵심 정리   
- 제네릭과 가변인수를 함께 쓸 때는 신중해야 한다.   
   가변인수(varargs) 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 달라 예상하지 못한 컴파일 오류가 발생할 수 있다.
- 제네릭 varargs 매개변수는 타입 안전하지는 않지만, 허용된다.
- 메서드에 제네릭 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는 데 불편함이 없게끔 하자.

### 힙 오염(heap pollution)에 대한 컴파일러의 경고
가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어지는데, 이 배열이 노출되어 매개변수화 타입의 변수가 타입이 다른 객체를 참조하게 된다면 힙 오염이 발생할 수 있다.
```java
Type safety: Potential heap pollution via varargs parameter stringLists
```

### 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.
이렇게 타입이 다른 객체를 참조하는 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있어, 제네릭 타입 시스템이 약속한 타입 안전성의 근간이 흔들려버린다.

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

### @SafeVarargs 애너테이션: 메서드가 타입 안전함을 보장하는 장치
- 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있다.
- 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다.

### 안전한 메서드
1) 메서드가 이 배열에 아무것도 저장하지 않고(그 매개변수들을 덮어쓰지 않고)
2) 배열의 참조가 밖으로 노출되지 않는다면 (신뢰할 수 없는 코드가 배열에 접근할 수 없다면)
> 즉, varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.


### varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다
toArray는 Object[] 타입을 리턴하여, pickTwo() 도 Object[] 결과를 리턴하게 된다. 이후 Object[] 가 String[] attributes = 에 저장되는 과정에서 String[] 으로 형변환을 시도하는 중 실패하여 오류가 발생하게 된다.

```java
static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError();
    }

    static <T> T[] toArray(T... args) {
        return args;
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
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}


static <T> List<T> flatten(List<List<? extends T>>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}

```

혹은, 정적 팩터리 메서드인 List.of()를 활용할 수도 있다. 