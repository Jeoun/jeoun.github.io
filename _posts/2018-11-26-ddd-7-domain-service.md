
---
layout: post
author: "jeoun"
---


# DDD - 도메인 서비스

## 도메인서비스가 왜 필요한가?

하나의 애그리거트로 기능을 구현하기 어려운 경우가 있다. 

하나의 예로 결제 금액 계산 로직을 보겠다. 
결제금액 계산로직에서 필요한 애그리거트는 아래와 같다.
1. 상품 애그리거트 : 상품의 가격/배송비
2. 주문 애그리거트 : 구매 개수 
3. 할인 쿠폰 애그리거트 : 할인 쿠폰
4. 회원 애그리거튼 : 회원등급에 따른 추가할인율

위 4개의 애그리거트에서 결제금액에 대한 로직을 가지고 있어야 할까?

Order(주문)애그리거트에서 결제 금액과 관련된 애그리거트나 필요 데이터를 모두 가지고 금액 계산의 책임을 주문 애그리거트에 할당하는 방식을 생각해보자.

````
public class Order {
    ...
    private Orderer orderer;
    private List<OrderLine> orderLines;
    private List<Coupon> usedCoupons;

    private Money calculatePayAmounts() {
        Money  totalAmounts = calculateTotalAmounts();

        Money discount = usedCoupons.stream().map(coupon -> calculateDiscount(coupon))
                .reduce(Money(0), (v1, v2) -> v1.add(v2));

        Money membershipDiscount = calculateDiscount(orderer.getMember().getGrade());

        return totalAmounts.minus(discount).minus(membershipDiscount);
    }

    private Money calculateDiscount(Coupon coupon) {
        ....
        return money;
    }

    private Money calculateTotalAmounts() {
        ...
        return money;
    }
}
````
과연 결제 금액 계산 로직이 애그리거트의 책임이 맞는가의 고민이 생긴다.
새로운 이벤트로 인해서 상품에 대한 추가 할인이 발생될 경우  
order 애그리거트에서는 자신과 상관없는 정보임에도 불구하고 이벤트에 대한 코드 수정이 필요하다.

하나의 애그리거트에 넣기 애매한 도메인 기능을 특정 애그리거트에 넣게 될경우 코드가 길어지기도 하며  
외부에 대한 의존도가 높아지게 된다.

이렇듯 하나의 애그리거트에서 처리하기 어려운 문제를 해소하기 위해 나온 구현방식이 도메인 서비스를 별도 구현하는 방식이다.

## 도메인 서비스 

하나의 애그리거트에 넣기 애매한 도메인 개념을 애그리거트에 억지로 넣기보다는 도메인 서비스를 이용해서 도메인 개념을 명식적으로 드러내면된다.
여기서 도메인서비스는 도메인 영역의 로직을 다룬다.

도메인 서비스가 다른 구성요소와의 다른점이 있다면 상태없이 로직만을 구현한다는 것이다. 도메인 서비스에서 구현하는데 필요한 상태정보는 애그리거트나 다른방법으로 전달해 처리한다.

````
public class DiscountCalcualationService {
    public Money calculateDiscountAmounts(List<OrderLine>orderLines, List<Coupon> coupons, MemberGrade memberGrade) {
        Money couponDiscount = coupons.stream().map(coupon -> calculateDiscount(coupon))
                .reduce(Money(0), (v1,v2) -> v1.add(v2));

        Money membershipDiscount = calculateDiscount(memberGrade);

        return couponDiscount.add(membershipDiscount);
    }

    private Money  calculateDiscount(Coupon coupon) {
        return null;
    }

    private Money calculateDiscount(MemberGrade memberGrade) {
        return null;
    }
}
````

위와 같이 도메인 서비스를 별도로 구현한뒤 애그리거트나 응용서비스에서 해당 도메인서비스를 사용할수 있다.

도메인서비스를 사용하는 방식은 여러가지가 있을 수 있다.

1 애그리거트에서 도메인 서비스를 사용할경우

애그리거트가 도메인서비스에 의존성을 갖도록 하는 것은 좋지 않다.  
애그리거트가 도메인서비스에 의존성을 갖는다는 것은 데이터와 상관없는 도메인서비스를 포함한다는 것이기 때문인다.   
그렇기 때문에 애그리거트에서 도메인서비스를 사용하고자 한다면 응용 서비스에서 애그리거트로 도메인 서비스에 대한 정보를 넘겨 사용할 수 있도록하는것이 좋다.

2 도메인 서비스에서 애그리거트를 사용할 경우

도메인 서비스의 매소드를 호출할 때 애그리거트를 전달하여 도메인 기능을 실행한다.   
이때 주의점은 트랜잭션처리와 같은 응용 로직은 도메인 서비스의 롤이 아니기에 호출하는 응용 서비스 영역에서 담당하는 것이 좋다.

## 도메인 서비스의 패키지 위치

도메인 서비스는 도메인 로직을 실행하므로 다른 도메인구성요소와 동일한 패키지에 위치한다.

## 도메인 서비스의 인터페이스와 클래스 

도메인 서비스의 구현이 특정 기술에 종속된다면 인터페이스를 통해서 추상화하여 도메인이 특정 구현에 종속되는 것을 방지할 수 있다.

 

