# item18. 상속보다는 컴포지션을 사용하라

## 핵심 정리
- 상속은 강력하지만 캡슐화를 해친다는 문제가 있다
- 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자.

### 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다   
상위 클래스는 릴리즈마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다.   

### 상속은 상위 클래스와 하위 클래스 관계가 명확한 경우에도 문제가 될 수 있다
1) 하위 클래스의 패키지가 상위 클래스의 패키지와 다르거나
2) 상위 클래스가 확장을 고려해 설계되지 않았다면 문제될 수 있다.   
- 아래의 코드는 3을 출력하지 않고 6을 출력한다.
- 상위 클래스에 구현된 add() 는 상위 클래스에 구현된 add 가 아닌 InstrumentedHashSet 에서 재정의된 메서드이기 때문이다.

```java
public class InstrumentedHashSet<E> extends HashSet<E>{
    private int addCount = 0;

    public InstrumentedHashSet() {}

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

```java
public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```

### 메서드를 재정의하지 않아도 안전하지 않다
- 다음 릴리즈에서 상위 클래스에 새 메서드가 추가됐는데, 하필 내가 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입은 다르다면?
   > 컴파일이 되지 않는다.


### 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자. (컴포지션)
- 전달(Forwarding): 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다.   


### 상속을 사용하기로 결정했다면 자문해야 할 질문
- 확장하려는 클래스의 API 에 아무런 결함이 없는지?
- 결함이 있다면 내 클래스의 API 까지 전파되어도 괜찮은지?