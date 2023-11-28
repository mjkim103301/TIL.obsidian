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
1. 만약 조회한 데이터에서 페이지 데이터가 필요하지 않다면 다른 반환 타입을 사용할 수 있다. (count 쿼리를 실행하지 않기 위한 방법)
	- List
	- Slice
2. QueryDSL
	- 조건절을 재사용한다. ..?


[Spring, JPA 페이징 처리와 성능 최적화 알아보기](https://blog.leaphop.co.kr/blogs/36)
[JPA 페이징 Performance 향상 방법](https://cheese10yun.github.io/page-performance/)