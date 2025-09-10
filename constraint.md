# 데이터 무결성

"Garbage In, Garbage Out (GIGO)"

쓰레기가 들어가면 쓰레기가 나온다는 뜻으로 잘못된 데이터가 데이터베이스에 저장되는 순간 그 데이터 기반으로 한 분석, 어플리케이션 동작 등이 신뢰를 잃어버리게 된다.

쓰레기 데이터 하나가 초래할 영향은 생각보다 클지도 모른다. 

또한 일이 벌어지면 단순히 이것을 삭제하는 것으로 해결되지 않을지도 모른다.

어쨋든 어플리케이션은 그 쓰레기 데이터에 영향을 받아 다른 추가적인 결과를 생산했을지도 모르기 때문이다.

이러한 쓰레기 데이터가 들어오는데에는 누구의 책임이 가장 클까?

어플리케이션단에서 제대로 걸러내지 못한 것, DB에 데이터가 들어오기전 제대로 걸러내지 못한 것

둘 다의 책임으로, 소프트웨어 시스템에서는 어플리케이션단에서도, 데이터베이스단에서도 적절한 검증 조치가 필요하다.

## PRIMARY KEY
primary key는 기본개념상 중복과 null을 허용하지 않기 때문에 PRIMARY KEY로 지정한 컬럼은 당연히 NOT NULL, UNIQUE 효과가 포함된다.

## FK - 외래키 제약조건
`참조 무결성`이란 두 테이블의 관계가 항상 유효하고 일관된 상태를 유지해야한다는 뜻이다.

예를 들어, fk인 user_id 컬럼을 orders 테이블이 가지고 있을 때, 이 user_id는 users 테이블에 "실제로 존재하는" user_id를 참조하고 있어야한다.

이말은즉 orders의 user_id는 항상,무조건! users에 존재해야한다.

항상,무조건!을 지키게 하기 위해 존재하는 것이 FK이며 FK가 발려져있지 않은 채 존재하는 orders테이블의 user_id는 안전장치가 전혀 없는 건물이 되고만다.

- fk 제약조건은 자식 테이블의 삽입, 수정시 users에 user_id 값이 존재하는지 확인한다.
- fk 제약조건은 부모 테이블에서 삭제나 수정시 자식 테이블에서 참조하고 있는 user_id 값을 가진 행을 함부로 삭제하거나 수정할 수 없도록 한다.(함부로 수정한다면 부모를 잃을 수 있기 때문)

## CASCADE

CASCADE는 부모의 변화(삭제 및 수정)을 자식에게도 그대로 전파하는 것을 말한다.

```sql
CREATE TABLE orders (
      order_id BIGINT AUTO_INCREMENT,
      user_id BIGINT NOT NULL,
      product_id BIGINT NOT NULL,
      order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
      quantity INT NOT NULL,
      status VARCHAR(50) DEFAULT 'PENDING',
      PRIMARY KEY (order_id),
        CONSTRAINT fk_orders_users FOREIGN KEY (user_id)
            REFERENCES users(user_id) ON DELETE CASCADE, -- CASCADE 옵션 추가
        CONSTRAINT fk_orders_products FOREIGN KEY (product_id)
            REFERENCES products(product_id)
        );
```
CASCADE는 외래키를 거는 명령어에 추가할 수 있으며 ON DELETE, ON UPDATE와 함께 사용 가능하다.

기본적으로 아무것도 적지 않으면 ON DELETE RESTRICT, ON UPDATE RESTRICT가 디폴트로 적용되는데 자식 테이블에 참조하는 행이 있으면 부모 테이블의 행을 삭제/수정할 수 없는 우리가 항상 사용해오던 FK 방식이다.

ON DELETE CASCADE, ON UPDATE CASCADE를 사용하게 되면, 부모 테이블에서 삭제시 FK로 연결된 자식 테이블의 행 또한 삭제되며, ON UPDATE CASCADE는 pk 수정시 자식 테이블의 fk도 수정되는 것을 말한다.

하지만 ON UPDATE CASCADE는 거의 사용되지 않는데, pk를 수정할 일은 없기 때문이다.

### CASCADE는 안쓰는게 좋다.
CASCADE가 필요한 비즈니스(부모가 지워지면 자식이 모두 지워지는)라도 이를 사용하지 않고 애플리케이션 단에서 개별로 처리하는 것이 좋은데,

그 이유는 삭제자체가 굉장히 민감한 절차라 그렇다. 그렇기에 부모 삭제시 자식이 삭제되는게 꼭 필요한 경우라도 어플리케이션단에서 명확하게 코드로 처리할 필요가 있다.

