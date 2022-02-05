item75.예외의 상세 메시지에 실패 관련 정보를 담으라

### 예외의 toString 메서드에 실패 원인에 대한 정보를 가능한 한 많이 담아 반환하는 일은 중요하다
- 스택에는 아래와 같이 예외 객체의 toString 을 호출해 얻는 문자열이 표시되므로 객체의 toString 메서드에 실패 원인에 관한 정보를 가능한 많이 담아 반환한다면 실패 원인 분석에 도움이 된다.
```
Exception in thread "main" IndexOutOfBoundsException.toString() 에서 리턴된 string
        at Item75.main(Item75.java:9)
```

### 예외 메시지는 가독성보다는 담긴 내용이 훨씬 중요하다
- 예외를 유발한 문제를 분석하는 사람은 스택 추적뿐 아니라 관련 문서와 (필요하다면) 소스코드도 함께 살펴보기 때문에 문서와 소스코드에서 얻을 수 있는 정보는 길게 늘어놔봐야 군더더기가 될 뿐이다.
- 다만, 보안과 관련된 정보는 주의해서 다뤄야 한다. 문제를 진단하고 해결하는 과정에서 스택 추적 정보를 많은 사람이 볼 수 있으므로 상세 메시지에 비밀번호나 암호 키 같은 정보까지 담아서는 안 된다.
- 발생한 예외에 관여된 모든 매개변수와 필드의 값 등 필요한 정보를 예외 생성자에서 모두 받아서 상세 메시지를 미리 생성해놓는 방법도 있다.

### 예외는 실패와 관련한 정보를 얻을 수 있는 접근자 메서드를 적절히 제공하는 것이 좋다
- 아래의 예에서는 lowerBound, upperBound, index 를 제공할 수 있으나, item70. 의 내용과 같이 쇼핑몰에서 물건을 구입하려는 데 카드 잔고가 부족하여 검사 예외가 발생하면, 이 예외는 잔고가 얼마나 부족한지 알려주는 접근자 메서드를 제공하는 것이 좋다.

``` java
public class Item75 {
    public static void main(String[] args) throws Exception {
        int minIndex = 100;
        int maxIndex = 500;
        int inputIndex = 1000;

        // 예외 분석에 필요한 내용이 담겨 있지 않은 경우
        // if (inputIndex < minIndex || inputIndex > maxIndex) {
        //     throw new IndexOutOfBoundsException();
        // }
        
        // 예외 분석에 필요한 내용이 담겨 있는 경우
        if (inputIndex < minIndex || inputIndex > maxIndex) {
            throw new IndexOutOfBoundsException(minIndex, maxIndex, inputIndex); 
        }
    }
}

/**
 * IndexOutOfBoundsException 을 생성한다
 * 
 * @param lowerBound 인덱스의 최솟값
 * @param upperBound 인덱스의 최댓값 + 1
 * @param index 인덱스의 실젯값
 */
class IndexOutOfBoundsException extends RuntimeException {
    int lowerBound;
    int upperBound;
    int index;

    public IndexOutOfBoundsException() {
        super();
    }

    public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
        super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));

        this.lowerBound = lowerBound;
        this.upperBound = upperBound;
        this.index = index;
    }

    // public String toString() {
    //     return "IndexOutOfBoundsException.toString() 에서 리턴된 string";
    // }
}
```

#### 에러 메시지
```java
Exception in thread "main" MyException
        at Item75.main(Item75.java:7)

Exception in thread "main" IndexOutOfBoundsException: 최솟값: 100, 최댓값: 500, 인덱스: 1000
        at Item75.main(Item75.java:14)
```
