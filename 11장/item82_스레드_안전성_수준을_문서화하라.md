item82.스레드 안전성 수준을 문서화하라

## 요약
1. 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화해야 한다.  
2. synchronized 한정자는 문서화와 관련이 없다.
3. 문서화 시 스레드 안전성 수준을 정확히 명시해야한다.
4. 조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다.  
5. 무조건적 스레드 안전 클래스를 작성할 때는 synchronized 메서드가 아닌 비공개 락 객체를 사용하자.  



### 1. 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화해야 한다.  
하나의 메서드를 여러 스레드가 동시에 호출할 때 API 문서에 아무런 언급도 없다면 그 클래스 사용자는 나름의 가정을 해야만 한다.   
> 만약 가정이 틀리면 클라이언트 프로그램은 동기화를 충분히 하지 못하거나 지나치게 한 상태가 되고, 두 경우 모두 심각한 오류로 이어질 수 있다.

### 2. synchronized 한정자는 문서화와 관련이 없다.
API 문서에 synchronized 한정자가 보이는 메서드는 스레드 안전하다는 말은 몇 가지 면에서 틀렸다.
1. 자바독이 기본 옵션에서 생성한 API 문서에는 synchronized 한정자가 포함되지 않는다.  
>> 메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다.  
>> 그렇다면 Java 클래스들은 스레드 안전성을 어떻게 문서화하고 있을까?  
   > * 예1) ArrayList: [클래스 설명 내 진하게 표시](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html)  
   > * 예2) SimpleDateFormat: [클래스 설명 내에 synchronization 항목에서 'not synchronized' 임을 표시](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html)  
   > * 예3) apache HttpGet: '자바 병렬 프로그래밍' 에서 제안된 [JCIP 어노테이션](https://jcip.net/annotations/doc/net/jcip/annotations/package-summary.html) 을 이용하여 [표시](https://www.javadoc.io/doc/org.apache.httpcomponents/httpclient/4.3.4/org/apache/http/client/methods/HttpGet.html)

2. 스레드 안전성에도 수준이 나뉘기 때문에 단순히 synchronized 유무로 스레드 안전성을 완벽히 파악할 수 없다.


### 3. 문서화 시 스레드 안전성 수준을 정확히 명시해야한다. 
1. 스레드 안전성 구분  
이 분류는 (스레드 적대적만 빼면) 『자바 병렬 프로그래밍』의 부록에 나오는 스레드 안전성 애너테이션(@Immutable, @ThreadSafe, @NotThreadSafe) 과 대략 일치한다.   
안전성이 높은 순으로 아래와 같이 나열할 수 있다.

> 1) 불변(immutable)  
>> * 이 클래스의 인스턴스는 상수와도 같아서 외부 동기화도 필요없다.
>> * @Immutable  
예) String, Long, BigInteger

>2. 무조건적 스레드 안전(unconditionally thread-safe)  
>> * 클래스의 인스턴스가 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다.
>> * @ThreadSafe  
예) AtomicLong, ConcurrentHashMap

>3. 조건부 스레드 안전(conditionally thread-safe)  
>> * 2번의 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다.  
>> * @ThreadSafe  
예) Collections.synchronized 래퍼 메서드가 반환한 컬렉션: 이 컬렉션들이 반환한 반복자는 외부에서 동기화해야 한다.

>4. 스레드 안전하지 않음(not thread-safe)  
>> * 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의(혹은 일련의) 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다.  
>> * @NotThreadSafe  
예) ArrayList, HashMap 같은 기본 컬렉션  

>5. 스레드 적대적(thread-hostile)  
>> * 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다. 
> * 일반적으로 문제를 고쳐 재배포하거나 사용 자제(deprecated) API로 지정한다.

### 4. 조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다.  
1. 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할 때 외부 동기화가 요구되고 그때 어떤 락을 얻어야 하는지도 알려줘야 한다.

2. 클래스의 스레드 안정성은 보통 클래스의 문서화 주석에 기재하지만, 독특한 특성의 메서드라면 해당 메서드의 주석에 기재하도록 하자. 
반환 타입만으로는 명확히 알 수 없는 정적 팩터리라면 자신이 반환하는 객체의 스레드 안전성을 반드시 문서화해야 한다.   
예) Collections.synchronizedMap
``` java
    /**
     * Returns a synchronized (thread-safe) map backed by the specified
     * map.  In order to guarantee serial access, it is critical that
     * <strong>all</strong> access to the backing map is accomplished
     * through the returned map.<p>
     *
     * It is imperative that the user manually synchronize on the returned
     * map when traversing any of its collection views via {@link Iterator},
     * {@link Spliterator} or {@link Stream}:
     * <pre>
     *  Map m = Collections.synchronizedMap(new HashMap());
     *      ...
     *  Set s = m.keySet();  // Needn't be in synchronized block
     *      ...
     *  synchronized (m) {  // Synchronizing on m, not s!
     *      Iterator i = s.iterator(); // Must be in synchronized block
     *      while (i.hasNext())
     *          foo(i.next());
     *  }
     * </pre>
     * Failure to follow this advice may result in non-deterministic behavior.
     *
     * <p>The returned map will be serializable if the specified map is
     * serializable.
     *
     * @param <K> the class of the map keys
     * @param <V> the class of the map values
     * @param  m the map to be "wrapped" in a synchronized map.
     * @return a synchronized view of the specified map.
     */
    public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
        return new SynchronizedMap<>(m);
    }
```    


### 5. 무조건적 스레드 안전 클래스를 작성할 때는 synchronized 메서드가 아닌 비공개 락 객체를 사용하자.   
클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다. 하지만 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없게 된다.
또한, 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격을 수행할 수도 있다.
서비스 거부 공격을 막으려면 synchronized 메서드 대신 비공개 락 객체를 사용해야 한다.  
> 락 필드의 변경 가능성을 최소화하기 위해 항상 final 로 선언한다.

```java
private final Object lock = new Object();

public void foo() {
    synchronized(lock) {
        ...
    }
}

```
