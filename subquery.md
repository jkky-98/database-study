# 서브 쿼리

## JOIN으로는 답할 수 없는 문제
Q : `우리 쇼핑몰에서 판매하는 상품들의 평균 가격보다 비싼 상품은 무엇이 있을까`

이 질문에서 건들여야할 테이블은 products, 즉 상품 정보 테이블 하나만 존재하기에 join은 고려대상이 아니다.

판매하는 상품들의 평균 가격을 where 조건절에서 활용할 수 있다면 위 질문에 대한 대답을 할 수 있다.

하지만 이를 하나의 쿼리로 형성할 수는 없고

1. 평균 가격을 구하는 쿼리
2. 평균 가격으로 where을 구성해서 필터링된 결과를 얻는 쿼리

두 절차를 거쳐야한다.

서브쿼리로 하여금 where 조건절에 `1.`과정을 그대로 넣을 수 있다.

```sql
select
    *
from
    products
where price > (
    select avg(price) from products
);
```
`select avg(price) from products` 이 결과는 컬럼명이 `avg(price)`인 컬럼을 하나만 가지며 하나의 행만 가진다.

# 스칼라 서브 쿼리

위의 예시처럼 하나의 컬럼, 하나의 행으로만 존재하는 경우 스칼라 서브쿼리라고 한다.

위의 쿼리문 예시의 where에서 스칼라 서브쿼리를 활용했는데 만약 스칼라 서브쿼리가 아닌, 다중 행 혹은 다중 컬럼 혹은 다중 행 + 다중 컬럼의 결과일 경우 에러가 난다.

# 다중 행 서브쿼리

당연하게도 다중 행 서브쿼리는 > = 와 같은 단일 스칼라 값을 활용하는 연산기호를 사용할 수 없다.

where price > (1000, 2000, 3000)은 받아들여질 수 없는 연산이기 때문이다.

그렇기에 IN, ANY, MAX, ALL, MIN을 보통 활용한다.

IN 연산자로 하여금 목록에 포함된 값과 일치하는지 확인하는 방식으로 연산을 짤 수도 있고

ANY와 ALL은 주로 <, >와 같은 비교 연산자와 함께 사용된다.

`price < ANY(다중 행 서브쿼리 결과)` : price가 서브쿼리 결과의 가장 큰 값보다 작을 때 true
`price < ALL(다중 행 서브쿼리 결과)` : price가 서브쿼리의 모든 결과보다 작을 때 true

**비교연산자 + ANY, ALL은 생각이 꼬일 수 있기에 보통 MIN, MAX를 활용하는 것이 편하다.**

# 다중 컬럼 서브쿼리

```sql
SELECT order_id, user_id, status, order_date
  FROM orders
  WHERE (user_id, status) = (SELECT user_id, status
                             FROM orders
                             WHERE order_id = 3);
```
다중 컬럼이면서 단일 행일 경우 위와같이 풀어낼 수 있다.

역시 동일하게 다중컬럼+다중행일 경우 IN, ANY, MAX, ALL, MIN과 같은 것을 활용해야한다.

비교군은 당연히 컬럼수와 같아야한다.

# 상관 서브쿼리

상관 서브쿼리는 서브쿼리가 메인쿼리의 값에 의존하는 경우이다.

`각 상품별로, 자신이 속한 카테고리의 평균 가격 이상의 상품들을 찾아라`와 같은 문제가 주어졌을 때 상품 테이블의 각각의 행들에 대응되는 카테고리 평균값이 존재할 것이다.

만약 이를 서브쿼리 없이 풀어낸다면 직접 모든 평균값을 조사하여 카테고리마다 쿼리를 짜주어야 할 것이다.

상관 서브쿼리를 활용하면 이를 쉽게 풀어낼 수 있다.

```sql
SELECT
      product_id,
      name,
      category,
      price
FROM
products p1
WHERE
      price >= (
                SELECT
                    AVG(price)
                FROM
                    products p2
                WHERE
                    p2.category = p1.category
        );
```

상관 서브쿼리는 쿼리 실행 순서가 독특한데,

product 테이블 첫 행을 읽어 category를 파악하여 서브쿼리(p1.category) 부분에 전달하고

서브쿼리가 실행된다.

또 다시 product 테이블 두번째 행을 읽고 category를 파악하여 서브쿼리에 전달하고 서브쿼리가 실행된다.

이를 반복한다.

즉 product테이블의 행의 수만큼 서브쿼리가 실행된다.(비용에 관한 문제인식이 생기게 된다.)







