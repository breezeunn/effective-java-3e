item88. readObject 메서드는 방어적으로 작성하라


### item50 에서 불변인 날짜 범위 클래스를 만드는 데 가변인 Date 필드를 이용하여 Date 객체를 방어적으로 복사했던 코드
```java
public class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(start + "가 " + end + " 보다 늦다.");
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }

    public String toString() { return start + " - " + end; }
}
```
이 클래스를 직렬화하기로 결정했다고 가정하자.  
Period 객체의 물리적 표현이 논리적 표현과 부합하므로 기본 직렬화 형태를 사용해도 나쁘지 않아서, implements Serializable 을 추가한다면?  
이 클래스의 주요한 불변식을 더는 보장하지 못하게 된다.

---

> readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다.  

readObject 로 바이트 스트림을 받는데, 이 바이트 스트림이 정상적인 객체의 바이스 스트림이라고 기대하지만 실제 불변식을 깨뜨릴 의도로 임의 생성되었을수도 있다.  
``` java
class BogusPeriod {
    private static final byte[] serializedForm =  {
        // byte stream....
    };

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }

    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```
---
결국, readObject 에서도 인수가 유효한지 검사해야 하고, 필요하다면 매개변수를 방어적으로 복사해야 한다.  
이 문제를 고치려면 Period 의 readObject 메서드가 defaultReadObject 를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다.   
이 유효성 검사에 실패하면 InvalidObjectException 을 던지게 하여 잘못된 역직렬화가 일어나는 것을 막을 수 있다.
``` java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    if (start.compareTo(end) > 0) 
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```
--- 
그러나, 아직 문제가 숨어있다.  
정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다.  
공격자는 ObjectInputStreamPeriod 에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 이 '악의적인 객체 참조'를 읽어 Period 객체의 내부 정보를 얻을 수 있다.  이때 참조로 얻은 Date 인스턴스들을 수정할 수 있게 되어, Period 인스턴스는 불변이 아닌 상태가 된다.  
```java
class MutablePeriod {
    public final Period period;
    public final Date start;
    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            out.writeObject(new Period(new Date(), new Date()));

            byte[] ref = { 0x71, 0, 0x7e, 0, 5 };
            bos.write(ref);
            ref[4] = 4;
            bos.write(ref);

            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }

    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period p = mp.period;
        Date pEnd = mp.end;

        pEnd.setYear(78);
        System.out.println(p);

        pEnd.setYear(69);
        System.out.println(p);
    }
}
```
결과
<pre>
Wed Feb 23 14:49:16 KST 2022 - Thu Feb 23 14:49:16 KST 1978
Wed Feb 23 14:49:16 KST 2022 - Sun Feb 23 14:49:16 KST 1969
</pre>
--- 
이 문제의 근원은 Period 의 readObject 메서드가 방어적 복사를 충분히 하지 않은 데 있다.   
객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.   
따라서, readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.
한편, final 필드는 방어적 복사가 불가능하니 주의해야 한다.
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException{
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0) 
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```
--- 
### 기본 readObject 메서드를 써도 좋을지 판단하는 방법
transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?  
에 대한 질문이 "아니오" 라면 커스텀 readObject 메서드를 만들어 생성자에서 수행될 모든 유효성 검사와 방어적 복사를 수행해야 한다.