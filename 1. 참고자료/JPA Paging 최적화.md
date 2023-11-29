## JPA 페이징 처리에서 발생할 수 있는 성능 이슈

JPA 의 페이징 기능을 사용하게 되면 다음 메서드를 실행하게 된다.
```
Page<T> findAll(Pageable pageable);
```

그러면 select, count 쿼리가 각각 날라가게 된다.
```
Hibernate: select userentity0_.id as id1_0_, userentity0_.created_at as created_2_0_, userentity0_.email as email3_0_, userentity0_.password as password4_0_, userentity0_.updated_at as updated_5_0_ from user userentity0_ limit ?, ? Hibernate: select count(userentity0_.id) as col_0_0_ from user userentity0_
```

성능 이슈는 count 쿼리를 실행할 때 발생한다.
count 쿼리는 테이블 full-scan을 필요로 하기 때문에 테이블에 데이터가 많을수록 성능 저하가 발생한다.



## 성능 이슈 해결 방안
1. 만약 조회한 데이터에서 Total Count 데이터가 필요하지 않다면 다른 반환 타입을 사용할 수 있다. (count 쿼리를 실행하지 않기 위한 방법)
	- List
	- Slice
```
class OrderCustomRepositoryImpl : QuerydslRepositorySupport(Order::class.java), OrderCustomRepository {  
    override fun findSliceBy(pageable: Pageable, address: String): Slice<Order> {  
        val query: JPAQuery<Order> = from(order).select(order).where(order.address.eq(address))  
        val content: List<Order> = querydsl.applyPagination(pageable, query).fetch()  
        val hasNext: Boolean = content.size >= pageable.pageSize  
        return SliceImpl(content, pageable, hasNext)  
    }  
}
```
1. QueryDSL
	- 조건절을 재사용한다. ..?
2. 여러 테이블을 조인해서 조회할 때 count 쿼리는 최대한 조인하지 않고 사용한다.
```
-- Content 조회 쿼리  
select o.*,  
       u.*,  
       c.*  
from orders o  
         left join coupon c on o.coupon_id = c.id  
         inner join user u on o.user_id = u.id  
where o.address = ? limit ?, ?  
;  
  
-- Content 조회 쿼리를 그대로 사용하는 경우  
select count(o.id)  
from orders o  
         left join coupon c on o.coupon_id = c.id  
         inner join user u on o.user_id = u.id  
where o.address = ?  
;  
  
-- Content 쿼리를 사용하지 않고 별도의 Count 조회 쿼리  
select count(o.id) ascount  
from orders o  
where o.address = ?  
;
```


4. Content 조회 쿼리, Count 조회 쿼리 병렬처리하기
	*  Content 조회쿼리는 500ms 소요되고, Count 조회 쿼리는 1000ms 소요될 때 전체적인 페이징 시간은 1500ms 소요되게 된다.
	* 위의 2개의 조회 작업은 의존성이 없기 때문에 병렬로 처리할 수 있다.
```
class OrderCustomRepositoryImpl : QuerydslRepositorySupport(Order::class.java), OrderCustomRepository {  
    override fun findPagingBy(pageable: Pageable, address: String): Page<Order> = runBlocking {  
        log.info("findPagingBy thread : ${Thread.currentThread()}")  
        val content: Deferred<List<Order>> = async {  
            log.info("content thread : ${Thread.currentThread()}")  
            from(order)  
                .select(order)  
                .innerJoin(user).on(order.userId.eq(user.id))  
                .leftJoin(coupon).on(order.couponId.eq(coupon.id))  
                .where(order.address.eq(address))  
                .run {  
                    querydsl.applyPagination(pageable, this).fetch()  
                }  
        }  
        val totalCount: Deferred<Long> = async {  
            log.info("count thread : ${Thread.currentThread()}")  
            from(order)  
                .select(order.count())  
                .where(order.address.eq(address))  
                .fetchFirst()  
        }  
  
        PageImpl(content.await(), pageable, totalCount.await())  
    }  
}
```


[Spring, JPA 페이징 처리와 성능 최적화 알아보기](https://blog.leaphop.co.kr/blogs/36)
[JPA 페이징 Performance 향상 방법](https://cheese10yun.github.io/page-performance/)