# 내부 조인(INNER JOIN)

두 테이블을 연결하여 하나의 테이블의 결과로 보여준다.

```sql
select
    u.user_name
from
    users u
inner join
    orders o on u.user_id = o.user_id
```

위와 같이 inner join 할 수 있다.

`on`절에서 연결의 조건을 지정하여 참이되는 부분에 대해 서로의 테이블의 행이 합쳐진 형태의 새로운 행을 추출하는 것이다.

즉 user_id를 기준으로 on절을 작성하였으니,
users 테이블의 행에도 user_id가 존재해야하며 orders 테이블의 행에도 user_id가 존재해야한다.

- inner join은 inner을 생략하여 join으로만 사용해도 무방하다.(보통 그렇게 사용한다.)
- inner join은 서로의 테이블의 행들에 대해 on 절의 조건을 만족하는 경우에만 연결한다, 즉 교집합 관계이다.
- 테이블들인 users u, orders o는 alias로 as를 생략해서 표현하는 것이 일반적인 관행이다, 가독성 면에서 select절에 들어가는 컬럼명은 as를 사용하는 것이 일반적인 관행이다.
- 만약 users 테이블에만 user_name이라는 컬럼이 존재한다면 위 SQL 쿼리문에서 `u.`를 생략해서 user_name만 작성해도 무방하다. 하지만 서로의 테이블에 컬럼명이 같을 경우 select절에 이를 표현하기 위해서는 앞에 테이블 출처를 명확히 적어야한다.
- 컬럼명 작성시 테이블 출처를 적도록 하자. 추후 쿼리문을 확인할 때 이 컬럼이 어디 테이블에서 온 것인지 파악하기 쉽기 때문이고 바로 위 문제 사례도 피할 수 있다.
- users(from)에서 orders(inner join)를 조인하는 것과 orders(from)에서 users(inner join)을 조인하는 것은 차이가 없다. 어차피 교집합만 보여주는 것이 inner join이기 때문
- 교집합이므로 어느 방향으로 join하던 결과의 차이는 없지만 가독성을 위해 이 쿼리문을 작성한 목적을 생각해서 from + join의 순서를 결정하자. (예 : 주문 목록을 중심으로 고객정보를 추가하고 싶을 경우 -> from orders join users)
- 만약 where문이 존재한다면 이는 join이 이루어진 후 적용된다.
- from+join이 이루어지고 where -> group by -> having... 의 순서로 기존 순서에 from 부분에 join이 달라붙었다고 이해하자.
- join은 여러번에 걸쳐 수행할 수 있다. join ~ on 문 작성 후 다시 join ~ on 문을 작성하여 2개 이상의 테이블을 from절 테이블과 연결가능하다.

# 외부 조인(OUTER JOIN)

```sql
select
    u.user_name
from
    users u
left outer join
    orders o on u.user_id = o.user_id

select
    u.user_name
from
    users u
right outer join
    orders o on u.user_id = o.user_id
```

## 외부 조인이 필요할 이유

외부 조인은 내부 조인의 결과(교집합)와 함께 

교집합은 아닌 한쪽 테이블의 행을 보여준다.

아래의 사례를 참고하면,

users.user_id = (1,2,3)
orders.user_id = (1,1,2)

회원은 1,2,3으로 총 3명 존재하고, 주문의 경우 id=1이 두번 수행하였으며, id=2가 한번 수행하였으므로 id=3은 한번도 하지 않았다.

이때 교집합은 (1,2)이고 users 테이블 부분에 교집합이 아닌 부분 id=3이 존재한다.

내부 조인할 경우 (1,2)로만 연결되어 id=3인 행은 누락된다.

하지만 외부 조인할 경우 id=3 행이 "보일 수 있다".

보일 수 있다고 표현한 이유는 외부 조인은 조인 순서가 중요하기 때문이다.

만약 from절로 users를 사용하고 left outer join orders을 사용한다면 id=3행이 user_id가 null인채로 join 테이블에 포함된다.

반면 from절로 users를 사용하고 right outer join orders를 사용한다면 id=3행이 누락된다.

또한 from절로 orders를 사용하고 left outer join users를 사용한다면 id=3 행이 누락되며 right outer join users를 사용하면 포함된다.

from절에 오는 테이블을 주 테이블이라고 가정한다면 left outer join은 주 테이블의 모든 결과를 보여주는 것이다.

right outer join은 조인 테이블의 모든 결과를 보여주는 것이다.

책을 왼쪽에서 오른쪽으로 읽는 느낌으로 받아들일 수 있다.

## 외부 조인에 대한 팁
- left(right) outer join은 outer를 생략하고 사용가능하다. = left join, right join
- 그렇다면 생략된 표현으로만 정리하면 우리는 `join`, `left join`, `right join`을 사용할 수 있다.
- right join은 보통 사용되지 않는다. 이는 쿼리를 쉽게 이해하는 것을 방해하기에 그렇다. left join의 사용으로만 고정하고 from절의 들어올 테이블만 조절해준다면 원하는 결과를 얻을 수 있다. 일관성에서 오는 이해향상과 외부 조인 사용시 주 테이블의 결과를 모두 보여주는 것이 직관적이기 때문이다.
- "교집합이 아니면서 주(조인) 테이블 소속"을 구할 때 유용하다. -> 내부 조인으로는 이것을 알 수 없기 때문이다.

# 조인의 특징

- join의 on절은 pk, fk 관계가 아니더라도 사용가능하지만 일반적으로는 pk, fk관계로 테이블을 연결한다.
- FK를 가진 쪽에서 PK로 조인 할 경우(ToOne) 조인 후 행의 개수는 동일하다.
- PK를 가진 쪽에서 FK로 조인 할 경우(ToMany) 조인 후 행의 개수는 늘어날 수 있다.
- 특수한 경우가 아니라면 FK를 가진쪽(자식 테이블, 외래키의 주인)에서 PK(부모 테이블)로 연결하도록 하자.


# 셀프 조인

하나의 테이블에서 pk를 활용해서 pk값을 이용한 새로운 id컬럼을 만들 수 있다.

예로 user_id가 pk인 테이블에 manager_id(해당 user를 관리하는 책임자)가 존재할 수 있다.

그 때 managers라는 새로운 테이블을 두기보단,

같은 테이블에 두어 user_id(pk)를 사용하고, 필요시 셀프 조인을 이용하면 편리하다.

user_id가 1이고 manager_id가 3인 경우

pk=1인 유저는 pk=3인 유저의 관리를 받는다고 볼 수 있다.

물론 manager_id가 null일 수 있다.

그렇다 같은 테이블에 FK를 두는 느낌인 것이다.

그렇기에 조인 시에도

from user join user 이렇게 같은 테이블을 from, join 하는 것이다.

# CROSS JOIN

우선 굉장히 위험한 조인이다.

A테이블의 행이 3개 B테이블의 행이 4개라면

이 둘을 크로스 조인 할 경우 12개의 행을 가진 결과가 도출된다.

그렇기에 크로스 조인은 ON절 또한 없다.

그냥 무식하게 둘의 카테시안 곱을 치는 것이 크로스 조인이다.

예를 들어 사이즈 정보 (S, XS, M, L, XL) 와 같은 테이블이 있고색상 정보를 가진 테이블이 있다면 서로 크로스조인한 S, Blue 이런 조합들을 만들어낼 수 있다.

## INSERT INTO SELECT

insert into select 는 select된 여러 행을 한번에 해당 테이블에 insert 하는 것을 말한다.


