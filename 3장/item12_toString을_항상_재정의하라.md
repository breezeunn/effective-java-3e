# item12. toString을 항상 재정의하라
## 핵심   
**모든 구체 클래스에서 Object의 toString을 재정의하자.**   
> 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다.   
> toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 >디버깅하기 쉽게 해준다.   
> toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.

---

## toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
> Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.   
> 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.

``` java
class PhoneNumber {

	int areaCode, prefix, lineNum;
	
	public PhoneNumber(int areaCode, int prefix, int lineNum) {
		this.areaCode = areaCode;
		this.prefix = prefix;
		this.lineNum = lineNum;
	}

	/* @Override
	public String toString() {
		return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
	} */
}
```   

> 주석 처리 후 실행 시 결과
``` java
    public static void main(String[] args) {
		PhoneNumber pn = new PhoneNumber(123, 456, 7890);
		System.out.println(pn + "에서 오류가 발생했습니다.");
	}
```
> PhoneNumber@515f550a에서 오류가 발생했습니다.  
  (클래스_이름@16진수로_표시한_해시코드)

> 하지만, 주석을 해제하면  
> 123-456-7890에서 오류가 발생했습니다.  


## toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다.
> 물론 포맷을 명시하면 좋으나 포맷을 명시하면 평생 포맷에 얽매이게 된다.   
> 포맷 명시와는 관계없이 의도는 명확히 밝혀야 한다.   
**포맷 명시와는 관계없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API 를 제공하자.**   
그렇지 않으면 정보가 필요한 경우 toString 반환값을 파싱할 수 밖에 없다.

### 포맷 명시 예제 코드
```java
/**
  * 이 전화번호의 문자열 표현을 반환한다.
  * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다. 
  * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
  * ...
  * (생략)
  * ...
  */
  @Override
  public String toString() {
      return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
  }

```

### 포맷 명시하지 않는 예제 코드
```java
/**
  * 상세 형식은 정해지지 않았으며, 향후 변경될 수 있다.
  * "[약물 #9: 유형=사랑, 겉모습=먹물]"
  * ...
  * (생략)
  * ...
  */
  @Override
  public String toString() { ... }

```