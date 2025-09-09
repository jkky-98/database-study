# CASE 

CASE문은 select, order by, group by에서 사용 가능하다. 주로 select절에서 자주 사용된다.

기존의 where, join, union와 같은 것들은 데이터의 구조를 바꾸거나 범위를 한정하는 기술이지만
case의 경우 데이터 자체를 동적으로 가공하여 새로운 의미를 부여하는 기술이다.

## Simple CASE Expression
```sql
CASE status
    WHEN 'PENDDING' THEN '주문대기'
    WHEN 'SHIPPED' THEN '배송'
    WHEN 'CANCELLED' THEN '주문취소'
    ELSE '알 수 없음'
END
```
## Searched CASE Expression
```sql
CASE
    WHEN status = 'PENDDING' THEN '주문대기'
    WHEN status = 'SHIPPED' THEN '배송'
    WHEN status = 'CANCELLED' THEN '주문취소'
    ELSE '알 수 없음'
END
```
단순 CASE 문 (Simple CASE Expression)은 하나의 컬럼(ex.status) 또는 표현식을 지정하고 값에 따라 결과를 다르게 하고 싶을 때 사용한다.

검색 CASE 문 (Search CASE Expression)은 WHEN절 자체에 조건을 지정하고 만족할 경우 THEN의 결과를 나타내고 싶을 때 사용한다.

# 순서주의
WHEN-THEN-ELSE 문법에서 기본적으로 파악해야할 주의사항이다.

처음 만나는 WHEN 조건부터 파악하면서 순서대로 읽어가기 때문에 처음 조건을 통과하면 그 다음 조건은 처음 조건에는 해당하지 않는 경우로 배치해야한다. 예시를 통해 쉽게 이해가능하다

```sql
select
    case
        when price >= 1000000 then '고가' --- (1)
        when price >= 50000 then '저가' --- (2)
    end
from
    products    
```
위 쿼리문을 참고

(1)을 통과하면 (2)는 무조건 통과하기에 위 case문은 잘못된 case문이다 (1), (2) 순서가 바뀌어야한다는 것을 바로 알 수 있다.

# order by에서 사용

```sql
CASE
    WHEN status = 'PENDDING' THEN '주문대기'
    WHEN status = 'SHIPPED' THEN '배송'
    WHEN status = 'CANCELLED' THEN '주문취소'
    ELSE '알 수 없음'
END
```
이 결과로 각각의 레코드들은 주문대기, 배송, 주문취소와 같은 상태를 가질 것이다.

만약 (주문대기, 배송, 주문취소)순으로 정렬하고 싶을 때 기존의 desc, asc 정렬을 쓴다면 한글순, 한글역순으로 배치 될 것이다.

직접 case문을 활용하여 정렬순서를 줄 수 있는데, 위 쿼리문을 그대로 활용할 수 있다.

```sql
select
    CASE
        WHEN status = 'PENDDING' THEN '주문대기'
        WHEN status = 'SHIPPED' THEN '배송'
        WHEN status = 'CANCELLED' THEN '주문취소'
        ELSE '알 수 없음'
    END
from products
order by
    CASE
        WHEN status = 'PENDDING' THEN 1
        WHEN status = 'SHIPPED' THEN 2
        WHEN status = 'CANCELLED' THEN 3
        ELSE '알 수 없음'
    END
```
조건절 필터 결과로 숫자를 주어 의도된 숫자 순으로 정렬을 유도할 수 있다.

# group by
```sql
select
    CASE
        WHEN status = 'PENDDING' THEN '주문대기'
        WHEN status = 'SHIPPED' THEN '배송'
        WHEN status = 'CANCELLED' THEN '주문취소'
        ELSE '알 수 없음'
    END,
    count(*) as '주문 상태별 총량'
from products
group by
    CASE
        WHEN status = 'PENDDING' THEN '주문대기'
        WHEN status = 'SHIPPED' THEN '배송'
        WHEN status = 'CANCELLED' THEN '주문취소'
        ELSE '알 수 없음'
    END
```
위와 같이 그루핑에서도 CASE 문을 그대로 활용할 수 있다.

# 조건부 집계

## 패턴1. COUNT(CASE...)
`COUNT(CASE WHEN status = 'COMPLETED' THEN 1 END)`

COUNT 집계 함수에 위와 같이 CASE 문을 활용하여 원하는 것만 count 할 수 있다.

위 예시에서는 주문 상태가 COMPLEDTED인 경우만 1로 주고 나머지는 주지 않도록 하여 집계한다.

COUNT이기 때문에 1이던 2던 무엇을 주던 상관없다.

## 패턴2. SUM(CASE...)
`SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END)`

SUM 집계 함수에도 CASE 문을 활용할 수 있으며 count와 달리 원하는 조건에 1을 주고 나머지는 0을 주어 합쳐 count처럼 구현할 수 있다.

다른 숫자를 주어 합쳐서 원하는 구도를 만들어내도 상관없다.
