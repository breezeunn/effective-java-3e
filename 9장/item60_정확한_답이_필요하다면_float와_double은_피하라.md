item60.정확한 답이 필요하다면 float와 double 은 피하라


### 핵심 요약
- 정확한 답이 필요한 계산에는 float 나 double 을 피하자.
  - int: 숫자를 아홉 자리 십진수로 표현할 수 있는 경우
  - long: 숫자를 열여덟 자리 십진수로 표현할 수 있는 경우
  - BigDecimal: 열여덟자리가 넘어가는 경우


### 정확한 결과가 나오지 않는 경우 1
아래 코드의 결과로 0.61 을 기대하지만, 실제로는 0.6100000000000001 이 출력된다.
```java
System.out.println(1.03-0.42);
``` 

### 정확한 결과가 나오지 않는 경우 2
주머니에 1달러가 있고, 선반에는 10센트, 20센트, ... 1달러짜리 사탕이 있다고 가정하고 10센트짜리부터 하나씩 살 수 있을 때 까지 산다면 사탕을 몇 개 살 수 있을까? 잔돈은 얼마나 남을까?

0.1 + 0.2 + 0.3 + 0.4 = 4개 살 수 있고, 잔돈이 없는 것을 기대하지만 아래 코드를 실행하면 3개, 잔돈은 0.3999999999999999 로, 의도하지 않은 결과가 출력된다.
```java
double funds = 1.00;
int itemsBought = 0;
for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
}
System.out.println(itemsBought + "개 구입");
System.out.println("잔돈(달러):" + funds);    
```

double 타입을 BigDecimal 로 교체 후 올바른 답이 나옴을 알 수 있다.
```java
public static void main(String[] args) {
    //double funds = 1.00;
    final BigDecimal TEN_CENTS = new BigDecimal(".10");
    BigDecimal funds = new BigDecimal("1.00");
    int itemsBought = 0;
    // for (double price = 0.10; funds >= price; price += 0.10) {
    for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {    
        // funds -= price;
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러):" + funds);    
}
```

하지만 BigDecimal 은 기본 타입보다 쓰기가 불편하고, 느리다는 단점이 있다. 사실 위의 예는 모든 계산을 달러 대신 센트로 수행하면 BigDecimal 을 사용하지 않아도 되었다.
```java
int funds = 100;
int itemsBought = 0;
for (int price = 10; funds >= price; price += 10) {
    funds -= price;
    itemsBought++;
}
System.out.println(itemsBought + "개 구입");
System.out.println("잔돈(달러):" + funds);
```