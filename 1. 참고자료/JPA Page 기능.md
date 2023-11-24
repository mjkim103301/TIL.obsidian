![[Pasted image 20231124164952.png]]

JpaRepository 는 부모로 PagingAndSortRepository 인터페이스를 가지고 있다.
이 인터페이스에서 페이징 기능과 정렬 기능을 제공한다.

PagingAndSortingRepository 인터페이스에서 아래와 같은 메서드를 사용할 수 있다.
```
Page<T> findAll(Pageable pageable); 

```

인자로 Pageable을 넘기고 있다.



